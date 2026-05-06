# STL Viewer

Control a 3D model on your computer by physically rotating your phone.

Load any STL file on the desktop, pair your phone by scanning a QR code, then tilt and rotate your phone to orbit the model in real time. Two-finger pinch zooms the model in and out.

---

## Requirements

- A modern desktop browser (Chrome, Firefox, Edge, Safari)
- An iPhone or Android phone with a browser
- Both devices on the same WiFi network (recommended)
- The pages must be served over **HTTPS** — motion sensors require it

---

## Quickstart

### 1. Deploy to GitHub Pages

Push this repository to GitHub, enable GitHub Pages (Settings → Pages → Deploy from branch `main`), then access:

```
https://yourusername.github.io/stl-space-viewer/desktop.html   ← open on computer
```

On the phone, open the camera app and scan the QR shown on desktop. The QR opens a URL like:

```
https://urfdvw.github.io/stl-space-viewer/mobile.html?token=<peer-id>
```

### 2. Local testing with ngrok

```bash
npx serve .          # serves on http://localhost:3000
ngrok http 3000      # creates a public https:// URL
```

Open `http://localhost:3000/desktop.html` on the computer, then scan the desktop QR with your phone camera.

---

## Using the Desktop Page

### Load a model

Click **Choose File** and select any `.stl` file (binary or ASCII). The model loads and auto-centers in the viewport.

### Adjust perspective

Use the **Perspective (FOV)** slider:
- Left end (1°) — nearly flat, orthographic-style projection
- Right end (90°) — wide-angle, strong depth distortion
- Default (30°) — natural perspective

### Mouse controls (desktop)

| Action | Result |
|--------|--------|
| Left-drag | Orbit the model |
| Right-drag or two-finger drag | Pan |
| Scroll wheel | Zoom in/out |

Orbit is automatically disabled while a phone is connected (the phone takes over rotation). Pan and zoom remain available.

### QR code panel

A QR code appears in the sidebar within a few seconds of page load. This is your pairing code — show it to your phone camera.

---

## Using the Mobile Page

### Step 1 — Scan the desktop QR using your phone camera app

The QR opens `mobile.html` with a `token` query parameter. The mobile page connects automatically using that token.

### Step 2 — Enable motion sensors

Tap **Enable Motion Sensors**. On iPhone, a system dialog will ask for permission — tap **Allow**. On Android, sensors start immediately.

### Step 3 — Control the model

Hold the phone naturally and move it:

| Motion | Effect on model |
|--------|----------------|
| Tilt phone left/right | Rotate model left/right |
| Tilt phone forward/backward | Rotate model up/down |
| Rotate phone flat (spin) | Spin model around its axis |

### Reset Pose

Tap **Reset Pose** at any time to re-zero the orientation. The model snaps to its neutral position, and all subsequent rotations are measured relative to the phone's current angle.

**Tip:** hold the phone in the position where you want the model to look "straight-on", then tap Reset Pose.

### Pinch to Zoom

Use two fingers on the phone screen to pinch in or out. This scales the model on the desktop.

---

## Troubleshooting

**QR code does not appear**
The page connects to a free signaling server on startup. It usually takes 1–2 seconds. If it never appears, check your internet connection or try refreshing.

**Phone camera does not open the QR link**
Try scanning again in good lighting and make sure the QR is fully visible on the desktop screen.

**Motion sensors don't work on iPhone**
The **Enable Motion Sensors** button must be tapped — iOS requires a button press to grant orientation access. If you tapped it and still see no response, make sure the page is loaded over HTTPS (not plain HTTP).

**Rotation feels wrong or jumpy after pairing**
Tap **Reset Pose** once while holding the phone in a comfortable position. This calibrates the baseline.

**Model not rotating on desktop after pairing**
Check that "Mobile connected" shows in the desktop sidebar. If it shows "Disconnected", scan the desktop QR again.

**Connection unreliable / doesn't work on mobile data**
WebRTC works best when both devices are on the same WiFi. Mobile carrier networks often block direct peer-to-peer connections.

---

## Supported File Formats

| Format | Support |
|--------|---------|
| Binary STL | ✓ |
| ASCII STL | ✓ |
| OBJ, GLTF, etc. | ✗ (STL only) |
