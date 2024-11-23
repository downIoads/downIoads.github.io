---
title: How to install ntfs-3g on macOS using brew
date: 2024-11-22T21:50:00+02:00
description: Brew works nicely, until it doesn’t.
draft: false
tags: [c++, tutorial, macos, ntfs-3g]
---

## Introduction

Life could be so easy. Want to install ntfs-3g on your new M4 Mac Mini? You might be naive and think you can just run:
```bash
brew install ntfs-3g
```
But wait, it does not work. It says
```bash
Error: ntfs-3g has been disabled because it requires FUSE!
```
And that is because Homebrew has disabled any formulae that rely on FUSE. You could now install an unofficial tab like “gromgit/homebrew-fuse” but do you trust unofficial tabs? Since this solution would be boring, this tutorial is dedicated to building ntfs-3g from source on a modern Mac.

## Dependencies
Instead of going error-by-error and showing how to fix it, I will just provide all the prerequisites you need:

```bash
xcode-select --install
	
brew install autoconf automake libtool libgcrypt pkg-config gettext
	
brew link --force gettext
	
brew install --cask macfuse
```

## Building ntfs-3g from source
First get the official source code:
```bash
git clone https://github.com/tuxera/ntfs-3g

cd ntfs-3g
```

Now generate the configure script:
```bash
autoreconf -fvi
```

Before building the installer, first we need to set an environment variable. Then we can build the installer and configure it so that it uses our brew installation of macfuse:
```bash
export SDKROOT=$(xcrun --sdk macosx --show-sdk-path)

./configure --prefix=/usr/local --with-fuse=external
```
The previous command might have shown you a warning that you should use some recommended flag so that you can boot from NTFS devices, but the issue with this is that brew does not allow you to build macfuse with that option enabled! You also cannot build macfuse from source yourself with that option enabled because the latest version is not open-source anymore! I could now go into detail about the [drama behind osxfuse and macfuse and why they are not open-source anymore](https://colatkinson.site/macos/fuse/2019/09/29/osxfuse/), but someone already did that for me. So let’s just ignore the warning (it’s an exotic use-case anyways) and continue with trying to build ntfs-3g. 

Before we can just type ‘make’ we need to change some things in the source code, because macOS does not allow access to certain locations (even when csrutil is disabled), so let’s change relevant paths. Basically, any mention of
```bash
rootlibdir = /lib
rootbindir = /bin
```
must be changed to 
```bash
rootlibdir = /usr/local/lib
rootbindir = ${exec_prefix}/bin
```
I decided to find and adjust the above lines in all the following files (where ‘.’ is the root of the ntfs-3g folder):
```
./Makefile
./include/Makefile
./include/fuse-lite/Makefile
./include/ntfs-3g/Makefile
./libfuse-lite/Makefile
./libntfs-3g/Makefile
./ntfsprogs/Makefile
./src/Makefile
```
But it might not be necessary to do it for every file. Anyway, after you have done that you can build and install ntfs-3g using:
```bash
make

sudo make install
```
If there are no errors, congrats! The binary ntfs-3g should now be located at ``` /usr/local/bin/ntfs-3g ```. There was no tutorial for any of the above info btw, this was just me fixing every single error that stopped me from building ntfs-3g on my new M4 Mac Mini.

## Using ntfs-3g to write to NTFS devices
I only have up to one NTFS devices connected at a time, so I only need to create one mounting point:
```bash
sudo mkdir /Volumes/NTFS
```
This only has to be done once. Now, whenever you have connected an NTFS device, find out where it is mounted and unmount it. Do not use the Finder Eject symbol to unmount it; otherwise you won’t be able to see the devices anymore via ‘diskutil list’ and we need to do that to determine its path. Instead use:
```bash
diskutil list

diskutil unmountDisk /dev/disk4
```
Replace ‘disk4’ with whatever it is in your case. For the rest of the tutorial, I will assume it was disk4. You should for now remember what is shown as ‘IDENTIFIER’ of the entry that has type ‘Windows_NTFS’, because it will be needed later. In my case, it was ‘disk4s1’.

Now use ntfs-3g to mount the device in read/write mode:
```bash
sudo ntfs-3g /dev/disk4s1 /Volumes/NTFS -o auto_xattr -o volname=MyNTFS
```
The ‘-o auto_xattr’ is super important, because without it there would be a bug where all files are invisible in Finder and only viewable via the terminal. Shoutouts to [lezgomatt](https://github.com/macfuse/macfuse/issues/574#issuecomment-541309529) for finding this fix! Kinda crazy that this has been known for 5 years and still not fixed by ntfs-3g devs, but who am I to complain? You can replace ‘MyNTFS’ with anything you like btw, this is what is shown in Finder as the name of the mounted device.

You can now use Finder to write to or delete from that device. When you are done and want to unmount it, either use Finder (but then you won’t see it via the ‘diskutil list’ anymore until it physically has been removed and reconnected) or you just run:
```bash
diskutil unmount /Volumes/NTFS
```
which will always work, no matter which path your device had earlier. 

## Unmounting the device sometimes doesn’t work; help!
MacOS does lots of indexing on externally connected devices, which means you try to unmount them via Finder, but it just won’t work, and you are not sure why. You can disable indexing for a device by storing a file called “.metadata_never_index” in the root folder of the device. This should greatly decrease the amount of times you can’t unmount via Finder. Should it still happen, just find out what uses the devices via:
```bash
sudo lsof | grep /Volumes/NTFS
```
and then kill each number you see like, e.g.:
```bash
sudo kill -9 <number>
```
where you replace \<number\> with each shown number. Then try unmounting again, and it should work.

## Conclusion
Half my weekend gone, just because I was stupid enough to think
```bash
* My Apple device would just work
* Installing sth like ntfs-3g could easily be done via brew
* After installing ntfs-3g I would be able to see the files on the device without having to employ exotic fixes like '-o auto_xattr'
```
Anyway, I learned a lot, and I hope you (yes, you!) did too.