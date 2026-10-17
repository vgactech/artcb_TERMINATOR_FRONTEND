# R002 — MediaPipe Hologram P0 Hardening and Track 3 Fit

**Date:** 17 October 2026  
**Repository:** `vgactech/artcb_TERMINATOR_FRONTEND`  
**Implementation branch:** `feature/mediapipe-hologram-p0-hardening`  
**Scope:** Frontend-only prototype hardening. No changes to ARTCB backend, `artcb_kyc`, VLC&ARTCB, secrets, or runtime logs.

## Executive summary

The current `index.html` already contains a browser-based Three.js holographic provenance graph and MediaPipe Hand Landmarker integration. The interface explicitly labels its graph and events as simulated demo data and reports that the live ARTCB link is not connected. This is an interaction prototype, not a live telemetry integration.

This iteration hardens two interaction paths:
- Pinch selection now raycasts from the projected index fingertip and selects the graph node under that point instead of cycling through nodes regardless of where the hand points.
- MediaPipe initialization now tries the GPU delegate first and falls back to CPU if the GPU delegate cannot initialize. Initialization failure also closes a partially created landmarker and clears the camera stream reference.

## Expertise activated

- Browser-based 3D rendering and Three.js raycasting.
- MediaPipe Tasks Vision and webcam lifecycle.
- Gesture-to-command interaction design.
- Git change isolation and reviewable delivery.
- Screenpipe hackathon Track 3 demo design and acceptance criteria.

## Process — how the prototype works

1. The user explicitly selects **Enable camera**. The browser requests camera permission.
2. MediaPipe Hand Landmarker processes video frames and estimates hand landmarks.
3. The index fingertip is mapped to normalized screen coordinates. A pinch gesture triggers a Three.js raycast against graph nodes.
4. A selected node updates the object ID and type in the provenance panel.
5. Mouse, keyboard, and touch-capable pointer input remain available as fallbacks.

The scene is still populated with fixed synthetic fixtures. The frontend does not claim to be connected to a live ARTCB event stream.

## Problems found and corrections

### 1. Pinch selection did not follow the fingertip

**Problem:** the earlier pinch handler selected the next node in a list. The result could contradict where the user was pointing, weakening the perceived link between hand movement and the 3D graph.

**Correction:** the handler now maps the index fingertip into scene pointer coordinates and raycasts the graph nodes. If no node is under the fingertip, the interface reports that result instead of silently selecting a different object.

**Remaining limitation:** the fingertip-to-screen mapping is a simple normalized mapping. It still requires real-device usability testing across camera mirroring, viewport aspect ratios, left/right hands, and varied lighting. Selection is not yet a dwell-confirmed interaction.

### 2. GPU-only initialization could make hand tracking unavailable

**Problem:** some browser/device combinations may fail to initialize the GPU delegate even when CPU inference is possible.

**Correction:** initialization attempts GPU first and retries with CPU if GPU initialization fails.

**Remaining limitation:** CPU fallback may have lower frame throughput. No latency, FPS, or hardware compatibility claim is made without measurement.

### 3. Partial initialization could leave camera resources in a stale state

**Problem:** a failure after camera permission was granted could leave the stream referenced by the video element or leave a partially created landmarker alive.

**Correction:** the error path now closes the landmarker when present, stops stream tracks, clears `video.srcObject`, resets the camera controls, and returns the interaction label to mouse/keyboard.

## Track 3 relevance

The Screenpipe Track 3 examples define the goal as making one person's workflow usable by another person, including evidence review, decisions, and a new case. For ARTCB, the 3D scene can support that goal only if the demo sequence shows a reviewer following an event path, inspecting evidence provenance, and explaining the result to a teammate.

Recommended judging sequence:
1. Identify the scenario as simulated fixture data.
2. Select an agent, tool call, policy gate, and evidence node.
3. Explain the relationships in the graph and the event sequence.
4. Replay the same deterministic case for a second viewer.
5. State clearly that live backend integration is not yet connected.

The official examples recommend fictional or permissioned data, clear explanation of what runs locally, and a second person completing a fresh case. The frontend should therefore avoid presenting fixture events as real detections.

## Validation status

- Source fragments for the four hardening changes were matched before applying edits.
- Automated browser execution and webcam validation have **not** been run in this environment.
- A production build cannot yet be reported: this repository currently has a standalone `index.html` rather than a package manifest, test runner, or build pipeline.
- CDN reachability, browser camera permissions, GPU/CPU delegate behavior, and real device performance remain to be verified in a browser.
- Live ARTCB telemetry remains disconnected by design in this P0 prototype.

## Next actions

1. Add a small project README documenting secure local serving and browser requirements.
2. Verify syntax and run a browser smoke test on localhost or HTTPS.
3. Add a deterministic gesture-state test harness and test camera denied/lost scenarios.
4. Add a visible pointer/reticle and dwell-based pinch selection after usability testing.
5. Inspect the real backend event contract before implementing a live data adapter. Keep simulated and live modes separate and unmistakably labelled.
6. Prepare a second-person walkthrough and record acceptance evidence for the Track 3 submission.

## References

- Track 3 examples: https://github.com/screenpipe/hackathon-starter/blob/main/docs/track-3-examples.md
- Frontend specification R001: https://github.com/vgactech/artcb_TERMINATOR_FRONTEND/blob/main/rapports/R001_mediapipe_jarvis_3d_single_hand_frontend_specification.md
- MediaPipe Hand Landmarker web guide: https://developers.google.com/edge/mediapipe/solutions/vision/hand_landmarker/web_js
- Three.js: https://threejs.org/
