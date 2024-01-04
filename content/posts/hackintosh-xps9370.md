---
title: "Hackintosh #3: Running Sonoma on XPS 9370 (i7 8550u - 4k display)"
date: 2023-10-24T08:00:23+02:00
description: Quick rundown of my third Hackintosh setup, what works and what does not not.
draft: false
tags: [hackintosh, macos]
---

## Introduction
macOS Sonoma is the newest macOS version (which always is a challenge for Hackintosh) and installing it on an older laptop is an extra challenge. In this post I will talk about how I successfully hackintoshed the XPS 9370. Note that currently online installers do not work for Sonoma, so you require an Apple machine or a Hackintosh to create an offline installer (first you create a FAT32 partition called EFI (Size around 200MB), then you make an Mac OS Extended (Journaled) partition called MyVolume, then you follow [this Apple guide](https://support.apple.com/en-mide/HT201372)).

## Spoiler: What works?
Before you continue reading, you probably want to know whether the effort will be worth it. So here is what works and what does not.

### Working
* 4k resolution
* Trackpad (not as nice as an actual MacBook trackpad but good enough. Gestures work decently)
* Ethernet with USB-adapter (I tried two different ones and both worked)
* Bluetooth (e.g. Audio)
* SD card reader
* Keyboard Backlight
* Battery Percentage
* Wi-Fi with TP-Link Archer 3U dongle and Chris1111 BigSur repo
* Fn Buttons (except shortcut for brightness)
* Sound via speakers (thanks to [SSDTTime](https://github.com/corpnewt/SSDTTime) with 1.FixHPET and [CodecCommander.kext](https://bitbucket.org/RehabMan/os-x-eapd-codec-commander/downloads/))

### Not working
* Sound via AUX
* Thunderbolt ports rarely work, but they sometimes do
* Sleep mode (disable in settings and try not to click it accidently under Apple logo top-left drop-down)

Sleep mode is right above Shutdown or Reboot (not sure right now) in the Apple logo drop-down menu. If I accidently press it it goes into sleep mode, but when trying to leave sleep mode to continue working it gets stuck on the lockscreen:

![targets](/images/xps-hackintosh/sleep.png "Sleep mode is broken")

### Not tested
* Touchscreen (I never liked this, so I disabled it in the BIOS)
* Camera (I never use this, so I disabled it in the BIOS)
* Microphone (I never use this, so I disabled it in the BIOS)

All in all, I got everything except non-bluetooth sound working so far. 
4k resolutions works out-of-the-box, but it is called (1920x1080 (default)). There also is 1920x1080 (low resolution) but this is actual 1080p and looks slightly blurry on this 4k display. The former is 4k with upscaling so that everything is nice, sharp and big.
The XPS has two Thunderbolt 3 ports on the left side and one "normal" USB-C port on the right side. The right side port works perfectly well with every device I have tested. The thunderbolt ports rarely recognize anything, but I did manage to have my Nintendo Switch Ethernet adapter working on these ports. I was able to plug it in while the OS was already running, and it worked after a few seconds. I have read from others that their Thunderbolt devices work if the USB device was already attached to them before booting, but for me this did not help with anything. So basically you are left with one reliable USB-C port on the right side and should use one of the left side Thunderbolt ports for your charger (this always works). I recommend attaching a USB-hub with many ports (including Ethernet) and maybe even active power to the right-side USB port.
Keyboard backlight is working when using the same kexts as I do, not much to say about this.
Surprisingly, Bluetooth works out of the box with my kexts. So the built-in Killer Wi-Fi card (which sadly is soldered and thus can't be replaced) can be used for Bluetooth, but nothing Wi-Fi related will work with it. Bluetooth works perfectly with my Shure headset, even all buttons on it work!
In order to get Wi-Fi working, I recommend getting a TP-Link Archer 3U dongle (Chris uses that one too and who would know better than him) or the large antenna version of this dongle (also tested by Chris). Then just follow the steps described in [Chris repo](https://github.com/chris1111/Wireless-USB-Big-Sur-Adapter). Note that you can not re-enable csrutil or spctl master after the install as this will break your install, and you will have to re-install. So just follow his video tutorial, then Wi-Fi will work perfectly well as long as you attach the dongle to the right-side USB slot. In short: Go to recovery mode, "csrutil disable && spctl --master-disable", reboot, install the stuff and allow it, reboot.
Battery percentage works if you enable it in macOS.
Fn buttons work, but macOS does not by default allow you to use them as a shortcut for brightness adjustment. So the best workaround is to bind lower brightness to F2 and increase brightness to F3. This way, you use F2 for lower brightness and Fn+F2 for lower audio, F3 for higher brightness and Fn+F3 for higher audio.
Getting sound to work was difficult. First I did the layout-id mapping with help of [gfxutil](https://github.com/acidanthera/gfxutil/releases) in the config.plist (cleaner than alcid boot-arg), then I used SSDTTime to fix IRQ conflicts. This allowed me to adjust sound, but I never got any sound. This only changed after I added the CodecCommander kext. Now it is working perfectly, but sadly AUX is still not working. Many others online had CPU usage issues when using AppleALC and CodecCommander kexts together, but I can't confirm this. My CPU usage is low and the battery is not drained as far as I can tell. So either I got lucky or they fixed it in later versions of the kexts.


## Overview of ACPI, Drivers and Kext
Of course, you first do a USB port mapping on Windows before you do anything. Then you follow the standard Dortania tutorial for Kaby Lake. The i7-8550u is a Kaby Lake R generation CPU (which is sometimes referred to as Kaby Lake in the tutorial). As SMBIOS you use MacBookPro14,2 (Ventura) or MacBookPro16,3 (Sonoma). Note that Sonoma requires 15,x +. After following the tutorial, there are two more things you must do to get it to install (possibly only required for 4k display versions of this laptop):
* In your config.plist change "DevirtualiseMmio" to True
* In your config.plist add the following boot-arg: -igfxmpc

Also note that when doing the recommend BIOS settings, Dell has some weird names for some things. So just google what the Dell equivalent for x is etc.
After trying lots of different things, the following setup ended up working best for me:
### ACPI
* SSDT-EC-USBX-LAPTOP
* SSDT-PLUG-DRTNIA
* SSDT-PNLF.aml
* SSDT-HPET.aml (created with SSDTTime and 1.FixHPET)

### Drivers
* HfsPlus.efi
* OpenRuntime.efi
* (ResetNvramEntry.efi) [I have this one but I don't think you need it]

### Kexts
* AirportItlwm.kext (v2.3.0-alpha or newer) [not sure if this does anything with my dongle setup]
* AppleALC.kext
* BlueToolFixup.kext
* BrightnessKeys.kext
* CodecCommander.kext
* IntelBluetoothFirmware.kext
* IntelBTPatcher.kext
* Lilu.kext
* NVMeFix.kext (technically the Toshiba 512GB NVME is supported but it is recommended to use this kext anyways)
* Sinetek-rtsx.kext (fixes SD card reader but after ejecting you must reboot to recognize the same card again)
* SMCBatteryManager.kext
* SMCDellSensors.kext
* SMCProcessor.kext
* SMCSuperIO.kext
* USBToolBox.kext
* UTBMap.kext
* VirtualSMC.kext
* VoodooI2C.kext
* VoodooI2CHID.kext
* VoodooPS2Controller.kext
* WhateverGreen.kext

Generally, try to get the newest version unless there is a specific reason why you should not. I even used the debug versions of everything even though that is not needed.

## Conclusion
I got the newest macOS version running on a 5-year-old laptop and everything is smooth. The touchpad can be annoying because it is just not that great and the battery could be better too. But the OS runs well and stable, 4k is working, and I can actually use it as a productive machine. Wi-Fi works with the dongle thanks to Chris amazing work, and I hope I can figure out how to make aux sound work too.