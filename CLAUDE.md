# CLAUDE.md — Technical Reference

## Project Overview

Two standalone HTML files, no build system. All dependencies loaded from CDN.

## Files

| File | Purpose |
|------|---------|
| `desktop.html` | STL viewer with WebGL; WebRTC host; QR code generator |
| `mobile.html` | QR scanner; WebRTC client; device orientation → pose sender |

## CDN Dependencies

| Library | Version | Tag type | Global |
|---------|---------|----------|--------|
| Three.js | 0.168.0 | `importmap` + ES module | — |
| Three.js STLLoader | 0.168.0 | `importmap` + ES module | — |
| Three.js OrbitControls | 0.168.0 | `importmap` + ES module | — |
| PeerJS | 1.5.4 | `<script src>` (before module) | `window.Peer` |
| qrcode.js | 1.0.0 | `<script src>` (before module) | `window.QRCode` |
| jsQR | 1.4.0 | `<script src>` (before module) | `window.jsQR` |

Non-module scripts must be loaded **before** `<script type="module">`. The importmap must appear before any module script.

## Architecture

### WebRTC Signaling Flow

```
Desktop                           PeerJS Cloud (api.peerjs.com)             Mobile
  |                                        |                                   |
  |── new Peer() ──────────────────────────|                                   |
  |<─ peer.on('open', id) ────────────────|                                   |
  |── generateQR(id) (QR shown on screen) |                                   |
  |                                        |   <── jsQR decodes peer ID ──────|
  |                                        |   <── new Peer() ────────────────|
  |                                        |   <── peer.connect(id) ──────────|
  |<─ peer.on('connection', conn) ─────────|─────────────────────────────────|
  |<═══════════════ WebRTC data channel open ═══════════════════════════════>|
  |<─ conn.on('data', pose) ───────────── direct P2P ───── conn.send(pose) ──|
```

- Desktop is always the **host** (creates Peer, waits for connection).
- QR encodes only the alphanumeric peer ID string (not a URL).
- After handshake, data flows peer-to-peer via WebRTC (STUN: `stun.l.google.com:19302`). No TURN relay — both devices should be on the same WiFi for reliability.

### Data Protocol (mobile → desktop)

```js
// Sent at ~30fps while sensors are active
{ type: 'pose', q: [x, y, z, w], scale: 1.0 }
```

`q` is a unit quaternion (Three.js `Quaternion` component order). `scale` is a multiplicative factor applied on top of the model's normalization scale.

### Pose Math

**Euler → Quaternion (device orientation):**
```
Euler order 'ZXY':
  x = degToRad(beta)    (pitch: front/back tilt)
  y = degToRad(alpha)   (yaw: compass heading)
  z = degToRad(gamma)   (roll: left/right tilt)
```
This matches iOS `DeviceOrientationEvent` convention (same as Three.js `DeviceOrientationControls`).

**Pose reset (quaternion delta):**
```
At reset:      baselineQ = currentDeviceQ
Each frame:    deltaQ = inverse(baselineQ) × currentDeviceQ
Desktop sets:  modelMesh.quaternion = deltaQ
```
When phone is in the same orientation as at reset time, `deltaQ = identity` → model shows neutral pose.

**Pinch zoom:**
```
scale *= distance_new / distance_prev   (multiplicative delta each touchmove)
scale = clamp(scale, 0.1, 10)
```

### STL Loading & Normalization

1. `FileReader` → `ArrayBuffer` → `STLLoader.parse()` → `BufferGeometry`
2. `computeBoundingBox()` → translate geometry to center at origin
3. `computeBoundingSphere()` → `baseScale = 2 / radius` (fits model in a 2-unit sphere)
4. `modelMesh.scale.setScalar(baseScale)` — stored as `baseScale` for later use
5. Incoming scale factor multiplied by `baseScale`: `modelMesh.scale.setScalar(data.scale * baseScale)`

### FOV / Perspective Slider

Single `PerspectiveCamera`. Slider directly sets `camera.fov` (1°–90°), followed by `camera.updateProjectionMatrix()`.

| Slider value | Visual effect |
|---|---|
| 1° | Nearly orthographic (parallel projection feel) |
| 30° | Default — natural perspective |
| 90° | Wide-angle, strong perspective distortion |

### OrbitControls Behavior

- Default: full orbit/zoom/pan enabled (user can interact with mouse on desktop).
- On mobile connect: `controls.enableRotate = false` (rotation yielded to WebRTC pose data; zoom/pan remain active).
- On mobile disconnect: `controls.enableRotate = true` restored.

## iOS-Specific Notes

- `DeviceOrientationEvent.requestPermission()` exists only on iOS 13+. Feature-detected with `typeof DeviceOrientationEvent.requestPermission === 'function'`.
- Must be called from a direct user gesture (button click). Cannot be called from `DOMContentLoaded` or any async chain not traceable to a click.
- Permission is per-origin and per-browser-session (not persisted across reloads).

## HTTPS Requirement

Both camera (`getUserMedia`) and device orientation (`DeviceOrientationEvent`) require a secure context:

- `https://` origin, or
- `localhost` (camera works; device orientation may still need HTTPS on some iOS versions)

**Recommended deployment:** GitHub Pages (`https://username.github.io/stl-viewer/`).
**Local testing:** `npx serve .` + ngrok for the mobile URL.

## Known Limitations

- No TURN server — WebRTC may fail on mobile carrier networks (CGNAT). Both devices on the same WiFi is recommended.
- Position/translation from device IMU is not implemented (double-integrating accelerometer is too noisy). Rotation and pinch-zoom only.
- PeerJS free cloud server (`api.peerjs.com`) may have availability/rate-limit constraints. For production, self-host `peerjs-server`.
- Large STL files (>50MB) may briefly block the main thread during parsing (no Web Worker).
- One mobile controller at a time — if a second device connects, it replaces the first.
