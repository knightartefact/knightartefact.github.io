---
layout: post
title: Blogging with Obsidian
date: 2025-12-16 20:29 +0100
categories:
  - Personal Thoughts
tags:
  - Blogging
  - Workflow
  - Obsidian
---
I've started my blog a little over a year ago with my [[2024-11-01-well-how-do-i-start-with-this|first blog post]]. I have only written only a handful of "devlogs" about the development of my Playstation emulator in C++. Currently I'm in a sticky situation with my emulator but this is a story for another time. I've taken a (really) big break from writing articles even though I like it a lot.

## Writing a blog introduces friction
When I started to write blog posts I just downloaded a Jekyll template, wrote my articles with VSCode and previewed them by spinning up a Jekyll server. It was a lot of effort each time I wanted to write a single post. The part that introduced the most friction was the writing part. Writing notes and posts with a code editor is not such a great idea, you don't get instant feedback on what you write so ideas do not come as naturally as with a dedicated notes editor such as Obsidian. I am using Obsidian to write this post.
Now, when I'm writing anything, I don't feel as much "resistance" from the process. Words flow more naturally and my ideas are clearer. I still need to rewrite or reword some parts, but it does not feel like work.

## Organizing my notes and posts
One problem still remains. I have solved my writing-friction-related issues by using Obsidian. But now how do I organize my ideas, notes and posts. I want to keep some parts of my vault private, while other parts have to be published for you readers, to be able to access them online.

For now I didn't find a solution that I feel is 100% compatible with my workflow. Some great people such as Steph Ango, use 2 vaults to keep their notes private and publish onyl the ones that are meant to be seen. I don't really like that approach because it splits the whole writing workflow? The 2 separate vaults need to be kept in sync periodically which I find to be a pain.

What I currently do is directly write my posts in my blog repo to avoid duplicating the data. I developed a small plugin to manage my drafts and posts. The plugin is called **Jekyll Compose** from the ruby plugin of the same name which I took a lot of inspiration from.

## My low-friction workflow
To summarize quickly I do the following to write and publish my notes.
- My Obsidian vault points to the Git repository of the blog
- Edit my notes and posts with Obsidian
- Create posts and drafts using my Jekyll Compose plugin
- Deploy the site with Github Actions

## Towards a frictionless workflow
In the future I wish I could unify my blog workflow with my personal blog and even other blogs. A single vault that allows me to put my every thought and idea in the same place. Like a second brain. The problem with finding the perfect solution is balance. I don't want to have to set up something for hours upon hours just for it to work, and neither I want something that is easy to setup up but just doesn't so what I want.

I will just keep on thinking about potential solutions, trying to implement them as I go and experiment to maybe find the one that suits me best.
