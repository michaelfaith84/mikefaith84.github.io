---
layout: post
title: "UG4 Config from Android"
date: 2026-09-20
tags:
  - rfid
  - nfc
  - reverse-engineering
  - ug4
author: Mike Faith
---

A rabbit hole of Proxmark-fueled reverse engineering and app development that required a couple of hacks.

---

A buddy took a stab at this on my suggestion a couple of years ago. He drew the conclusion that it was impossible. Because I am—as a friend described—low-key competitive, I had to check it out myself. These are just the sorts of projects that a Proxmark is essential for.

## Getting Started

First of all, the repository itself is a vast wealth of knowledge—like where we even start. How does the PM3 detect the UG4?

![ug4 magic card notes](/images/ug_id_notes.png)

OK, what are the params being used?

![proxmark3 hf 14a raw help](/images/pm3_hf_14a_raw_help.png)

Seems like it’s a pretty normal way to send a 14a-3 command—select happens, so we’re going through anticollision as normal.

## The Hurdle That Stopped Past Attempts

Let's give this a go with a couple of apps that send raw commands.

### NFC Tools

<img src="/images/nfc_tools_ug4.png" />

Very messy. There is a reader flag to turn off NDEF checks on connect. NFC Tools TXs when you press a button—you have to be in contact with a card. All those reads of block 0 are hacky presence detection. Block 2 is an NDEF read—that's the OS doing its thing. You can turn this off with a reader flag.

### Raw NFC

<img src="/images/raw_nfc_ug4.png" />

Raw NFC sends on tap. It's much cleaner. But you can see that the NDEF check isn't off and the same presence detection is happening—sort of. NFC Tools looks to have incorporated it at an app level. Investigation using an app I made shows that Android does this by itself as well—same reason. When you feel it buzz or it waits to TX until it "detects" a transponder—this is how. Going through AC isn't enough.

Both attempts and my own failed to successfully send a magic command to the UG4, but it did give me a theory: the first message after the last select in AC _must_ be a magic command, else the transponder will reject it.

Not to be deterred, and already working with a custom fork of react-native-nfc-manager, I played around a bit. I noticed that on `.connect()` the presence check happens and `.close()` sends HLTA. But if you do another `.connect()`, this skips the presence detection read.

<img src="/images/ug4_config_read.png" />

Party. Proof of concept achieved.

## The Second Hurdle

The problem with MIFARE Classic is that it's pretty baked into the NFC controller. As in, doing a `.transceive()` on an MFC transponder will result in non-MFC commands being utterly rejected. This was annoying. Even using the NFC-A tech type's TX didn't seem to help. This was a problem because it meant the card had to be in Ultralight mode _and_ we couldn't change it once it was in MFC mode. This is a pretty massive limitation.

But I had a weird idea spawned from time in smart card-landia: the J3R family of NXP's smart cards can have MIFARE emulation. One particularly relevant to this is the DESFire EV3C. Why? Because it lets you add a DESFire app that is a MIFARE Classic 1K sector, complete with Crypto1 support. And I knew these worked well with Android.

So I set the SAK/ATQA to 0x28/0x0008 (later corrected to the appropriate SAK bit flips) + ATS. Sure enough, I was able to use my presence hack to send config changes.

![magic commander probe on android](/images/magic_commander_detection_sniff.png)

This is what it looks like when the Magic Commander Android app does its magic.
