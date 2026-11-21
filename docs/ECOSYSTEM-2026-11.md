# CEC↔MQTT ecosystem survey — 2026-11-21 (GitHub API)

Question: is our bridge the right base going forward, or migrate to something
better-maintained? **Verdict: stay.** The ballle98 fork + our 2026-11
modernization (paho-mqtt 2.x, venv deployment, reconnect/retry hardening) is
the most modern *maintained* CEC↔MQTT bridge available today.

## Lineage

| Repo | Lang | ★ | Last push | Status |
|---|---|---|---|---|
| michaelarnauts/cec-mqtt-bridge (upstream) | Python | 157 | 2023-09-20 | **archived** by owner |
| **ballle98/cec-mqtt-bridge ← our base** | Python | 10 | 2025-10-08 | most-active of the 42 forks |
| other 41 forks (best: NixM0nk3y) | Python | ≤0 | 2025-01 or older | personal patches, no adoption |

## Alternatives

| Repo | Lang | ★ | Last push | Notes |
|---|---|---|---|---|
| splattner/cec2mqtt (continuation of RobertMe) | Go | 0 | 2026-11-21 (created 11-19!) | docker-first, multi-arch signed images, HA discovery, keypress rx/tx, active source. Watch list #1 |
| RobertMe/cec2mqtt | Go | 4 | 2021-12 | dead, predecessor of splattner's |
| tobiash/hdmi-cec-mqtt | Go | 8 | 2020-02 | dead; env-var config, k8s manifests |
| blchinezu/cec-client-mqtt-bridge | Go | 7 | 2019-07 | dead; shells out to `cec-client` |
| nvella/mqtt-cec | JS | 6 | 2016-10 | dead |
| theodorx7/cec-mqtt-bridge-home-assistant-app | Python | 2 | 2026-10-08 | HA-supervisor *app* packaging of a similar bridge — only useful if the bridge ran under HA OS, not our topology |
| Voxvalve/ha-cec-bridge, nm47/mqtt_cec, cteachworth/cec-mqtt, Khronos31/home-assistant-cec-control | Python | 0 | 2026 | tiny personal projects, no adoption |

Migration isn't warranted: nothing combines adoption + maintenance + our
feature depth (per-device power state, volume/mute with feedback, raw tx/rx,
IR hooks). But the survey surfaced **ideas worth borrowing** (below).

## Ideas to borrow (backlog → bd tickets)

| Idea | Source | Value for us |
|---|---|---|
| **Home Assistant MQTT discovery** — publish `homeassistant/…/config` payloads for TV/AVR power, volume, mute | RobertMe & splattner cec2mqtt | entities auto-appear in HA instead of hand-built automations; biggest UX win |
| **CEC key-press receive → MQTT** (`media/cec/…/key`) | splattner; our `_on_key_press_callback` is debug-only today | TV-remote presses as HA automation triggers (e.g. activity switching) |
| **Active-source get/set** (`SetActiveSource`) | splattner | input switching from HA, not just power |
| **Adapter auto-detect** (`DetectAdapters`) when `port` unset | theodorx7 | replaces the Linux/RPI guessing in hdmicec.py |
| **CI: multi-arch Docker images, signed releases + SBOM** | splattner | repo already has a Dockerfile; add GH Actions workflow |
| Volume scale normalization (`volume_correction` currently 1) | internal observation (published `127`) | AVR-relative volume instead of raw 0–127 CEC scale |

## GitHub fork decision (2026-11-21)

Recommended: **yes, fork ballle98/cec-mqtt-bridge into the owner's GitHub
account** — durable home for our 3 modernization commits (currently
local-only), keeps lineage (`gh` shows forked-from chain), enables the
optional upstream PR to ballle98, and gives the venv's `origin` a pushable
remote. SSH keys exist on mozart; GitHub's host keys verified against
published fingerprints.
