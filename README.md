# Tongveo TV‑V20U — Browser‑based PTZ Control

## Overview

Most PTZ webcams expose their video stream over USB (UVC) only. The Tongveo TV‑V20U is no exception: it normally accepts **Pan / Tilt / Zoom (PTZ) commands only via the IR remote or the RS‑232 / RS‑485 serial port**.
This project shows how to control the camera directly from any Chromium‑based browser:

* **Video preview** is obtained with `getUserMedia()` (UVC).
* **PTZ commands** are sent either

  * as standard **UVC extension controls** (if the driver exposes them), or
  * as **VISCA packets** over a USB‑to‑Serial adapter via the **Web Serial API**.

The result is full PTZ control (including hotkeys) without installing proprietary software.

---

## Requirements

* Tongveo TV‑V20U (or other VISCA‑compatible PTZ camera)
* Chrome 89 / Edge 89 or later (Web Serial + UVC)
* USB‑to‑RS232 (or RS485) adapter wired to the camera’s control port
* Enable **“Experimental Web Platform features”** in `chrome://flags` if Web Serial is not yet enabled by default

---

## Getting Started

You can try this project in two ways:

### Online Demo

Try the application in your browser at https://sphaerox.github.io/tongveo_v20u
**Note:** The online demo only supports camera preview. USB Serial PTZ control features are only available when running locally due to browser security restrictions.

### Local Installation

1. Clone or download this repository.
2. Open **`tongveo_ptz_control.html`** in a Chromium‑based browser.
   (The file can be served from any local web server or opened directly from disk.)
3. Grant permission when the browser asks for camera access.
4. Click **Connect Serial (VISCA)** and choose the USB‑serial port.
5. Use the on‑screen buttons or the hotkeys:

   | Action    | Hotkey | Behaviour             |
   | --------- | ------ | --------------------- |
   | Zoom In   | `+`    | Single zoom step      |
   | Zoom Out  | `-`    | Single zoom step      |
   | Pan Left  | `←`    | Continuous while held |
   | Pan Right | `→`    | Continuous while held |
   | Tilt Up   | `↑`    | Continuous while held |
   | Tilt Down | `↓`    | Continuous while held |

---

## How It Works

### 1. UVC PTZ (if available)

Some webcams expose PTZ as UVC **extension controls**.
If the browser reports `track.getCapabilities().pan` / `tilt` / `zoom`, the script moves the lens by calling `applyConstraints()` with absolute positions.

### 2. VISCA over Web Serial

When UVC PTZ is absent, the script falls back to the serial port:

* **Pan / Tilt**: `0x81 0x01 0x06 0x01 [panSpeed] [tiltSpeed] [panDir] [tiltDir] 0xFF`
* **Zoom Tele / Wide**: `0x81 0x01 0x04 0x07 0x20|0x30 0xFF`, followed by `0x00` (stop)

The browser sends these bytes with `port.writable.getWriter().write()`. A short timeout stops each movement, giving fine‑grained steps.

---

## Customising

* **Step size** and **speed** are defined near the top of the script (`delta` values and VISCA speed bytes).
* To invert axes or change hotkeys, edit the `hotkeyMap` object.
* For other VISCA‑compatible cameras, adjust the baud rate or command set if necessary.

---

## Limitations

* Web Serial currently works only in Chromium‑based browsers.
* The page must be served over **HTTPS** (or from `localhost`) for Web Serial to work.
* Browsers restrict serial access to user gestures — the user must click **Connect Serial** at least once per session.

---

## License

MIT License
