# covert-recording-threat-model

## Key Takeaway:
A tablet is a seemingly innocuous device, however can be setup to secretly (i.e: when the screen is off / tablet is in "sleep" mode) start an audio recording on the tablet from 10 meters away and save it to the disk. 

## Intro 
The [Lenovo Tab M11](https://www.lenovo.com/us/en/p/tablets/android-tablets/lenovo-tab-series/lenovo-tab-m11-11-inch-mtk/len103l0022) can be used with the [Lenovo Tab Pen Plus](https://www.lenovo.com/us/en/p/accessories-and-software/tablet-accessories/tablet-accessories_android-tablets/zg38c09172?orgRef=https%253A%252F%252Fwww.bing.com%252F).

The pen, that connects to the tablet via Bluetooth, comes with a button.
<img width="1200" height="964" alt="image" src="https://github.com/user-attachments/assets/0d337fb8-7140-4fd2-827c-182bc2eeb75f" />

## What the button is normally for

On an Android tablet like the Tab M11: Android's **Settings → Pen & keyboard** only offers a short fixed list of actions, and third-party remapping apps (e.g. Button Mapper) don't detect the press at all. 

## Bluetooth connection 

The pen auto-pairs to the tablet over Bluetooth. Lenovo doesn't publish a specific working distance — it's ordinary Bluetooth Low Energy range (roughly 10 m / 30 ft line-of-sight, less through walls/pockets)

## Battery

- 80 mAh Li-ion cell, rated up to 35 hours of use per charge

## What this repo is 

This started as a hands-free "press the pen button, start recording" convenience script, then became a small threat-model exercise: **a stylus button most people never think about can be read, at the raw input level, by anything with shell access on the device — including to silently arm a microphone.**

This repo documents that finding as a writeup. 

## Components

**Shizuku** is an Android app that lets other apps use system-level APIs without rooting the device. It runs as a background service that you start once (over Wireless Debugging or USB/ADB) and then authorize per app. This project uses it to give the recorder system-level access that a normal app can't get.

**Termux** is a terminal environment for Android with a shell, package manager, and scripting tools (bash, Python, etc.). This project uses it to run the recording scripts and handle the pen button input, connecting the button event to the recording command.

Termux runs the scripts, and Shizuku provides the Android permissions they need. Termux alone can't reach those system APIs.

## Overview of how the script works
Setup: sets RISH_APPLICATION_ID, takes a wake lock so the script keeps running with the screen off, and creates the PenRecordings folder and log path.

Capture: deletes the old capture file, then starts getevent on event8 through rish in the background, writing straight to /sdcard/pen_ev.txt.

Follow: tail -F reads new lines from that file, and grep keeps only the MSC_SCAN button lines.

Filter: for each event, anything other than a double press (0601) is logged as ignored, and a double press within 3 seconds of the last toggle is also ignored.

Check state: asks termux-microphone-record -i whether it's recording, retries once after 1 second if the answer is empty, and gives up and logs it if still empty.

Toggle: if recording, it stops and logs "stop". If idle, it starts a no-limit recording to a timestamped .m4a and logs "start" only if "Recording started" appears, otherwise "start FAILED".

Reset: it saves the time of this toggle for the debounce, then loops back to wait for the next press.
