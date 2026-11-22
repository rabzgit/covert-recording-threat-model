# covert-recording-threat-model

The [Lenovo Tab M11](https://www.lenovo.com/us/en/p/tablets/android-tablets/lenovo-tab-series/lenovo-tab-m11-11-inch-mtk/len103l0022) can be used with the [Lenovo Tab Pen Plus](https://www.lenovo.com/us/en/p/accessories-and-software/tablet-accessories/tablet-accessories_android-tablets/zg38c09172?orgRef=https%253A%252F%252Fwww.bing.com%252F).

The pen, that connects to the tablet via Bluetooth, comes with a button.
<img width="1200" height="964" alt="image" src="https://github.com/user-attachments/assets/0d337fb8-7140-4fd2-827c-182bc2eeb75f" />

## What the button is normally for

On Windows PCs, Lenovo's pen button is fully configurable (single press, double press, long press each map to an app or shortcut, e.g. OneNote, Sketchpad, screenshot capture). On an Android tablet like the Tab M11, there's no such settings panel: Android's **Settings → Pen & keyboard** only offers a short fixed list of actions, and third-party remapping apps (e.g. Button Mapper) don't detect the press at all. The only way to see the press is at the raw input level (`/dev/input/event*`), which needs shell access (Shizuku or adb) — a regular app can't read it.

## Bluetooth connection and the "lost pen" alert

The pen auto-pairs to the tablet over Bluetooth and ships with a built-in **anti-lost alert**: if the Bluetooth link drops, the tablet pops a notification. Lenovo doesn't publish a specific working distance — it's ordinary Bluetooth Low Energy range (roughly 10 m / 30 ft line-of-sight, less through walls/pockets), and once you're out of that range the link drops and the alert fires.

One testing wrinkle worth noting: **holding the button for several seconds forces the pen to disconnect and re-pair** (its own reset/pairing gesture), which triggers the exact same "lost pen" alert as walking out of range. The two are easy to confuse unless you're watching for which one actually happened.

## Battery

- 80 mAh Li-ion cell, rated up to 35 hours of use per charge
- USB-C charging, with Lenovo's datasheet quoting ~25% charge in under 15 minutes
- Charging port is charge-only — no data/firmware access over USB-C

## What this repo is (and isn't)

This started as a hands-free "press the pen button, start recording" convenience script, then became a small threat-model exercise: **a stylus button most people never think about can be read, at the raw input level, by anything with shell access on the device — including to silently arm a microphone.**

This repo documents that finding (what was tested, what worked, what didn't) as a writeup, **not** as a ready-to-run tool. The assembled trigger script, exact device identifiers, and step-by-step reproduction commands are intentionally left out, so this stays a documented finding rather than a working covert-recording recipe someone could point at another person's device.
