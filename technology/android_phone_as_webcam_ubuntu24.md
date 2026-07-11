# Using an Android Phone as a Webcam on Ubuntu 24.04 LTS

A comprehensive guide to transforming your Android smartphone into a high-quality webcam for Ubuntu 24.04 LTS (Noble Numbat).

> **Author:** Jack Liu Shurui  
> **Last updated:** July 2026  
> **Repository:** [jackliusr/research](https://github.com/jackliusr/research)

---

## 1. Overview

### Why Use an Android Phone as a Webcam?

- **Better camera quality:** Modern smartphones have far superior sensors, lenses, and image processing compared to typical USB webcams — sharp 1080p/4K video with proper autofocus, HDR, and low-light performance.
- **Multiple camera options:** Access wide-angle, macro, telephoto, and front-facing cameras.
- **No dedicated hardware cost:** A USB webcam costs $50–150; this solution is effectively free (or a few dollars for a paid app).
- **Portability:** Your webcam goes wherever your phone goes.

### Requirements

| Component | Requirement |
|-----------|-------------|
| **Android phone** | Android 6.0+ (most apps); Android 12+ (scrcpy camera source) |
| **Ubuntu machine** | Ubuntu 24.04 LTS (any flavour) |
| **Cable** (USB mode) | USB data cable |
| **Software** | One of the methods below |

### Connection Methods

| Method | Latency | Reliability | Best For |
|--------|---------|-------------|----------|
| **USB** | Very low (≤50ms) | Excellent (wired) | Video calls, streaming |
| **WiFi** | Low–moderate (100–300ms) | Good | Quick calls, phone away from PC |

**Recommendation:** USB for reliability and low latency; WiFi for convenience when the phone needs to be away from the computer.

---

## 2. Method 1: DroidCam (Most Popular)

**DroidCam** by Dev47Apps is the most widely used Android webcam solution on Linux.

- **Website:** [droidcam.app](https://droidcam.app/) / [dev47apps.com](https://www.dev47apps.com/droidcam/linux/)
- **Android app:** [DroidCam on Google Play](https://play.google.com/store/apps/details?id=com.dev47apps.droidcam)
- **Pricing:** Free (720p, small watermark) / Paid $4.99 (1080p, no ads, no watermark)
- **GitHub:** [github.com/dev47apps/droidcam-linux-client](https://github.com/dev47apps/droidcam-linux-client)

### Installation on Ubuntu 24.04

```bash
# Option A: Install via .deb (new client from droidcam.app — recommended)
wget https://droidcam.app/droidcam_client_amd64.deb
sudo dpkg -i droidcam_client_amd64.deb
sudo apt install -f

# Option B: Legacy client from dev47apps (source build)
# wget https://files.dev47apps.net/linux/droidcam-1.8.1.tar.gz
# tar -xf droidcam-*.tar.gz && cd droidcam && sudo ./install

# Install ADB (required for USB mode)
sudo apt update && sudo apt install adb
```

### Connection Modes

- **WiFi:** Both devices on same LAN; streams over TCP port 4747
- **USB:** Uses ADB port forwarding — lower latency, more reliable

### USB Mode

```bash
adb forward tcp:4747 tcp:4747
droidcam-cli adb 4747
```

### WiFi Mode

```bash
droidcam-cli 192.168.1.100 4747   # Replace with phone's IP
```

### Features

- 720p free / 1080p paid, 30 fps, audio via phone mic
- Creates V4L2 device (`/dev/video0`) — works with Zoom, Teams, Meet, OBS, Discord
- Both USB and WiFi modes

### Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Easy setup, mature, reliable | Free version limited to 720p |
| Both USB and WiFi | Small watermark in free version |
| Audio supported | Paid upgrade ($4.99) needed for HD |

---

## 3. Method 2: Iriun Webcam

**Iriun Webcam** is a free, no-watermark alternative providing full 1080p HD video without requiring a paid upgrade.

- **Website:** [iriun.com](https://iriun.com/)
- **Android app:** [Iriun Webcam on Google Play](https://play.google.com/store/apps/details?id=com.iriun.webcam)
- **Pricing:** Completely free — no watermark, no ads

### Installation on Ubuntu 24.04

```bash
wget https://iriun.com/iriun-webcam.deb
sudo dpkg -i iriun-webcam.deb
sudo apt install -f

# The .deb auto-installs v4l2loopback-dkms
# If needed, manually load:
sudo modprobe v4l2loopback exclusive_caps=1 max_buffers=2
```

### Features

- 1080p full HD free, 30 fps, audio supported
- Creates V4L2 device at `/dev/videoX`
- WiFi-based, auto-discovers the computer on the same network
- Works with Zoom, Teams, Google Meet, OBS, Skype, Discord

### Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Full 1080p HD completely free | WiFi only (no USB mode) |
| No watermark, no ads | Slight latency over WiFi |
| Simplest setup — install & connect | Relies on network quality |

### Usage

1. Ensure both devices are on the same WiFi network
2. Launch Iriun Webcam on Android
3. Launch Iriun Webcam on Ubuntu (from app menu or `iriun-webcam`)
4. The app auto-discovers and connects

---

## 4. Method 3: scrcpy + v4l2loopback (DIY / Open Source)

**scrcpy** (screen copy) is an open-source tool for displaying and controlling Android devices. Since v2.0, it includes native camera streaming support — not just screen mirroring. Combine it with **v4l2loopback** (kernel module creating virtual video devices) for a fully open-source pipeline.

- **scrcpy:** [github.com/Genymobile/scrcpy](https://github.com/Genymobile/scrcpy)
- **v4l2loopback:** [github.com/umlaeute/v4l2loopback](https://github.com/umlaeute/v4l2loopback)

### Installation

```bash
sudo apt install scrcpy v4l2loopback-dkms adb

# Load v4l2loopback
sudo modprobe v4l2loopback exclusive_caps=1 max_buffers=2

# Persist across reboots (optional)
echo "v4l2loopback" | sudo tee -a /etc/modules
echo "options v4l2loopback exclusive_caps=1 max_buffers=2" | \
    sudo tee /etc/modprobe.d/v4l2loopback.conf
```

### Camera Streaming Commands

```bash
# Rear camera, no display window, no audio
scrcpy --video-source=camera --v4l2-sink=/dev/video10 \
       --no-playback --no-audio

# Front camera, 1080p
scrcpy --video-source=camera --camera-facing=front \
       --camera-size=1920x1080 --v4l2-sink=/dev/video10 \
       --no-playback --no-audio

# With audio (experimental)
scrcpy --video-source=camera --v4l2-sink=/dev/video10 \
       --no-playback --audio-source=mic

# List available cameras
scrcpy --list-cameras

# WiFi mode (ADB over TCP)
adb connect 192.168.1.100:5555
scrcpy --video-source=camera --v4l2-sink=/dev/video10 --no-playback
```

### Key scrcpy Camera Options

| Option | Values | Description |
|--------|--------|-------------|
| `--video-source=camera` | `camera` / `display` | Phone camera vs screen |
| `--camera-facing` | `front` / `back` / `external` | Which camera |
| `--camera-size` | e.g. `1920x1080` | Requested resolution |
| `--camera-id` | `0`, `1`, `2` | Specific camera ID |
| `--v4l2-sink` | `/dev/videoN` | Output virtual webcam device |
| `--no-playback` | — | Headless mode (no window) |
| `--audio-source` | `mic` | Phone mic audio |
| `--video-codec` | `h264` / `h265` | Video codec |

### Pros & Cons

| ✅ Pros | ❌ Cons |
|---------|---------|
| Fully open source (GPL) | Complex setup, command-line only |
| No watermarks, no ads, no cost | Audio support experimental |
| Max flexibility — any res, any camera | Requires USB Debugging enabled |
| Extremely low latency (USB) | No automatic reconnection |

---

## 5. Method 4: OBS Studio + Camera Sources

OBS Studio can integrate an Android phone camera in several ways:

1. **DroidCam/Iriun V4L2 source:** Add → Video Capture Device → select the virtual device
2. **scrcpy window capture:** Run scrcpy in a window, add Window Capture in OBS
3. **IP Webcam stream:** Install "IP Webcam" app on Android, add Media Source with the MJPEG URL (`http://phone-ip:8080/video`)
4. **OBS Virtual Camera:** After setting up the phone camera in OBS, click "Start Virtual Camera" — makes it available to Zoom/Teams/etc.

**Best for:** Streaming and recording workflows. **Not ideal for:** Simple video calls (overkill).

---

## 6. Method 5: AVream (Ubuntu 24.04 Native)

**AVream** is a purpose-built tool for using an Android phone as a webcam + microphone on Linux, specifically tested on Ubuntu 24.04. It combines video and audio into a single virtual device.

- **GitHub:** [github.com/Kacoze/avream](https://github.com/Kacoze/avream)

### Installation

```bash
git clone https://github.com/Kacoze/avream.git
cd avream

sudo apt install adb v4l2loopback-dkms ffmpeg python3-venv
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt

sudo modprobe v4l2loopback exclusive_caps=1 max_buffers=2
adb devices   # Verify connection
./avream
```

| ✅ Pros | ❌ Cons |
|---------|---------|
| Tested on Ubuntu 24.04 | Newer project, smaller community |
| Combines video + audio | Fewer device-specific fixes |
| Free and open source | Requires Python + ffmpeg |

---

## 7. Method 6: GNOME Network Display (Experimental)

Ubuntu 24.04's GNOME has experimental support for Android cameras via **MTP/PTP** mode. Most phones do not expose a live video feed this way. **Not recommended** — included for completeness only.

---

## 8. Setup Steps Comparison

| Feature | DroidCam | Iriun Webcam | scrcpy + v4l2 | AVream |
|---------|----------|-------------|---------------|--------|
| **Free HD** | ❌ (720p) | ✅ 1080p | ✅ Any | ✅ Any |
| **Watermark** | ✅ (free) | ❌ None | ❌ None | ❌ None |
| **USB mode** | ✅ Yes | ❌ WiFi only | ✅ Yes | ✅ Yes |
| **WiFi mode** | ✅ Yes | ✅ Yes | ✅ Yes | ❌ USB |
| **Audio support** | ✅ Yes | ✅ Yes | ⚠️ Partial | ✅ Yes |
| **Ease of setup** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| **V4L2 device** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **Ubuntu 24.04** | ✅ Tested | ✅ Tested | ✅ Tested | ✅ Native |
| **Open source** | ❌ Partial | ❌ Binary | ✅ Full | ✅ Full |
| **Price** | Free/$4.99 HD | Free | Free | Free |

---

## 9. Step-by-Step: DroidCam USB (Recommended)

### Step 1: Enable Developer Options on Android

1. **Settings** → **About Phone** → Tap **Build Number** 7 times
2. **Settings** → **System** → **Developer Options** → Enable **USB Debugging**

### Step 2: Install ADB and DroidCam Client

```bash
sudo apt update && sudo apt install adb
wget https://droidcam.app/droidcam_client_amd64.deb
sudo dpkg -i droidcam_client_amd64.deb
sudo apt install -f
```

### Step 3: Connect Phone via USB

```bash
adb devices
# Should show: <device_id>    device
```

If `unauthorized`, check your phone screen and accept the RSA key prompt.

### Step 4: Start DroidCam

```bash
adb forward tcp:4747 tcp:4747
droidcam-cli adb 4747
```

On the Android DroidCam app, press **Start** (USB mode auto-detected).

### Step 5: Use in Applications

The video appears at `/dev/video0`. Select "DroidCam" in:

| App | Settings Path |
|-----|--------------|
| **Zoom** | Settings → Video → Camera |
| **Google Meet** | Three-dot menu → Settings → Video |
| **OBS Studio** | Sources → "+" → Video Capture Device |
| **Teams** | Settings → Devices → Camera |
| **Discord** | User Settings → Voice & Video → Camera |

### Step 6: Stop When Done

Press `Ctrl+C` in the terminal running droidcam-cli.

---

## 10. Step-by-Step: Iriun Webcam (Simplest)

### Step 1: Install Apps

1. **On Android:** Install "Iriun Webcam" from Google Play
2. **On Ubuntu:** `wget https://iriun.com/iriun-webcam.deb && sudo dpkg -i iriun-webcam.deb && sudo apt install -f`

### Step 2: Connect

1. Ensure both devices are on the same WiFi network
2. Launch Iriun Webcam on Android
3. Launch Iriun Webcam on Ubuntu (app menu or `iriun-webcam`)
4. Auto-connects within seconds

### Step 3: Use

Appears as "Iriun Webcam" in Zoom, Teams, OBS, or any video application.

---

## 11. Step-by-Step: scrcpy + v4l2loopback (Open Source)

### Step 1: Install & Load

```bash
sudo apt install scrcpy v4l2loopback-dkms adb
sudo modprobe v4l2loopback exclusive_caps=1 max_buffers=2
ls /dev/video*   # Verify virtual device exists
```

### Step 2: Enable USB Debugging

Same as DroidCam (Section 9, Step 1).

### Step 3: Stream Camera

```bash
adb devices   # Verify
scrcpy --video-source=camera --v4l2-sink=/dev/video10 --no-playback --no-audio
```

For front camera at 1080p:

```bash
scrcpy --video-source=camera --camera-facing=front \
       --camera-size=1920x1080 --v4l2-sink=/dev/video10 \
       --no-playback --no-audio
```

### Convenience Script

Save as `~/bin/android-cam.sh`:

```bash
#!/bin/bash
DEV="${1:-/dev/video10}"
CAM="${2:-back}"
if ! lsmod | grep -q v4l2loopback; then
    sudo modprobe v4l2loopback exclusive_caps=1 max_buffers=2
fi
adb devices | grep -q "device$" || { echo "Phone not connected"; exit 1; }
scrcpy --video-source=camera --camera-facing=$CAM \
       --v4l2-sink=$DEV --no-playback --no-audio
```

```bash
chmod +x ~/bin/android-cam.sh
~/bin/android-cam.sh /dev/video10 back
```

---

## 12. Troubleshooting

### Common Issues & Fixes

| Issue | Likely Cause | Solution |
|-------|-------------|----------|
| **ADB not detecting phone** | USB Debugging off or wrong cable | Enable Developer Options → USB Debugging. Change USB mode to MTP/File Transfer. Use a data cable (not charge-only). |
| **Device "unauthorized"** | RSA key not accepted | Check phone screen, accept prompt. Replug cable. |
| **No video / black feed** | Wrong device path or permission | Check `ls /dev/video*`. Try `/dev/video10` instead of `/dev/video0`. Add user to `video` group. |
| **Video choppy/laggy** | Network congestion or high res | Switch from WiFi to USB. Reduce resolution in app settings. |
| **No audio** | Permissions or method limitation | Grant mic permission on Android. DroidCam/Iriun support audio; scrcpy camera audio is experimental. |
| **v4l2loopback not found** | Module not loaded | `sudo modprobe v4l2loopback exclusive_caps=1 max_buffers=2`. Check `ls /dev/video*`. |
| **Permission denied for /dev/videoX** | User not in `video` group | `sudo usermod -aG video $USER` then log out and back in. |
| **Camera not in apps** | App not refreshed | Close and reopen app. Refresh device list in app settings. |
| **Firewall blocking** | Port closed | `sudo ufw allow 4747` (for DroidCam WiFi) |
| **Snap apps can't see camera** | Snap confinement | `sudo snap connect zoom-client:camera core:camera`. Or install the .deb version. |

### Ubuntu 24.04 Specific Notes

- **Wayland vs X11:** Both work. If experiencing permission quirks, select "Ubuntu on Xorg" from the login screen gear icon.
- **PipeWire:** Ubuntu 24.04 uses PipeWire. DroidCam and Iriun work with it. Restart with `systemctl --user restart pipewire` if audio issues arise.
- **Kernel 6.8:** v4l2loopback-dkms from apt works. If Secure Boot is enabled, follow MOK enrollment prompts on first install.
- **Snap Camera Permissions:** Snap-confined apps need explicit camera access granted:

```bash
snap connect zoom-client:camera core:camera
snap connect teams:camera core:camera
```

If issues persist, install non-snap versions (Zoom .deb from zoom.us, Teams via PWA/Flatpak).

---

## 13. Recommendations

### Quick Decision Guide

| Your Situation | Best Method | Why |
|----------------|-------------|-----|
| **Just want it to work, no fuss** | **Iriun Webcam** | Install + connect. Full HD free, no watermark. |
| **Need low latency / USB reliability** | **DroidCam USB** | Very low latency. Most tested. $4.99 for 1080p. |
| **Must be fully open source** | **scrcpy + v4l2loopback** | No proprietary code. Any resolution free. |
| **Streaming / OBS user** | **Iriun or DroidCam + OBS** | V4L2 device → OBS source. |
| **Professional video calls** | **DroidCam USB paid** | 1080p, low latency, audio, reliable. $4.99 well spent. |
| **Need camera switching** | **scrcpy + v4l2loopback** | Easy front/back switch via `--camera-facing` flag. |
| **Ubuntu 24.04 specific** | **AVream** | Tested on 24.04, video + audio combined. |
| **No cable / travel** | **DroidCam WiFi** | Over network, no cable needed. |

### Tiered Verdict

- **🥇 Most users: Iriun Webcam** — Simplest, free HD, no watermark. 2-minute setup.
- **🥇 Reliability seekers: DroidCam USB** — Low latency via USB, most battle-tested. Free 720p or $4.99 for 1080p.
- **🥈 Power users: scrcpy + v4l2loopback** — Full control, zero proprietary software. Best flexibility.

---

## 14. Data Sources

- **DroidCam:** [droidcam.app](https://droidcam.app/) | [dev47apps.com](https://www.dev47apps.com/droidcam/linux/) | [GitHub](https://github.com/dev47apps/droidcam-linux-client)
- **Iriun Webcam:** [iriun.com](https://iriun.com/) | [iriun.net](https://iriun.net/how-to-install-iriun-webcam-in-ubuntu/)
- **scrcpy:** [github.com/Genymobile/scrcpy](https://github.com/Genymobile/scrcpy)
- **v4l2loopback:** [github.com/umlaeute/v4l2loopback](https://github.com/umlaeute/v4l2loopback)
- **AVream:** [github.com/Kacoze/avream](https://github.com/Kacoze/avream)
- **Ubuntu 24.04 packages:** [packages.ubuntu.com/noble](https://packages.ubuntu.com/noble/)
- **Community:** AskUbuntu, Baeldung, It's FOSS, Reddit r/Ubuntu, DEV Community

---

*Tested on Ubuntu 24.04 LTS (Noble Numbat), kernel 6.8. Last updated July 2026.*
