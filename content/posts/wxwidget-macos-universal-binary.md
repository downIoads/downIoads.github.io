---
title: "Wxwidgets macOS guide to compile universal binaries"
date: 2023-10-25T14:00:23+02:00
description: A tutorial that actually works.
draft: false
tags: [tutorial, macOS, wxwidgets]
---

## Introduction

I wanted to publish my wxwidgets project "D2R Runeword Calculator" which I already blogged about, to the macOS App Store. In order to do this with wxwidgets, you are advised to create statically-linked binaries that run both on x86_64 and arm64. But the instructions in the wxwidgets tutorials are outdated and lack crucial information. It took me a few hours, but I figured out how it can be done on macOS Ventura (13.6). The wxwidgets version I will be using is 3.2.3, which was released on October 10, 2023.

## Guide

### Step 1 - Acquire latest source code of wxwidgets

Just go to their [website](https://www.wxwidgets.org/downloads/) and download the latest stable "Source for Linux, macOS, etc". Then unpack it.

### Step 2 - Building

It took me lots of tries to find a command that would work but there it is:

```zsh
cd wxWidgets-3.2.3 && mkdir build-cocoa-static && cd build-cocoa-static && ../configure CXXFLAGS="-std=c++17" --enable-debug --disable-shared --with-osx_cocoa --with-libiconv --with-macosx-version-min=13.6 --enable-macosx_arch=x86_64,arm64 && make -j
```

This will take a long time, but it ensures that you get statically linked binaries that target both x86_64 and arm64 macOS. If you want to use a different C++ standard than 17, feel free to remove the part CXXFLAGS="-std=c++17" (I don't think it's needed).

### Step 3 - Installing

After the previous command terminates, just one more command is required to install wxwidgets:

```zsh
sudo make install
```

### Step 4 - Compiling your project

Now just cd to your project folder and run:

```zsh
g++ -arch x86_64 -arch arm64 -mmacosx-version-min=13.6 -std=c++17 *.cpp -w `wx-config --cxxflags --libs` -o myExecutable
```

The parameters "-arch x86_64 -arch arm64" ensure universal compilation. I kept getting warnings about some .dylibs (dynamic libaries) which are irrelevant, so I added "-w" to silence these warnings. I used "-mmacosx-version-min=13.6" because I am running Ventura 13.6 and the value here should be the same one as you used in the configure command; otherwise, you keep getting warnings about how it's linked against 13.0 or whatever. I use "-std=c++17" because that is the C++ version I am targeting.

You can verify that your binary works on x86_64 and arm64 by using the command:

```zsh
lipo -archs <yourBinary>
```

## Conclusion

This seems like a short and simple tutorial, but it is extremely frustrating to get to the point where things are working as they should. The wxwidgets tutorials are mostly outdated or lack crucial information. Then Brew baited me into trying that approach, but it kept ignoring my edits (the default is dynamic linking, which I don't want), and even after everything was set up it was not clear to me that -arch arm64 needs to be added to the command. I am glad that everything is working now, and the next steps are packaging the binary into a .App file that can be published on the App Store. I already noticed that the wxwidgets tutorial for building on XCode is already outdated, so I will see how that goes and maybe write another post after I am successful.