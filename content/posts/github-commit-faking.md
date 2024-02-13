---
title: "How to fake GitHub commits"
date: 2024-01-03T14:00:23+02:00
description: This article explains how to make it seem as if famous devs contributed to your repo.
draft: false
tags: [tutorial, security, git]
---

## Introduction

In Git, you have to set your E-Mail address using the command:
```bash
"git config --global user.email myEmail@example.com"
```
The problem is that when you now commit to a GitHub repository, GitHub will map this E-Mail to a GitHub account (if it exists) and then show the commit as if that account would have committed it. This has two direct implications:
* You can find out the GitHub account of people you know the E-Mail of (security problem)
* You can use this technique to fake contribute to your repo to make it seem more trustworthy

The E-Mail addresses of popular devs are often known from mailing lists etc. or can be found with a simple Google search. Look at the following [example](https://github.com/downIoads/downIoads.github.io/graphs/contributors) (purely as a Proof-of-Concept), here you can see who "contributed" to my blog:
![targets](/images/github-fake-commit.png "Commit overview of my blog")

As you can see, it looks as if Notch (Minecraft), Linus Torvalds (Linux) and Tim Berners-Lee all contributed to my blog. There is no warning about unverified commit signatures and GitHub links to their actual profiles. Verfied commits do exist on GitHub (green checkmark) but the lack of verified commit does not necessarily raise suspicion. In fact, even signed commits [could be faked](https://iter.ca/post/gh-sig-pwn/) for some time.
The only way to tell whether the commit is real or not is to look at the recent activity on their profiles (where these fake contributions do not show up). I think it should not be that easy to spoof contributions. The next time you see an open source projects with 2000 contributors and a lot of stars (these can easily be faked too by creating accounts / pay services) better look at the actual source code instead of "trusting" the repo because it appears to be popular. It's trivial to write a script that uses a list of common E-Mail addresses (that might be used by someone on GitHub) to bloat the contributors number. IMO, GitHub should enable "verified commits" by default to make this strategy infeasible: I generate a private key and add the corresponding public key to my GitHub account (which is linked to MY e-mail address), now when I push changes it's trivial to check whether my Git E-Mail identify is equal to that of my account or not. While I do understand that it should be possible to push commits of someone else, there must be a solution to this problem as the current situation is not acceptable IMO.

## Conclusion

GitHub contributions can easily be spoofed. Do not trust repos that appear popular, always check the code yourself. If you checked the code, and it looks harmless, do not assume that the binaries in the Releases section are trustworthy too. I stumbled on this issue by accident, but of course it was already known. It seems that GitHub does not view this as a critical security problem, but IMO the current situation is unacceptable.
