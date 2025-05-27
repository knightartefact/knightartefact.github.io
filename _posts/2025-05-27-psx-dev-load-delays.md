---
layout: post
title: 'PSX Dev: Load Delays'
categories:
- PSX Dev
- RogEm
tags:
- PSX
- Emulator
- Programming
- CPU
description: Load delays enable the CPU to be pipelined making it faster. They appear to be easy to implement (and they are...) but it can sometimes be tricky to understand and implement them correctly on your first try.
date: 2025-05-27 20:15 +0900
---
Ahhhh load delays, I really don't know how it took me so long to correctly implement them.
I had an issue with load delays being overridden when multiple load instructions followed each other.

In my implementation, a load delay is a simple structure:
```cpp
struct LoadDelaySlot
{
    uint32_t value;
    uint8_t destRegister;
    int delay;
}
```

Which is then instantiated in my CPU class:
```cpp
class CPU {
    public:
        CPU(...)

    private:
        // More class code above
        LoadDelaySlot m_loadDelaySlot;
        // More class code below
}
```

Each time the CPU need to load data from memory, the structure is modified to reflect the register changes in the future. This means the destination register and the new values are set, as well as the delay needed which is 1 instruction delay.

Now in the cycle/step function of the CPU we decrement the delay and if it hits 0 we load the data to the destination register. Seems alright, riiight ? Well it was not at all because i forgot to take into account successive load operations....

> All registers are clear when executing the following code
{: .prompt-info}

> The parameters *str1* and *str2* are passed from the function argument registers *\$a0* and *\$a1* respectively. They are `char *` strings
{: .prompt-info}

So let's imagine we have this MIPS assembly code snippet form the **strcmp** function:

```s
0x0    lb         v0,0x0(str2)
0x1    lb         v1,0x0(str1)
0x2    addiu      str2,str2,0x1
0x3    bne        v0,v1,ReturnNotEqual
```

The first byte load sets the `m_loadDelaySlot` structure's internal data to the `$v0` register and the first byte of the `str2` string. Finally it sets the delay to 1 instruction.
Now when loading the second byte, we must also set the destination register and value to the load delay slot structure. The issue with that is we ***already*** have a pending load which will be written to the register during/after the execution of the current lb instruction. This means we are effectively discarding the previous load which will never be written and will inevitably lead to errors.
Okay now I know where the problem is coming from but how do I fix it ?

Simple solution: use an array of `LoadDelaySlot` to basically queue load delays
The array is 2 elements long since successive loads won't grow the queue bigger than 2 elements
We also discard the delay variable inside the struct since we no longer need it
When a load occurs there are 2 mechanics:

```cpp
struct LoadDelaySlot
{
	uint32_t value;
	uint8_t destRegister;
};

class CPU
{
	public:
        CPU(...)

	private:
        LoadDelaySlot m_loadDelaySlots[2];
};
```
**At the moment of executing the load instruction**
1. Set the load delay value and destination register at the back of the queue
2. That's it

**At the moment of checking the current pending loads**
1. If a load is waiting to be written (aka is in the front of the queue)
	1. Load the data to the destination register
	2. Reset the load delay slot to 0
2. If a load has occurred on the previous instruction
	1. Move the the load data to the front of the queue
	2. Reset the original load to 0 (since writing too register 0 does nothing)

Last but not least, the LWL and LWR load delays behaviors.
If a LWL or LWR instruction directly follows a load instruction and affects the same register as that previous load instruction, then there is no delay between these 2 operations. There will still be a delay applied **AFTER** the last unaligned load instruction that affects the same register.

> Reference from the NOCA$H PSX specification
{: .prompt-info}

```s
lwl    r2, 0x0003($t0) ;\ no delay required between these
lwr    r2, 0x0000($t0) ;/ (although both access r2)
nop                    ;-requires load delay HERE (before reading from r2)
and    r2, r2, 0xffff  ;-access r2 (eg. reducing it to unaligned 16bit data)
```

This weird behavior allowed the game programmers to load any unaligned word into a register.

I'm happy I fixed this issue in my CPU.
I will ***finally*** be able to work on the graphical side of my emulator.

There is an issue when successive load instructions use the same target register.

![load delay to same register](assets/img/posts/loads_to_same_register.png)
_Successive loads targetting the same register_

So when calling `loadWithDelay()` function, we need to check if the target register is the same as the pending load in the queue. If so, then we discard the pending load and queue a new load "event" at the back of the queue.

Now when a LWL/LWR instruction follows a normal load instruction it doesn't work anymore because of the mechanism just above. The load delay is discarded even when an unaligned instruction follows, which should not reset the load delay.

This is because LWL and LWR are not checking if they targeted the same register as the pending load, thus resetting loads even when not targeting the same register.
