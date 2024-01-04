---
title: "Mine your username"
date: 2023-07-03T21:50:00+02:00
description: Multithreaded C++ username miner.
draft: false
tags: [c++, multithreading, hashing, programming]
---

## Introduction

So you wanted to sign up for a website but couldn't think of a cool username that wasn't already taken? No way, I was in that same situation! So I decided to write a C++ program that takes your base name (e.g., "cafe") and adds a few numbers. The idea is to generate usernames that, when hashed using SHA256 or KECCAK256 result in strings that contain your base name again! I also added a mode where the requirement is that your base name must be included at least twice in both the SHA256 and the KECCAK256 output. The code of the resulting program is available at this [Github repo](https://github.com/downIoads/cpp_username_miner/). It goes without saying that your base word must be hex-compatible (e.g. cafe, decaf, decade or facade), otherwise it can't possibly be included in the hash output!

## Features

* Supports two modes. Mode 'false' only checks that your word is included in both the SHA and the KECCAK hash. Mode 'true' also requires multiple occurrences of your word in both hashes. This problem is much more difficult.
* Multithreading: The program checks how many threads your CPU supports, splits the input space evenly (e.g., you set upper limit to 16 and you have 8 threads, now thread 1 will check [0,1], thread 2 will check [2,3], ...). I found that 100 million was a decent upper limit for my CPU-only setup inside a VM. This way, each search takes around 120 to 150 seconds.

## Which cool usernames have you found so far?
So far, I like "bed147373429" the most. Its SHA256 is "f4a08820f996178982f697079d404b57a006720557bbb7dbed22bedfcaa9d2c5" and its KECCAK256 is "d6de1963ded05191276e9b9f06dfb99c8d832bed4bed31e574e8197593dd8ed9". So not only does "bed" occur twice in both hashes, but the occurrences are also very close to each other! Do you know how in Math all kinds of different numbers have names like e.g., happy numbers or unhappy numbers? Imagine a world where finding these weird hash inputs is like a sports. You could create a scoring ladder based on word length (which greatly increases difficulty), amount of occurrences (same here), closeness of occurrences, amount of popular hash functions where these occurrences happen and give points for everything. You might even give names to certain constellations, like the mathematicians do!

## Final words
I am currently learning C++ and reading through text is way less effective for learning than thinking of silly projects and implementing them. So here I learned about simple multithreading and how to incorporate commonly used cryptographic hashing algorithms into your program. Let me know if you find any cool usernames! You can find my Github Discussion page [here](https://github.com/downIoads/downIoads.github.io/discussions).