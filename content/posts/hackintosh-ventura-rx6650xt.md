---
title: "Hackintosh #2: Running Ventura using Haswell CPU and a spoofed AMD RX 6650 XT"
date: 2023-10-10T08:00:23+02:00
description: Quick rundown of my second Hackintosh setup and what I learned in the process.
draft: false
tags: [hackintosh, macos]
---

## Introduction
In my [previous post]({{< ref "hackintosh-project1.md" >}}), I wrote about how I used the following hardware to run Monterey (an older macOS version):
* Intel i5 4400 (Haswell)
* MSI B85-G41
* Intel AX210 BT/Wi-Fi

As you can see, this build did not have a dGPU. For this reason, I bought an AMD RX 6650 XT and added it to this computer. This GPU officially is not supported either (so it would have been easier to get an AMD RX 6600 instead), but you can do GPU ID spoofing to make it work. I preferred the 6650 XT over the 6600 because in my country both GPUs currently are very cheap, and these two models had the same price! The 6650 XT is slightly more powerful and allows me to learn how GPU spoofing works, so the decision was easy.

## Setup struggles and tips
The main difficulty for me was determining the PCIE path of the GPU (it needs to be defined in ACPI) and compiling the SSDT-BRG0.aml correctly so that the GPU can be found. Assuming you have a macOS installation (without detected GPU), the correct approach is to use [Gfxutil](https://github.com/acidanthera/gfxutil) to get the GPU device path. Then you pipe this path into [SSDTTime](https://github.com/corpnewt/SSDTTime)'s PCI Bridge function along with the system DSDT, and it will generate any needed bridges for you. Later for spoofing, you need to know how to edit the config.plist (EF730000 stands for RX 6650 XT, so you spoof it to FF730000 which stands for RX 6600 which is officially supported).

The reason why I struggled is that I tried to follow this [Dortania guide](https://dortania.github.io/Getting-Started-With-ACPI/Universal/spoof.html) when I should have followed these more [up-to-date instructions](https://www.tonymacx86.com/threads/success-amd-rx6000-series-working-in-macos.306736/post-2299282) by user @tedyun. The two files from these two approaches are slightly different, and I found the latter easier to follow thanks to the several screenshots that tedyun provided. This finally allowed my GPU to be recognized, which greatly increased the performance (when transparency starts working and everything is smooth, you know you did something correctly) of my machine.

Another difficulty for me was that my original plan was to install Sonoma instead of Ventura. But Sonoma kept rebooting into the BIOS as soon as the GUI for the macOS installer tried to open. When I tried Ventura, it did allow me to install it (even though at that time there still were many problems with my setup that I later fixed). So my idea was if Sonoma won't let me install, then just install Ventura and then in Ventura do the upgrade to Sonoma. But Ventura ended up working so well that I did not bother with Sonoma yet (maybe there will be a third blog post about this). As long as I can use the newest Xcode version and target the latest iOS versions, I am happy, and Ventura currently allows me to do just that.

I also noticed that my EFI partition was gone (I upgrade from Monterey to Ventura instead of doing a clean install). So I ended up using the built-in Disk Partition tool to shrink the current partition (while it's running, pretty cool that this even works) by a few hundred MB to have space for an MS-DOS (FAT) partition. Then just put the EFI folder in there, and you don't need a USB stick to boot anymore! In OpenCore boot selection just Ctrl+Enter the boot entry that should be default and in the config.plist remove -v to get the nice Apple boot logo and reduce OC timeout from 5 to like 2 seconds for a faster boot.

Last but not least, I initially struggled to find out which SMBIOS to target. Especially with Haswell this is complicated, because technically Haswell iGPU is only supported up to Monterey and not beyond that. But with the AMD GPU I can disable the iGPU of the CPU and only use the dGPU. So there ended up being slightly misleading and conflicting information about this specific setup in the Dortania guide, but in the end the best play is to go for MacPro7,1. And I can confirm this runs perfectly stable, but I still like to avoid things like sleep mode, which always can be problematic. Also don't forget to read through [WhateverGreen](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md)'s repo so that you know how to switch the Haswell iGPU to headless mode (you need to put AAPL,ig-platform-id 04001204) since it is not supported in the newest macOS versions.

## Overview of ACPI, Drivers and Kext
After trying lots of different things, the following setup ended up working best for me:
### ACPI
* SSDT-BRG0.aml (creating this was difficult)
* SSDT-Bridge.aml (creating this was difficult too)
* SSDT-EC-DESKTOP.aml
* SSDT-PLUG-DRTNIA.aml

### Drivers
* HfsPlus.efi
* OpenRuntime.efi

### Kexts
* AirportItlwm.kext (v2.3.0-alpha was released today and is the first version that made Wi-Fi and Bluetooth work for me on Ventura)
* AppleALC.kext
* BlueToolFixup.kext
* IntelBluetoothFirmware.kext
* IntelBTPatcher.kext
* Lilu.kext
* RadeonSensor.kext
* RealtekRTL8111.kext
* SMCProcessor.kext
* SMCRadeonGPU.kext
* SMCSuperIO.kext
* USBToolBox.kext (you have to do your own port mapping)
* UTBMap.kext
* VirtualSMC.kext
* WhateverGreen.kext

Generally, try to get the newest version unless there is a specific reason why you should not.

## Short MacOS shortcut rant
I generally like macOS a lot, but I disagree with trying to re-invent the keyboard and common shortcuts. The equivalent of Windows Alt+Tab on macOS is Ctrl+Tab. The equivalent of Windows Ctrl+X on macOS is Ctrl+C and then Ctrl+Alt+V. Instead of cancelling a process in the terminal with Ctrl+C you have to use Ctrl+. in zsh. Instead of pressing the Windows key to get a search, you have to do Ctrl+Space in macOS. This list could go on and on, I really have to re-learn my muscle memory. :)

## Conclusion
I am glad that I went for the RX 6650 XT instead of the RX 6600 because in the process I learned a lot more about various tools and techniques in the Hackintosh scene. Wi-Fi and Bluetooth are working, Sound is working, Xcode 15 is able to install a hello world app on my iPhone... I am very happy with this setup! It was frustrating for a few hours, but I managed to get the GPU recognized and hardware acceleration working before going to bed on the same day. Thanks to @corpnewt for patiently supporting me and answering my questions on Discord!