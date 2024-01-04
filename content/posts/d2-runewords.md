---
title: "C++ runeword calculator GUI for Diablo 2 Resurrected"
date: 2023-07-27T22:50:00+02:00
description: Uses wxWidgets and has been tested on Windows, macOS and Linux.
draft: false
tags: [c++, wxwidgets, gaming, programming, windows]
---

## Introduction

Diablo 2 Resurrected is one of my favorite games of all time, but there are a lot of things you need to remember. There are various online tools (the idea of a runeword calculator is not new), but many of them have a terrible User Interface, are outdated, or are located on slow websites that autoplay Twitch streams when you open them. So I decided to learn how to use wxWidgets by making an assistant GUI tool for my favorite game! I have open-sourced this tool, and you can download it from this [Github repo](https://github.com/downIoads/cpp_diablo2r_runewordCalculator/tree/main). The repo contains the source code and you can build it yourself. The tool has been tested on Windows 11, Ubuntu 22.04.3 LTS and macOS Ventura and Sonoma. It also is available for 4.99€ on the macOS App Store (best option if you are lazy).

## Screenshot

### Windows 11
![targets](/images/d2r.png "Windows 11")

### macOS Ventura
![targets](/images/osx-d2rrunes.png "macOS Ventura")

## Features
These features are currently supported:

1. Calculates possible runewords (you select which runes you have, which item type base, how many sockets, and optionally a level requirement, and it will list all possible runewords you can make)

2. Breakpoints for all classes (IBS, FCR, FHR). Attack speed is currently not supported, but I can recommend https://d2.lc/IAS/

3. List of runewords (in case you know the name of a runeword and want to check what it does)

4. Rune conversions (hover your mouse over a rune, and it will tell you how to upgrade this rune, what this rune is upgraded from, and also which effect this rune has when socketed)

5. Sort Runes A-Z or by rarity (sometimes by rarity is better, but sometimes you just want to know how to upgrade ITH in the cube, and you know where it is alphabetically, but you might not be able to find it quickly if the runes are sorted by rarity)

6. Rune word information (lists the exact details of every runeword, if a runeword has different effects for different item types, these are listed too)

## Future update ideas

1. Dark Mode (gaming at night is not a good idea, but if you do it you might as well have a nice dark mode GUI)

2. Remove sounds (the default info boxes that open when you click stuff play an annoying sound, but I did not find a simple way to disable sounds). If you know how to do this without writing custom classes, please let me know!). A workaround you can use on Windows is right-clicking the Sound symbol and choose "Open volume mixer", there just put volume to 0 and it will remember this setting

3. High DPI support by default (this can be done, but it currently seems to be fairly complicated with wxWidgets). A simple workaround is to create a shortcut to the .exe file, and then right-click the shortcut -> Properties ->  Compatability -> Change high DPI settings -> Check the box for "Override high DPI scaling behavior" and select "Application".)

4. More features (it is called a runeword calculator, but in the years I added breakpoints too. So I might as well add stuff like cube receipes for caster gear crafting)

## Summary

I have used this tool over the years and added features that I personally wanted to use. I learned a lot in the process (about wxWidgets, C++, and the game itself), so I would recommend anyone do something similar for their favorite game! Just think about what features you would like to have and start coding.