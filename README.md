# ARTCB Holographic Command Surface

A browser-native Three.js + MediaPipe single-hand interaction prototype for the ARTCB demo.

## Run locally

Camera access generally requires HTTPS or localhost. From this directory, start a static HTTP server with Python 3:

`python -m http.server 4173`

Then open `http://localhost:4173` in a current Chromium-based browser. The page imports Three.js and MediaPipe Tasks Vision from public CDNs and downloads the official Hand Landmarker model on demand, so the browser needs network access for those resources.

## Controls

- **Enable camera / Stop camera:** explicit camera permission and shutdown.
- **Mouse drag:** orbit the graph.
- **Mouse wheel:** zoom.
- **Arrow keys:** orbit.
- **+ / -:** zoom.
- **Enter:** select the next node using the keyboard.
- **Escape:** stop camera tracking.
- **Hand movement:** orbit the graph.
- **Thumb-index pinch:** select a node under the projected index fingertip.

## Data and privacy status

- The current nodes and event stream are **synthetic demo fixtures**.
- Live ARTCB telemetry is **not connected**.
- The prototype does not record or transmit camera video intentionally; camera frames are passed to the browser-side MediaPipe task.
- Browser access to external CDN/model resources is required.
- Gestures only control the UI. They do not authorize privileged backend operations.

## Current limitations

This is a P0 prototype, not a production-ready app. Gesture thresholds and pointer mapping need usability tests. Browser execution, device performance, camera-denied behavior, and GPU/CPU delegate fallback still require validation on target devices. No FPS or latency figures are claimed.
