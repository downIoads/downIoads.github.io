---
title: "Running Windows 98 SE in Vmware Workstation on modern hardware"
date: 2024-02-13T14:00:23+02:00
description: It works if you work around a few issues.
draft: false
tags: [tutorial, windows, windows 98, vmware]
---

## Introduction

Remember the good old days when your OS did not annoy you with ads or features you never asked for? When everything seemed minimalistic and lightweight, and it did not feel as if every mouse movement you make gets data mined and sent to Microsoft servers? I found an old Win98 SE (Second Edition, 4.10.2222A) install disc, created an image and tried to set it up using VMware Workstation 16. This blog post is about the general setup and a few tricks to make it work. It is not as easy to spin up as let's say a Win XP VM, but when you get it working it runs flawlessly (except that there is no 3D acceleration with stops some but not all games from running).

## VM settings and getting past the installer

There are a few things to keep in mind when configuring the settings of the VM itself, as the VMware/Win 98 combi has a few restrictions:
* Max 1 CPU / 1 Core per CPU
* Max 1 GB of RAM
* Less than 128 GB of storage
* Set USB mode to 1.1 or 2.0 (the latter requires a fix also described in this post)
* All virtualization options disabled

Not sure if that is necessary, but when setting up the VM I chose "Custom" and then set VMware's compatibility mode to VMware 5.x (as new versions of workstations are not officially supported, but maybe they just mean the 3D acceleration driver).

When you start your VM, you choose "Boot from CD" and then "Boot from this CD-ROM". Now, if you have a modern AMD CPU (Ryzen 3000+) or Intel CPU (11 Gen or newer) like me, the installer will crash due to a well-known [TLB invalidation bug](https://blog.stuffedcow.net/2015/08/win9x-tlb-invalidation-bug/). The workaround is to use [patcher9x on GitHub](https://github.com/JHRobotics/patcher9x) to patch the VMM.VXD driver. All you need to do is download the [floppy image](https://github.com/JHRobotics/patcher9x/releases/download/v0.8.50/patcher9x-0.8.50-boot.ima) and pass it to the VM by adding a floppy disk in the VM settings. When you then restart the VM, just enter "patcher9x" followed by pressing enter a few times and confirming with "y" and the bug is fixed! It is great to see an active community that builds simple, open-source tools to fix these niche problems. Now the VM will install without crashing. 

## Installing VMware tools to get support of high resolutions and true color mode

Even though not listed on their website, the file "winPre2k.iso" is a working vmware-tools installer for Windows 97, and it still exists in the installation folder of Vmware Workstation's modern version and can be passed to the VM via the CD-Rom ISO option. Windows then will install it and allow you to set high resolutions (my monitor maxes out at 1080p and that worked perfectly). Generally, since copy-paste between Host and Guest is not working (even with vmware-tools) and it also does not support folder sharing, at first the only option to pass data from the host to the guest is via the CD-Rom ISO option. Note that you can bundle any file as ISO (e.g. text.txt and myProgram.exe) by e.g. using Ubuntu and running 
```bash
sudo apt install genisoimage

genisoimage -r -J -o ./output.iso ./<folder-to-compress>/
```
This is your best option until you followed the next step that enables USB stick support (which Windows 98 SE does not have by default).

## Getting USB stick pass-through to work

There is a nice unoffical tool called [nusb36e](https://www.philscomputerlab.com/windows-98-usb-storage-driver.html) which adds the necessary drivers to Windows 98 so that USB sticks are recognized (even USB 2.0 will be working!). So you bundle that .exe as .iso and pass it through the VM and follows its steps (remove all USB drivers and Unknown Device drivers via Device Manager before running the tool). After rebooting you will get all necessary drivers (just keep clicking Next, Next, Ok, Ok, Skip if necessary due to error that occurs) and you will be able to pass-through your USB stick to the VM, and it will be recognized and usable if you make sure to format it to be FAT32. I think modern Windows does not even list FAT32 as a formatting option these days, so I recommend you use [Rufus](https://rufus.ie/en/) to do that. Now you are able to pass data from the VM back to the host! Before I tried this, I tried nusb33e, and I could not get it to work, so I recommend using nusb36e.

## Screenshots

The main thing that motivated me to do this afternoon project is to run the best version of Paint. I greatly dislike modern versions of it, and last time I tried using it on Windows 11 it failed on me. The good old Win 98 Paint on the other hand just works :)

![targets](/images/win98/win98_paint.png "Paint is an amazing tool")

![targets](/images/win98/win98_usb_working.png "My USB stick is now usable")

## Summary

It is much more work to get Win 98 to run than e.g. Windows XP, as tools like VMware officially do not support it anymore. But it still can be done, and once you figure out how it runs perfectly! The lack of 3D acceleration with the driver that winPre2k.iso provides only becomes noticeable when you try to play old games that rely on 3D acceleration, but there are plenty of games that will work without it. The only thing I would like to see in the future is a fix on how to enable copy-and-paste between guest and host, but even without it the USB stick pass-through is a decent workaround. All in all, a nice project and a beautifully simple OS.