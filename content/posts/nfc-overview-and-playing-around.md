---
title: "NFC in 2023"
date: 2023-07-09T14:00:23+02:00
description: Learn how to use NFC with your smartphone or dedicated NFC USB device.
draft: false
tags: [nfc, python, programming]
---

## Overview of NFC tags

NFC was invented in 2002 by Sony and NXP Semiconductors. Since then, a lot of different standards and tags have been created, so here is a basic overview of commonly used tags and their classification. To simplify things, commonly used NFC tags can usually be put into one of five forum types:
* Forum type 1 (read-only)
* Forum type 2 (usually read/write and more storage than type 1)
* Forum type 3 (mainly used in Japan)
* Forum type 4 (like type 2, but with more storage and features and a higher cost than type 2)
* Forum type 5 (highest communication range, not commonly used by consumers)

These forum types help cluster different tags into categories. Different manufacturers produce different chip series, but the most commonly used and best supported NFC tags are the NTAG series by NXP. If you just want to play around with NFC and want to buy the most compatible (e.g., with smartphones) NFC tags, you should purchase NTAG 2xx or NTAG 4xx tags. To provide a broader overview, here is an overview of the arguably most popular and most compatible NFC tag series called NTAG by NXP:
* NXP NTAG Series
  * NTAG 215 (popular because they are used for Nintendo Amiibo)
  * NTAG 216 (better than 215, but more expensive)
  * NTAG 223 / 224 (released in 2022; these are probably the best type 2 tags that exist, but I have not found a way to buy them yet)
  * NTAG 424 DNA TagTemper (released in 2018; these are the tags you want to buy if you care about encryption and are not able to find the 426Q for sale)
  * NTAG 426Q (released in 2022, probably the best type 4 tag there is)

TL;DR: Buy the NTAG 216 (good, cheap tags with great compatibility) or the NTAG 424 DNA TT (supports encryption and is marketed to be tamperproof).

There are other popular NFC tag series, such as the NXP MIFARE series (Classic 1k/4k, Ultralight, Plus S/X, DESFire EV1/2) [type 2, 4 or do not comply with forum type like the Classic], the Sony FeliCa series [type 3] and the NXP ICODE series [type  5]. There are also outdated or less popular and less compatible chips, such as the Innovision Topaz 512 [type 1] or the Kovio 2k [type 2 but read-only]. All in all, it seems as if NTAG type 2 and NTAG type 4 chips are what most readers want.

## Playing around with NFC tags

To play around with NFC, you will need:
* NFC Tags (e.g., NTAG216)
* NFC Reader (your smartphone or a dedicated reader like the ACR122U USB Reader)

I will divide this part into two sections. The first section is about playing around with the tag and your smartphone. Here I will talk about useful apps. The second section is about using a dedicated reader connected to your PC and programming your own tool for reading/writing from/to your tag.

### 1. Project: Using the smartphone (does not require NFC USB device and does not require programming skills)

NXP has two useful apps on the Google Play / Apple App Store:
* NXP TagInfo (get detailed info of the scanned tag)
* NXP TagWriter (write data to tag using your smartphone)

If you want advanced actions (e.g., a tag when scanned should open a particular file on your phone), I can recommend the paid app "NFC Tools PRO" by wakedev in combination with "NFC Tasks" by wakedev. The apps are self-explanatory and very simple to use, as long as you use NTAG tags. If you have accidently bought a Mifare Classic 1k tag, you are in trouble because, while these support NDEF, they are blank when you buy them (not formatted to use NDEF). Formatting them can be a complicated process if you did not buy them directly from NXP because they are not standardized. If you bought them directly from NXP you can just use TagWriter to format them correctly. Here are a few project ideas:
* Business Card: Put your contact info on a tag.
* Open URL: Put [any URL](https://www.youtube.com/watch?v=dQw4w9WgXcQ) on the tag, and if scanned, it will ask to be opened
* Good Night: Scan the tag to disable Wi-Fi, disable Bluetooth, enable do-not-disturb, and set an alarm for 8 hours. For complex actions like these, you can check out the paid Android app "Tasker". On iOS, you can play around with "Shortcuts".
* Mario Jump: Scan the tag to play the Mario coin sound on your device. Stick the tag to your ceiling and jump with the phone in your hand to destroy the boxes and collect the coins.

### 2. Project: Using a dedicated NFC device

If you do not want to use your smartphone but instead want to program your own tool that can read/write tags, this section is for you. I will briefly describe my setup and provide some basic Python code. Here is what you need to follow in this section:
* Windows 10/11 OS (I use Windows 11)
* NFC USB device such as the ACR122U (I use this, and it is recommended because of compatibility with existing libraries)
* Zadig ([URL](https://zadig.akeo.ie/), I use version 2.8)
* Libusb ([URL](https://github.com/libusb/libusb/releases), I use version 1.0.26)
* Python (I use version 3.11.4)
* Python library 'nfcpy' (I use version 1.0.4)

Now follow these steps:
1. First, you connect the NFC USB device to your PC and run Zadig. In Zadig click on "Options" and enable "List All Devices". Now select your NFC device from the list and make sure the arrow points to "WinUSB". Now click "Replace Driver", then disconnect your NFC USB device, restart your PC, and re-connect it. 

2. Now download libusb and extract its contents. In the folder /libusb/VS2015-x64/dll copy the file "libusb-1.0.dll" to the folder "C:\Windows\SysWOW64". Then, from the folder /libusb/VS2015-Win32/dll copy the file "libusb-1.0.dll" to the folder "C:\Windows\System32". BTW: Here, the [nfcpy documentation](https://nfcpy.readthedocs.io/en/latest/topics/get-started.html) currently has an error, because they tell you to copy the 64-bit file to the 32-bit directory and the 32-bit file to the 64-bit directory. What a great opportunity to link to my post about [How to write a tutorial](../how-to-write-a-tutorial)!

3. Now you can install Python (make sure to tick the box to add Python to PATH when installing it) and install the 'nfcpy' library by opening CMD and using "pip install nfcpy". Now you can run the following Python code:

#### Python

```py

import ndef # required for tag.ndef
import nfc  # imports nfcpy
import signal
from sys import exit
from time import sleep


def signal_handler(sig, frame):
    print('You pressed Ctrl+C! Exiting.')
    exit(0)


def write(clf, msg):
    while True:
        tag = clf.connect(rdwr={'on-connect': lambda tag: False})
        print("Tag detected:", tag, "\n")
        print("Current Content:", tag.ndef.records)
        
        # write to tag (tag.format() might be an idea for fresh tags)
        tag.ndef.records = [ndef.text.TextRecord(msg, 'en', 'UTF-8')]
        print("Successfully wrote to tag:", msg)
        break

    clf.close()


def read(clf):
    while True:
        tag = clf.connect(rdwr={'on-connect': lambda tag: False})
        print("Tag detected:", tag)
        for record in tag.ndef.records:
            print("Stored message:", record.text, "\n")

        sleep(1) # sleep 1 sec

    clf.close()


def main():
    signal.signal(signal.SIGINT, signal_handler)
    clf = nfc.ContactlessFrontend('usb')
    read(clf)
    # write(clf, "my msg")


main()
# this program assumes that the tag is formatted as ndef and only holds a simple text message
# you can also use the android app TagWriter by NXP to write a text to your tag
# this way it will be formatted correctly


```
If you run the program and get an error that your device was not found, try to re-connect the USB device or restart your PC. From time to time, I still get this error, and that seems to be the only workaround that consistently works. Otherwise, the program will start and wait until you scan a tag, then try to read the tag and display its contents. By commenting out the read(clf) line and removing the comment from the write(clf, "text to write") line,  you can write to your tag (overwriting any data that was stored on it). Change "text to write" to whatever you want to write to your tag. After writing to your tag, you can also use your smartphone to read the text. Or you use your smartphone to write text and use your PC to read the text and execute some script depending on what was read or the ID of the tag that was scanned. Just use this code as the basis for whatever you want to code!

## Conclusion

NFC is a fun technology with lots of potential. Luxury clothing brands have already started to put NFC tags on their clothes so that you can verify that you did not buy anything counterfeit. It is also used in logistics to track things or for verified ticket sales. Generally, NFC is a great way to provide cheap authentication for physical objects (tags can cost less than 50 cents). I recommend that use NTAG tags (2xx or 4xx) for best compatibility and have fun. The Python library "nfcpy" makes use of libnfc and is easy to use!