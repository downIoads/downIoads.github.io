---
title: "Hackintosh in 2023"
date: 2023-10-02T08:00:23+02:00
description: My personal experience with setting up a Hackintosh and using macOS for the first time.
draft: false
tags: [hackintosh, macos]
---

## Introduction
I always enjoy testing different tech-related things, such as different OSes. Having used Windows, Linux, iOS and Android, I thought it was about time I tried macOS for the first time. I prefer not to be trapped within Apple's walled garden, so instead of purchasing a Mac, I tried to give new life to old hardware I still had laying around. In the context of Hackintosh, it is important to know which hardware is supported for what. So first of all, here is an overview of the hardware I used:
* Intel i5 4400 (Haswell)
* MSI B85-G41
* Intel AX210 BT/Wi-Fi

As you can see, I have not listed a dGPU (dedicated GPU). The reason for this is that my old GTX 1060 is not supported for newer Hackintosh versions (High Sierra from 2017 was the last supported version for this GPU) and the iGPU of my i5 supports up to macOS Monterey (released in 2021). If you are new to the Hackintosh scene I think it is good that I provide a few important links:
* [Dortania](https://dortania.github.io/OpenCore-Install-Guide/) is best guide for anything Hackintosh related
* [Hackintosh Paradise Discord](https://discord.gg/8aKs69x) server mainly for Intel CPU users, [AMD Hackintosh Discord](https://discord.gg/EfCYAJW) server for AMD CPU users [if any links are dead, check /r/hackintosh sidebar]
* [USB Mapping Tool](https://github.com/USBToolBox/tool)
Any other information can be found from there. So let's start with my personal experience of setting everything up. My goal was to set up a Hackintosh that runs macOS Monterey because this is the last version that Haswell CPUs can run (unless you have a dGPU that can support a newer version like Sonoma). I will make a follow-up post to this one, where I use a RX 6650 XT in combination with my Haswell CPU to run macOS Sonoma.

---

## Setup
Everyone has different hardware (not really, but you know what I mean), so you will need to apply different patches and fixes to make your system work. Also you must do USB mapping because macOS seems to be strict about the number of ports being used and what they are used for. When you read the Dortania Guide, you will see that three things are required to make your hardware work:
* Kext (.kext)
* Firmware Drivers (.efi)
* SSDTs (.aml, can be found pre-built for different CPU generations)

After gathering all the required files (depending on which macOS version you target and which hardware you have), you need to make quite a few changes to the config.plist. This file basically describes how the PC should boot (you define custom flags) and how macOS will see your hardware (sometimes you must spoof the ID of some hardware to the ID of similar hardware that is by default supported in macOS). I think finding the correct values for the config.plist the hardest part of setting up a Hackintosh. With the Kext, Firmware Drivers and SSDTs it is pretty clear which versions you must choose in which situations. But with the config.plist there are a lot of things that are not clear when you are new or tradeoffs between compatibility and performance, so you must play around with it if your Hackintosh does not work as it should. For instance, I found a kext that supports my Intel AX210 Wi-Fi/BT chip, but AirDrop still will not work. So if you want out-of-the-box AirDrop functionality you must buy an adapter that is similar to the one used in Apple devices.

---

## Post-install - Using macOS for the first time
I have used Linux distros for many years, so I had a general idea of how to find my way around. But there were many small things that I liked and disliked, and I think it would be good to list them in random order:
### Creating a new text file
It was surprisingly difficult to do this. I was prepared because on Linux, if you don't have any templates, you will also only be able to create folders after right-clicking but not files like a simple text document. I usually "touch text.txt" into templates and then have a simple way to create new text files on Linux, but I was not able to find a templates folder on macOS. It surprised me that macOS was not user-friendly at all here, I doubt they want to have their users go to the terminal and use the touch command to create a text file. So the user-friendliest way to do this that I found was to open an application called "TextEdit" (not to be confused with the outdated Android Text User Input Widget, actually I think that was called EditText) and from there save an empty file. And since it does not allow you to choose the .txt extension, you must then manually edit it in the file explorer (but extensions might not even be shown to you, so you might have to enable that first), only to discover that TextEdit put some data in your file, and you manually delete it... So yes, at the end of the day I decided to put an empty text.txt file on my desktop, and whenever I need it, I Alt-Drag it somewhere (duplication) [actually, the key Alt is called Option in macOS].
### Save state on shutdown and restore it
I really love this macOS feature. When shutting down your system, it asks if you want everything restored to how it is right now when you boot again. It's surprising that Linux and Windows don't seem to offer it like macOS does. It's an amazing feature that most people will like.
### Paying for free things
It seems to me that macOS users are willing to pay for open-source software. If you visit, e.g., the Safari extensions store, you will see a popular extension called Dark Reader for more than $5. But this is an open-source extension that you can install for free when using Firefox or any other browser that is not Safari. Or another example: uBlock Origin is not available for Safari, but other ad blockers are allowed. So people seem to decide to pay for AdGuard Pro or Wipr (which seems to be the best uBlock Origin replacement for Safari) instead of just using a browser other than Safari and using uBlock Origin. I could list many more examples, but to keep it short, free things are sold on the App Store for money (and they are popular).
### Keyboard Compatibility
I was confused at first because shortcuts like Ctrl+C and Ctrl+V did not work. The fix seems to be to go to keyboard settings, click on Modifier Keys, and then switch the entries for "Control" and "Command" keys.
All in all, I think macOS looks beautiful and is a fun OS to use. It is more user-friendly than Linux and seems less bloated than Windows, and while it does have its unique quirks (e.g., the way you are supposed to drag and drop applications in the Applications folder to install them), I generally like using it.
### Font Rendering
I personally love the slightly blurry font rendering that macOS does. It might look bad on old 1080p monitors, but it looks amazing on 4k+ HighDPI monitors!

---

## Why bother with macOS?
I am currently learning how to make Android apps using Jetpack Compose (Kotlin) and since everything likes declarative UI nowadays, SwiftUI (Swift) seems to be accessible if you know JC. Since the only way to do iOS development is to use Xcode with its built-in iPhone simulator, my main usage of the Hackintosh will be to learn SwiftUI.

## Conclusion
Do not underestimate the amount of preparation required to make your first Hackintosh. But overall, I think it is a fun project if you have compatible hardware (check the Dortania Guide I linked to see whether your hardware is compatible) and definitely worth the time. The Hackintosh community is huge and willing to help you. Join a Discord server and start talking to people (but please do some research first). Most of the work has already been done for you, so be grateful and patient.
