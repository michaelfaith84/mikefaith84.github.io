---
layout: post
title: "And now, the PAK"
date: 2026-09-13
tags:
  - rfid
  - nfc
  - reverse-engineering
author: Mike Faith
---

I stumbled upon the PAK while exploring the longstanding issue of no smart phones supporting reading/writing to sectors other than 0 on the NTAGi2c Plus 2k.

---

## An Overview

NFC features three acknowledgements to commands that don't return data: ACK (0x0A), NAK (0x00; negative acknowledgement), and the PAK (no response; passive acknowledgement). The problem is NFC controllers handle the PAK as a disconnect since no response is recieved. Because <a href="">homophones are fun</a>, I couldn't resist diving deeper.

## But TagInfo Can Read Sector 1

<img src="/images/ntagi2c_pak.png" />

After a datasheet dive and failing at multiple attempts with NFC Tools and Raw NFC, I was annoyed that it didn't work. But then I was alerted that TagInfo successfully reads Sector 1.

<img src="/images/taginfo_screenshot.png" />

Sure enough. Looks to be reading--but is it really? If so, how?

<img src="/images/sector_select_sniff.png" />

The exact same way I was trying with raw commands--it just failed. And then the answer was suddenly obvious:

```typescript
async function sectorSelect(t: Transport, sector: number): Promise<void|false> {
	try {
		await t.transceive([0xc2, 0xff]) // Success returns ACK (0x0A)}
	catch {
		return false // We got a NAK or the transponder left the field
	}
  	try {
		await t.transceive([sector, 0x00, 0x00, 0x00]) // Success returns nothing
  	} catch {
    	// Error thrown because the controller thinks the transponder
		// 	left the field. In reality, PAK is most likely
  	}
}
```

Just suppress the error and keep on partying. Better accecss to NFC hardware would make PAK detection easier--simply telling if there are transponders in the field. But that would require firmware changes. The PAK, from what I can see, is fairly common with proprietary commands and extends to ISO15693 as well.

## A Better Solution?

During <a href="https://github.com/RfidResearchGroup/proxmark3/pull/3099">my time chasing down</a> a way to get the proxmark's `hw tune` command to stop saying the HF antenna was b0rked when a booster board (lc tank circuit) was in use. I _thought_ I was onto something with voltage but after binning the boards with a VNA that just wasn't going to work... But the coupling state **must** have an impact physically. So I sped up the ADC that monitors the antenna and watched the voltage decay. I discovered three bands: normal coupling, lc tanks, and if the antenna was on metal. This means that a NFC controller can inherently detect **presence** which gives us a way to _know_ the difference between a PAK and a transponder dropping out of the connection. Industry wide firmeware level up?? 🤞
