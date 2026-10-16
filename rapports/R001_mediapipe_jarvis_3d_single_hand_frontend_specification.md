# R001 — MediaPipe Research and ARTCB TERMINATOR Frontend: Single-Hand 3D JARVIS Command Environment

**Status:** Design proposal; no frontend implementation or live integration validated  
**Date:** 16 October 2026  
**Target repository:** `vgactech/artcb_TERMINATOR_FRONTEND`  
**Scope:** This document only. No changes to ARTCB core, ARTCB-TERMINATOR backend, VLC&ARTCB, secrets, runtime configuration, or logs.

## 1. Executive decision

Build an ARTCB-specific, browser-based 3D command environment inspired by cinematic JARVIS interfaces, but designed around real defensive telemetry rather than decorative animation. The interface should combine:

- A navigable 3D graph of agents, sub-agents, tools, events, resources, evidence and causal relationships.
- Single-hand gesture interaction through a standard webcam.
- Voice commands and spoken commentary.
- Mouse, keyboard and touch fallbacks.
- Live and historical timelines, incident replay, evidence inspection and a pitch/demo mode.
- A clear distinction between live evidence, inferred analysis, simulated demo data and unavailable telemetry.

**Recommended first stack:** TypeScript + React + Three.js / React Three Fiber + MediaPipe Tasks Vision. Use Web Workers for hand inference where practical. Keep voice and security decisions independent: audio generation must never block, delay or alter a defensive decision.

## 2. MediaPipe repository and how it works

### Official repository

- Main repository: https://github.com/google-ai-edge/mediapipe
- Official web samples: https://github.com/google-ai-edge/mediapipe-samples-web
- Official web vision task documentation: https://github.com/google-ai-edge/mediapipe/blob/master/mediapipe/tasks/web/vision/README.md
- Hand Landmarker guide: https://developers.google.com/edge/mediapipe/solutions/vision/hand_landmarker/web_js
- Gesture Recognizer guide: https://developers.google.com/edge/mediapipe/solutions/vision/gesture_recognizer/web_js

MediaPipe is a cross-platform framework and collection of on-device machine-learning tasks for processing live and streaming media. It is not itself a 3D rendering engine and it does not build the JARVIS interface. It supplies perception outputs that the frontend maps to UI actions.

### Process — from camera to a gesture-controlled 3D scene

1. The browser requests camera permission through an explicit user action.
2. The camera supplies frames to MediaPipe Tasks Vision.
3. Hand Landmarker estimates hand landmarks (21 landmarks per detected hand) and associated handedness/world-coordinate outputs where supported.
4. Gesture Recognizer can classify supported gestures and also provide landmark results. For richer custom interactions, application code can derive gestures from landmark geometry and temporal movement.
5. A gesture state machine checks confidence, duration, cooldown and current interaction mode.
6. A semantic command is sent to the frontend state store, for example `GRAPH_ROTATE`, `SELECT_NODE`, `ZOOM` or `OPEN_INCIDENT`.
7. Three.js renders the updated scene. A separate event adapter can ingest real ARTCB telemetry and update graph nodes, links and timeline events.

### Problem — landmarks are not perfect physical tracking

A webcam-based monocular model does not provide a guaranteed, metrically accurate 3D measurement of the user's hand. Depth estimates may be approximate; occlusion, lighting, camera angle, skin/background contrast and fast movement can reduce reliability. A hand landmark must not be treated as proof of user identity or as authorization for privileged operations.

### Solution

- Use MediaPipe for interaction convenience, not authentication.
- Smooth motion with a low-latency filter and add hysteresis to gesture transitions.
- Require a dwell interval for selections and a release/re-arm cycle after actions.
- Calibrate pinch thresholds for each session; provide sensitivity controls.
- Show tracking confidence and a visible “hands unavailable” state.
- Keep mouse/keyboard/touch controls available at all times.
- Keep camera processing local where possible, disclose permissions and never record or transmit camera footage by default.
- Use explicit confirmation for consequential operations. Gestures can navigate and inspect; they should not alone authorize containment, deletion, credential rotation or other high-impact actions.

## 3. Why MediaPipe is suitable

The official web samples include Hand Landmarker, Gesture Recognizer, Holistic Landmarker, Pose Landmarker and other on-device tasks. The web samples are a practical starting point for browser integration. The official documentation states that task inputs are processed on-device, although product metrics may be sent; applications remain responsible for privacy disclosure and consent.

MediaPipe supplies perception. Three.js supplies the scene graph, cameras, geometry, materials, lights, shaders and rendering. React Three Fiber is an optional React renderer for Three.js that can make UI integration easier. Neither MediaPipe nor Three.js automatically provides a complete interaction design, robust gesture state machine, ARTCB data contract or reliable event provenance; those are application responsibilities.

## 4. Proposed experience and information architecture

### Main screen

**Central 3D provenance graph**
- Nodes: user prompt, agent, sub-agent, tool/API, resource, security control, incident, evidence object and final outcome.
- Edges: called, read, wrote, transformed, delegated, detected, blocked, caused, verified and derived-from.
- Time: event-time and ingestion-time are distinct.
- Node details: ID, type, source, status, timestamps, confidence, provenance references and evidence links.
- Selection opens a side panel without destroying graph context.

**Peripheral HUD panels**
- Left: event timeline, filters and severity/status legend.
- Right: selected object details, evidence and integrity status.
- Bottom: command input, microphone state, voice transcript and safe action controls.
- Top: live/demo mode, connection health, telemetry coverage and clock status.

**Time controls**
- Live-follow mode.
- Pause visual playback without stopping data collection.
- Scrub and replay recorded events.
- Compare pre-incident and post-remediation state.
- Show missing telemetry and out-of-order events explicitly.

**Pitch mode**
- A curated scenario demonstrates: Attack → Detection → Containment → Evidence → Replay → Proof.
- Clearly label synthetic events as “SIMULATED DEMO DATA”.
- Use a guided camera path and optional narration.
- Never represent simulated or inferred activity as live, cryptographically verified evidence.

## 5. Single-hand interaction model

Start with one-hand interaction; do not make two hands a requirement. Provide an onboarding/calibration flow and a small, stable gesture vocabulary.

| Gesture | Proposed action | Safety/UX rule |
|---|---|---|
| Open palm held briefly | Enter/enable hand interaction or show command palette | Must not execute a destructive action |
| Index finger point | Move a reticle / hover a node | Hover is not a selection |
| Thumb-index pinch | Select / grab a node | Dwell and confidence threshold |
| Pinch + move | Drag or orbit selected graph object | Bounded movement; release cancels drag |
| Closed fist held | Pause visual navigation / clutch | Avoid mapping to irreversible actions |
| Open palm then pinch | Confirm a reversible UI selection | Explicit preview and cancel |
| Hand leaves frame | Return to neutral / suspend gesture input | Do not reuse stale coordinates |

Treat this as a proposal to test, not a claim that every gesture is already implemented. Gesture mappings must be configurable and tested across left/right hands, camera mirrors and accessibility needs.

### Critical interaction mechanics

- **Dead-man / clutch behavior:** movement only occurs while a deliberate gesture is held; release stops movement.
- **Hysteresis:** use different thresholds for entering and leaving a gesture to avoid flicker.
- **Dwell:** require a short stable interval before selection.
- **Cooldown:** prevent a sustained gesture from firing repeatedly.
- **Neutral reset:** tracking loss resets active gestures safely.
- **Command preview:** show the interpreted action before executing a consequential command.
- **Fallback:** provide visible buttons and keyboard shortcuts for every important navigation function.

## 6. Voice interaction

Voice should be a separate interaction channel writing into the same semantic command layer as gestures and mouse controls.

Example commands:
- “Show the origin of this alert.”
- “Follow the path from the prompt to the blocked tool call.”
- “Filter to confirmed events.”
- “Compare the incident before and after containment.”
- “Explain which evidence supports this conclusion.”
- “Start the pitch scenario.”
- “Stop narration.”

Recommended pipeline:
1. User explicitly activates microphone or uses a clear push-to-talk control.
2. Speech recognition creates a transcript and confidence value.
3. Intent parser maps the utterance to a limited command schema.
4. UI previews ambiguous or consequential commands.
5. Command router updates the application state or invokes an authorized backend API.
6. Text narration is generated from observed state and verified events.
7. TTS runs asynchronously and cannot block the event ingestion, policy engine or defensive response.

Do not let a voice model invent event status, evidence or cryptographic verification. Narration must distinguish observed facts, analytical hypotheses, simulated events and unknowns. Do not expose API keys in frontend bundles; any provider secret belongs in an authorized backend secret store, never in client code or this report.

## 7. Proposed technical architecture

**Presentation**
- React + TypeScript.
- Three.js, optionally via React Three Fiber.
- HTML/CSS accessible panels layered over the canvas.
- A semantic state store shared by gesture, voice, mouse and keyboard inputs.

**Perception**
- `@mediapipe/tasks-vision`.
- Hand Landmarker first; Gesture Recognizer where its supported categories fit.
- Run frame processing off the main UI thread when feasible.
- Keep only the most recent pending frame to prevent inference backlog.

**Domain layer**
- Gesture state machine and configurable mapping.
- Command interpreter with allowlisted semantic actions.
- Graph model and event normalization.
- Timeline/replay state separate from live ingestion state.

**Data integration**
- Adapter for the actual ARTCB event stream/API, to be selected only after inspecting the frontend/backend contracts.
- Use event IDs and correlation IDs for linking events across agents and services.
- Preserve event-time, received-time, source, version and provenance fields.
- Mark telemetry gaps; absence of events is not proof that no activity occurred.

**Rendering**
- Instanced geometry or level-of-detail for large graphs.
- Spatial clustering, edge bundling and progressive expansion.
- Selection/highlight should not depend on color alone.
- Maintain a 2D/table alternative for accessibility, dense graphs and low-performance hardware.

**Testing**
- Unit tests for gesture math, hysteresis, cooldown and command mapping.
- Replayable camera/landmark fixtures for deterministic gesture tests.
- Component tests for selection, timeline, filters and mode labels.
- Integration tests with synthetic event streams.
- End-to-end tests with camera denied, camera lost, voice unavailable, backend disconnected and GPU fallback.
- Performance tests on target devices before setting hard FPS/latency targets.

## 8. Performance and reliability criteria to validate

These are proposed acceptance targets, not measured current results:

- Interaction remains responsive while event ingestion continues.
- Gesture processing does not block the rendering/UI thread.
- The UI recovers cleanly when camera permission is denied or tracking is lost.
- Voice/TTS failure cannot delay defensive event processing.
- Live data, simulated data and replay data are visually distinguishable.
- Event IDs and evidence references remain stable across filtering and replay.
- No event disappears from the stored history because a visual filter is applied.
- All important actions remain possible without hand tracking.
- A production build and automated tests pass in CI.
- Performance budgets are measured on defined reference devices; do not claim a frame rate or latency until tested.

## 9. Three visual references to inspect

The following references illustrate different approaches. They are inspiration and research material, not endorsements and not proof that their code can be transplanted without review.

### Reference A — MediaPipe hand tracking + Unity 3D
YouTube: https://www.youtube.com/watch?v=ocB7OnGq14M

What to study:
- Hand landmarks and gesture recognition.
- Mapping hand movement to a 3D application's depth axis.
- How the demo explains the interaction to a viewer.
- The difference between a controlled Unity environment and a browser-native frontend.

### Reference B — MediaPipe + Three.js / WebGL hand tracking
YouTube: https://www.youtube.com/watch?v=9oVfWFgt6BM

What to study:
- Browser-based 3D rendering.
- Visual feedback that makes hand movement understandable.
- How a camera-driven interaction can control 3D content without special hardware.
- What must be redesigned for dense security graphs rather than motion-capture content.

### Reference C — Search current holographic / JARVIS-style interaction demos
YouTube search: https://www.youtube.com/results?search_query=MediaPipe+Three.js+JARVIS+holographic+hand+tracking+voice+assistant

Use this search to compare newer projects that combine hand tracking, voice and holographic UI. Video availability and quality can change; this is a search link, not a claim that one specific third video has been independently verified.

### Additional repositories worth examining

- Official MediaPipe: https://github.com/google-ai-edge/mediapipe
- Official web samples: https://github.com/google-ai-edge/mediapipe-samples-web
- JARVIS-style hand/voice holographic prototype: https://github.com/jvaughan007/jarvis
- J.E.S.T.E.R gesture-controlled holographic UI: https://github.com/heeelol/jester
- React + Three.js + MediaPipe hologram: https://github.com/PyRobix-Tech/3DWebsite_mediapipe
- Hand tracking and 3D scene control: https://github.com/AbijahKaj/handtracker-3d

Before adopting code from any third-party repository, inspect its current state, license, dependency health, security posture and actual run/build status. A README is not proof of a reproducible working build.

## 10. Process, problem, solution

### A. Single-hand usability
**Process:** camera frames become landmarks, landmarks become gestures, gestures become semantic commands and the scene updates.

**Problem:** jitter, occlusion, accidental triggers and physical fatigue can make an impressive prototype frustrating in daily use.

**Solution:** use a small gesture vocabulary, calibration, hysteresis, dwell, clutch behavior, neutral reset and always-visible alternatives. Test with varied users and camera setups.

### B. Security evidence visualization
**Process:** real events arrive from instrumented services and are correlated into a graph and timeline.

**Problem:** a beautiful graph can create false confidence if events are missing, simulated or only inferred.

**Solution:** display source, confidence, evidence references and telemetry coverage. Use explicit labels for verified, observed, inferred, simulated and unknown states.

### C. Voice commentary
**Process:** state is summarized into speech and audio is played asynchronously.

**Problem:** speech synthesis or an external provider can stall, fail or narrate stale information.

**Solution:** decouple narration from defensive processing, cancel stale narration, attach event/correlation IDs to generated commentary, and never let TTS determine a security decision.

### D. 3D complexity
**Process:** graph nodes and edges are rendered as interactive geometry.

**Problem:** large graphs cause clutter, occlusion and GPU load.

**Solution:** progressive disclosure, filters, clusters, temporal slices, focus mode, a 2D fallback and measured performance budgets.

## 11. Delivery roadmap

### P0 — Research and interaction prototype
- Confirm frontend repository structure and framework.
- Build a non-security-critical 3D graph with synthetic, explicitly labelled data.
- Add camera permission flow and one-hand calibration.
- Implement point, pinch-select, drag/orbit, clutch and neutral-reset gestures.
- Keep mouse/keyboard equivalents.
- Create deterministic demo replay.

### P1 — Real data integration
- Inspect and document the actual backend event/API contract.
- Connect live events through an adapter.
- Add source labels, event-time/ingestion-time, evidence references and telemetry-gap indicators.
- Verify the UI renders actual events rather than silently falling back to demo data.

### P2 — Voice and pitch mode
- Add push-to-talk and speech transcription.
- Add constrained command routing and safe confirmations.
- Add asynchronous narration with event-linked statements.
- Build a guided demo narrative around Attack → Detection → Containment → Evidence → Replay → Proof.

### P3 — Hardening and evaluation
- Accessibility and 2D alternatives.
- Privacy and permission review.
- Gesture false-positive and false-negative evaluation.
- Camera, microphone, backend and GPU failure testing.
- Build/test automation, dependency/license review and performance profiling.

## 12. Non-goals and boundaries

- This proposal does not implement or claim a working frontend.
- This proposal does not change ARTCB backend logic or any other repository.
- This proposal does not grant camera/microphone permission silently.
- Gestures and voice do not bypass existing authorization or confirmation requirements.
- No API secret, token, runtime log or personal camera recording belongs in the repository.
- No live telemetry should be fabricated for a pitch; synthetic scenarios must be visibly labelled.

## 13. Acceptance checklist

- [ ] The user can navigate the graph with one hand.
- [ ] All core interactions have mouse/keyboard alternatives.
- [ ] Tracking loss returns the gesture state machine to neutral.
- [ ] The user can disable camera and microphone independently.
- [ ] Live, simulated, replayed, inferred and verified states are visually distinct.
- [ ] Graph selection opens evidence details without losing context.
- [ ] Timeline filtering never deletes underlying history.
- [ ] Voice/TTS failures do not block defensive event processing.
- [ ] Real backend integration is proven with correlated events and test evidence.
- [ ] Automated tests, production build and device performance are recorded before claiming readiness.

## 14. Sources

1. MediaPipe repository — https://github.com/google-ai-edge/mediapipe
2. MediaPipe web samples — https://github.com/google-ai-edge/mediapipe-samples-web
3. MediaPipe web vision tasks — https://github.com/google-ai-edge/mediapipe/blob/master/mediapipe/tasks/web/vision/README.md
4. Hand Landmarker guide — https://developers.google.com/edge/mediapipe/solutions/vision/hand_landmarker/web_js
5. Gesture Recognizer guide — https://developers.google.com/edge/mediapipe/solutions/vision/gesture_recognizer/web_js
6. Three.js — https://threejs.org/
7. Video A — https://www.youtube.com/watch?v=ocB7OnGq14M
8. Video B — https://www.youtube.com/watch?v=9oVfWFgt6BM
9. Video C search — https://www.youtube.com/results?search_query=MediaPipe+Three.js+JARVIS+holographic+hand+tracking+voice+assistant

---
**Audit note:** Repository discovery returned `vgactech/artcb_TERMINATOR_FRONTEND` as a public repository on `main`, with GitHub reporting size 0 at the time of discovery. No existing report numbering or project files were available to inspect. This proposal therefore uses `R001` as the first report number in this repository; confirm the numbering convention if prior reports exist outside the visible repository state.
