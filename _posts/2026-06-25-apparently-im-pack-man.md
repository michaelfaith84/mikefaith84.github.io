---
layout: post
title: "Apparently I'm PACK-man"
date: 2026-06-29
categories: other
tags:
  - rfid
  - security-research
  - access-control
author: Mike Faith
---

Gather round and hear tales of my encounters with the P(assword)ACK(nowledgement) feature found in NXP's lower end transponders.

---

## An Overview

Some of NXP’s transponders feature a “Password ACKnowledgement” response to a successful auth. We see this on NTAG21X and NTAGI2C transponders that use a 4-byte password with a 2-byte PACK. What’s the point of it? It allows the reader to determine if the tag is genuine. It is, so far as things go, pretty weak security.

The most famous use of the PACK is probably Nintendo’s Amiibo.

## Where I Come In

### mo.lock

<video controls>
  <source src="/videos/packbuster_demo.webm" type="video/webm">
</video>

My life intersected with the PACK when my boss told me about Motogadget’s mo.lock. It was their second RFID lock, the previous being low frequency. The story goes that they cancelled the boss’s order and sent him an email stating more of less that they didn’t want him fucking with their products. Fast forward to a customer’s request to be able to use their implant with the mo.lock--the new fangled NFC version. The NTAG transponders in our implants are 216s--slightly different memory structure, different capability container, but same command set. The lock refused to enroll his implant. After some investigation, it is revealed that they are using the PACK (and also checking the memory for some data--not an NDEF record, just data). It comes up a few times over a couple of years but no one gives it a serious attempt. Until the boss tells me to because he’s still irked they canceled his order years ago.

Because we had such limited samples, we couldn’t reverse their derivation method. That left bruting it. 65k options isn’t too bad. But how would you know when it was done? Depending on how you can time things, If you managed the ability to try one PACK a second (spoiler: you can’t), that would take more than 18 hours. You can’t just watch a light that long--at least I can’t. So that meant a fully automated solution.

To check, we have to try and enroll a transponder. During regular operation, the mo.lock LED is off. When an authorized tag is scanned, it turns green. If an authorized tag is held in the field for a period during that auth, it will turn blue for enrollment. An invalid tag will trigger a red LED flashing on and off while a valid one will be green flashing. This rules out something simple like using like a color detector, color filter etc. We need to know the response. I snagged a webcam and got to work with opencv. As someone who hadn’t done a machine vision project before, the biggest lesson was that light is a fickle mistress. Be prepared to scream if you find yourself here (spoiler: the biggest accuracy boost I got was using a diffuser aka folded scrap of paper).

<img src="/images/pack_buster_desk.jpg" />

What resulted was an _appliance_ that I dubbed _“the PACK Buster.”_ Its principal components were a raspi, pm3 (for simming transponders and stealing passwords), and a mo.lock. Anyone that has worked with NFC knows just how fickle coupling can be. And those who have done a fair amount of sniffing understands just how maddening positioning can be with the pm3 easy’s HF antenna… The mo.lock was _tedious_ to position, to say the least--it must be in a mid-air. The wizard Hamspiced from [Midwest Gadgets](http://midwestgadgets.com/) even sent me an awesome _thing_ to use with the project (it was not used because **_it’s fucking working don’t touch it_**). Everyday for a certain time window, it would think it found the PACK. Something about the way the light at that particular angle and god knows what other variables (the glass, the camera lens, the curvature of the LED, who knows) would cause a false positive. It slowed down the process because I wasn’t home to catch it immediately often. And, to be frank, it made me think it just wasn’t going to work. Eventually, I told the boss that I thought it was fucked, that they were checking the capability container or GET_VERSION (I was simming an NTAG216 like the boss’s implant). And then when I got home, a funny thing happened. I saw the “PACK Found!” message as I had every day that week, I rolled it back again, and headed to the bathroom. I was over this project. But there it was again, “PACK Found!” with the same damn PACK. A second attempt showed the same thing.

I confirmed this by talking to a community member with a mo.lock and had them send me their implant’s UID. After bruting the PACK, I wrote a [Raw NFC](https://play.google.com/store/apps/details?id=com.vivokey.rawnfc) link for him to program his implant with and there it was: a happy cyborg authing his implant on a mo.lock.

<video controls>
  <source src="/videos/mo.lock_implant.webm" type="video/webm">
</video>

### Ultimate Magic Card aka Ultimate gen4

The thing about NFC transponders is that a lot of emphasis is placed on the immutability of some blocks such as those that contain the UID. For the transponders in question, their UID is 7-bytes. No UID is ever repeated. Which allows the aforementioned derivation schemes for vendor locking shenanigans. So called “magic” transponders break that rule. Most commonly, these emulate MIFARE Classic 1k. This magic card is cool because it can emulate many different kinds of high frequency transponders--including the type 2 NFC flavor of NTAG transponders. It even implemented the password and PACK!

During the mo.lock project I stumbled across something that others had: the PACK didn’t seem to work. I did some brief testing--the same testing many others had done--and walked away from it without an answer because time is precious. But it bothered me that I couldn't figure it out. Over the next ten months, it came up a couple of times and eventually someone said they figured it out. I was annoyed but didn’t bother to verify their solution until one day I saw yet another person requesting to be able to use the UG4 with an Amiibo… I found the post claiming victory and saw that it still didn’t work. Game fucking on.

I couldn’t remember why i felt like this was solvable when so many others had just said, “it’s fucked,” and given up. But I _knew_ there was a reason. I grabbed a fresh card and checked the config. As expected, MFC 1k with a 4-byte uid. I used the uf_mf_ultimatedcard script with my proxmark3 easy to change it to -t 17 (ntag213).

```
[usb] pm3 --> script run hf_mf_ultimatecard -w 1 -t 17
[#] Searching implicit relative paths
[#] Searching preferences paths
[#] Searching user .proxmark3 paths
[#] Searching current workdir paths
[+] executing lua /home/work/proxmark3/client/luascripts/hf_mf_ultimatecard.lua
[+] args '-w 1 -t 17'

Starting Ultralight Wipe
Wiping tag
.............................................................................


Setting: Ultimate Magic card to NTAG 213
Writing new UID 	04E10CDA993C80
Writing new version	0004040201000F03
Writing new UID 	04E10CDA993C80
Writing new NTAG PWD 	FFFFFFFF
Writing new PACK	0000
Writing new MFUL signature	8B76052EE42F5567BEB53238B3E3F9950707C0DCC956B5C5EFCFDB709B2D82B3
Setting: Ultimate Magic card to NTAG 213
Writing new UID 	04E10CDA993C80
Writing new version	0004040201000F03

[+] finished hf_mf_ultimatecard

[usb] pm3 --> script run hf_mf_ultimatecard -c
[#] Searching implicit relative paths
[#] Searching preferences paths
[#] Searching user .proxmark3 paths
[#] Searching current workdir paths
[+] executing lua /home/work/proxmark3/client/luascripts/hf_mf_ultimatecard.lua
[+] args '-c'

=========================================================================
			Ultimate Magic Card Configuration
=========================================================================
 - Raw Config      	01010000000003000978009102DABC19101011121314151644000001FB1D
 - Card Protocol    	MIFARE Ultralight/NTAG
 - Ultralight Mode   	NTAG21x
 - ULM Backdoor Key 	00000000
 - GTU Mode     	Disabled, high speed R/W mode for Ultralight
 - Card Type     	NTAG 213
 - UID           	04E10CDA993C80
 - ATQA          	00 44
 - SAK          	00

=========================================================================
			Magic UL/NTAG 21* Configuration
=========================================================================
 - ATS          	Disabled
 - Password     	[0xE5] FFFFFFFF	[0xF0] FFFFFFFF
 - Pack         	[0xE6] 0000	[0xF1] 0000
 - Version      	0004040201000F03
 - Signature    	8B76052EE42F5567BEB53238B3E3F9950707C0DCC956B5C5EFCFDB709B2D82B3
 - Max R/W Block  	FB

[+] finished hf_mf_ultimatecard


```

<img src="/images/ntag_213_memory.png" />

So, in theory, we're interested in page 2B (password) and the first two bytes of 2C.

```
[usb] pm3 --> hf 14a raw -skc 1bffffffff # auth
[+] 00 00 [ A0 1E ] # Success -> PACK returned: 0000
[usb] pm3 --> hf 14a raw -skc a22b01020304 #change password to 01020304
[+] 0A # Successful write
[usb] pm3 --> hf 14a raw -skc 302b
[+] 01 02 03 04 00 00 00 00 00 00 00 00 00 00 00 00 [ F9 C2 ] # reading shows the updated value
[usb] pm3 --> hf 14a raw -skc 1b01020304
[+] 04 # auth fails with new password
```

So, something isn’t quite right here…

```
[usb] pm3 --> script run hf_mf_ultimatecard -c
[#] Searching implicit relative paths
[#] Searching preferences paths
[#] Searching user .proxmark3 paths
[#] Searching current workdir paths
[+] executing lua /home/work/proxmark3/client/luascripts/hf_mf_ultimatecard.lua
[+] args '-c'

=========================================================================
			Ultimate Magic Card Configuration
=========================================================================
 - Raw Config      	01010000000003000978009102DABC19101011121314151644000001FB1D
 - Card Protocol    	MIFARE Ultralight/NTAG
 - Ultralight Mode   	NTAG21x
 - ULM Backdoor Key 	00000000
 - GTU Mode     	Disabled, high speed R/W mode for Ultralight
 - Card Type     	NTAG 213
 - UID           	04E10CDA993C80
 - ATQA          	00 44
 - SAK          	00

=========================================================================
			Magic UL/NTAG 21* Configuration
=========================================================================
 - ATS          	Disabled
 - Password     	[0xE5] FFFFFFFF	[0xF0] FFFFFFFF
 - Pack         	[0xE6] 0000	[0xF1] 0000
 - Version      	0004040201000F03
 - Signature    	8B76052EE42F5567BEB53238B3E3F9950707C0DCC956B5C5EFCFDB709B2D82B3
 - Max R/W Block  	FB

[+] finished hf_mf_ultimatecard
```

The lua script suggests it uses the NTAG216 pages for the password (E5) and PACK (E6). Second set of values is related to GTU/Shadow Mode (I believe).

```
[usb] pm3 --> hf 14a raw -skc a2e501020304 # Change password
[+] 0A
[usb] pm3 --> hf 14a raw -skc 1b01020304
[+] 00 00 [ A0 1E ] # Success.... It changed this time.

[usb] pm3 --> script run hf_mf_ultimatecard -c
[#] Searching implicit relative paths
[#] Searching preferences paths
[#] Searching user .proxmark3 paths
[#] Searching current workdir paths
[+] executing lua /home/work/proxmark3/client/luascripts/hf_mf_ultimatecard.lua
[+] args '-c'

=========================================================================
			Ultimate Magic Card Configuration
=========================================================================
 - Raw Config      	01010000000003000978009102DABC19101011121314151644000001FB1D
 - Card Protocol    	MIFARE Ultralight/NTAG
 - Ultralight Mode   	NTAG21x
 - ULM Backdoor Key 	00000000
 - GTU Mode     	Disabled, high speed R/W mode for Ultralight
 - Card Type     	NTAG 213
 - UID           	04E10CDA993C80
 - ATQA          	00 44
 - SAK          	00

========================================================================================
			Magic UL/NTAG 21* Configuration
========================================================================================
 - ATS          	Disabled
 - Password     	[0xE5] 01020304	[0xF0] FFFFFFFF
 - Pack         	[0xE6] 0000	[0xF1] 0000
 - Version      	0004040201000F03
 - Signature    	8B76052EE42F5567BEB53238B3E3F9950707C0DCC956B5C5EFCFDB709B2D82B3
 - Max R/W Block  	FB

[+] finished hf_mf_ultimatecard
```

Alright. We can write to E5 to change the PW.

```
[usb] pm3 --> hf 14a raw -skc a2e6beef0000 # try changing the PACK with the block from the script
[+] 0A # Successfully written

[usb] pm3 --> script run hf_mf_ultimatecard -c
[#] Searching implicit relative paths
[#] Searching preferences paths
[#] Searching user .proxmark3 paths
[#] Searching current workdir paths
[+] executing lua /home/work/proxmark3/client/luascripts/hf_mf_ultimatecard.lua
[+] args '-c'

=========================================================================
			Ultimate Magic Card Configuration
=========================================================================
 - Raw Config      	01010000000003000978009102DABC19101011121314151644000001FB1D
 - Card Protocol    	MIFARE Ultralight/NTAG
 - Ultralight Mode   	NTAG21x
 - ULM Backdoor Key 	00000000
 - GTU Mode     	Disabled, high speed R/W mode for Ultralight
 - Card Type     	NTAG 213
 - UID           	04E10CDA993C80
 - ATQA          	00 44
 - SAK          	00

=========================================================================
			Magic UL/NTAG 21* Configuration
=========================================================================
 - ATS          	Disabled
 - Password     	[0xE5] 01020304	[0xF0] FFFFFFFF
 - Pack         	[0xE6] BEEF	[0xF1] 0000
 - Version      	0004040201000F03
 - Signature    	8B76052EE42F5567BEB53238B3E3F9950707C0DCC956B5C5EFCFDB709B2D82B3
 - Max R/W Block  	FB

[+] finished hf_mf_ultimatecard

[usb] pm3 --> hf 14a raw -skc 1b01020304
[+] 00 00 [ A0 1E ] # Successful auth but the wrong PACK
```

Annoying. I would have given up here had I not seen this very UG4 return a PACK of FFFF initially. As in, from the default MFC 1k, I did a -t 19 (iirc) and then auth'd. It _seems_ the PACK should be derived from user memory as the password is. So, I striped the user memory to FB with page numbers and auth'd again.

```
[usb] pm3 --> hf 14a raw -skc a213beef0000
[+] 0A
[usb] pm3 --> hf 14a raw -skc 1b01020304
[+] BE EF [ 27 A1 ]
[usb] pm3 -->
```

There you have it. Page 13 sets the PACK. Why? The NTAG210 and MIFARE Ultralight (MF0UL11) both store the PACK on page 13. It’s not a satisfying answer. Were mistakes made in design or is this a config error? More research needs to be done.

<img src="/images/ntag_210_memory.png" />

<img src="/images/mifare_ultralight_memory.png" />
