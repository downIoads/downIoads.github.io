---
title: "How to write a tutorial"
date: 2023-07-02T14:00:23+02:00
description: Things to remember when you decide to write a tutorial.
draft: false
tags: [tutorial]
---

## Introduction

So you have decided to write a tutorial? Remember that there is nothing more frustrating than being a beginner at something, trying to become better at that thing by going through a tutorial and then being unable to complete the tutorial due to errors or unclear instructions. If you are not willing to write a good tutorial, it is better to not write a tutorial at all. So here are a few basic things that you should consider when trying to write a good tutorial.

## Checklist for writing good tutorials

### Date
Tutorials on technical topics will be outdated at some point. Therefore, it is crucial to include the date at which it was created. Don't waste other peoples' time with outdated tutorials.

### Complete information
Your readers will not be experts at what you are writing about. So do not skip steps that seem trivial to you. If you are writing a tutorial about how to set up and host a simple static website using Gohugo and Github Pages, then do NOT assume that your reader is an expert at git. So instead of writing: "Push your local repository to GitHub.", you should instead take the time to figure out the exact git commands that your reader will need. If you are not willing to do this, then you should reconsider whether you should write a tutorial in the first place.

### Software version
Expect software to change over time. If something works for you in version 1.123 it does not mean that it will still work in 1.124. Therefore it is crucial to include the exact version of software involved in your tutorial. It will be difficult to reproduce the tutorial if this information is missing. No one expects you to update your tutorial daily, so having at least the software versions of when it did work is useful.

### Explain what you are doing and why
At the beginning of your tutorial, briefly describe the goal of the tutorial. Additionally, keep in mind that giving your readers copy-pasteable code that works is nice, but taking the time to write a sentence or two that explain what the code pieces are doing and why they are needed will increase the quality of your tutorial.

### Test your own tutorial
Go ahead and test your own tutorial before you publish it. This might seem obvious, but it truly seems as if barely anyone is doing this. The amount of times I encountered errors due to typos in provided code is astounding. Remember the guy who replied to your recent Stackoverflow question with: "I haven't tried it myself, but \<code-that-does-not-work\> should work."? Don't be that guy.

## Final words
Congratulations, you know understand how to write a decent tutorial (in theory). Now go ahead and do it!