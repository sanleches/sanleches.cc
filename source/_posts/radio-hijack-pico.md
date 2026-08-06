---
title: I Hijacked a Tiny F1 Radio with a Pico 2 W
date: 2026-08-06 15:00:00
category: projects
tags:
  - C++
  - embedded
  - Raspberry Pi Pico
  - radio
---

At an F1 event they handed out tiny FM radios. Most people probably used them normally. I opened mine, found an RDA5807M receiver inside, and decided the correct next step was to put it on Wi-Fi.

The result is [RadioHijackCpp](https://github.com/sanleches/RadioHijackCpp): C++17 firmware for a Raspberry Pi Pico 2 W that controls the radio over I2C and serves a small browser UI. It tunes, seeks, scans, changes volume, handles presets, and displays RDS station data. This is a Pico SDK rewrite of my earlier MicroPython version.

## The hardware

The connection is pleasantly small:

```text
Pico 2 W GP4  -> RDA5807M SDA
Pico 2 W GP5  -> RDA5807M SCL
i2c0          -> 100 kHz
```

The radio chip lives at I2C address `0x11`. The firmware writes its registers directly, including the tune, seek, mute, power, RDS, and volume bits. FM frequencies are converted into channel numbers using 87.0 MHz as the base and 100 kHz spacing. Nothing magical, just bit fields and the occasional moment of wondering why the radio stopped making noise.

## The part that got out of hand

I could have stopped after tuning a station. Instead, the firmware now has a radio driver, an RDS parser, flash-backed presets, Wi-Fi management, a DHCP server, an HTTP server, a JSON API, and an entire HTML/CSS/JavaScript dashboard compiled into program flash.

There is no web framework and no filesystem. The HTTP server runs directly on lwIP callbacks, accepts small GET and HEAD requests, and sends responses in TCP-sized chunks. The browser UI uses plain `fetch()` calls and polls for fresh RDS text every 2.5 seconds.

The useful API bits look like this:

```text
/status
/tune?frequency=99.9
/seek?direction=up
/volume?level=8
/scan
/presets
```

If the Pico cannot join the configured Wi-Fi network, it creates `RadioHijack-AP` and serves the dashboard at `192.168.4.1`. I even wrote a tiny DHCP implementation for the fallback network. It offers one client address, which is enough for connecting a phone to a radio and absolutely not enough for running an office.

## Small embedded details I liked

RDS arrives in pieces, so the parser gradually assembles the eight-character station name and RadioText buffer from different group segments. A changed station identifier resets the buffers so text from two stations does not get mixed together.

Presets are stored in the Pico's last flash sector using a small versioned binary format. It supports 16 entries and disables interrupts while erasing and programming flash. It is deliberately simple: no wear leveling, no database, just enough persistence for radio presets.

The main application stays cooperative. Networking runs through lwIP's background mode while the main loop polls RDS, serial commands, and the status LED. No RTOS was needed.

## Where it is now

It works as a compact local device, not a hardened internet service. The API is unauthenticated HTTP, scans are synchronous, and the tiny DHCP server only understands the path I need. Those constraints are fine because the goal was to turn giveaway hardware into a self-contained Wi-Fi radio, not build the next enterprise radio platform.

Code: [github.com/sanleches/RadioHijackCpp](https://github.com/sanleches/RadioHijackCpp)
