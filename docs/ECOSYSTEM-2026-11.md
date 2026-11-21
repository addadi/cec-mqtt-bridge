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

## Direct forks of ballle98 (surveyed 2026-11-21)

| Fork | ★ | Last push | vs ballle98 master | Notes |
|---|---|---|---|---|
| **addadi/cec-mqtt-bridge (ours)** | 0 | 2026-11-21 | ahead 4 | paho-2 migration, venv deploy, docs |
| **Markkuuss/cec-mqtt-bridge** | 1 | 2026-05-03 | **ahead 15** (branch `integration`) | per-device keypress→MQTT w/ duration+throttle config, optional cec/lirc imports, venv install script, Trixie refactors — **adopt into ours** (bd ticket) |
| michaeldeborst | 0 | 2025-10-04 | behind 3 | author of upstream PR #20, nothing new |
| clockbrain | 0 | 2024-03-05 | — | untouched since 2024 |

## Why we forked ballle98, not archived michaelarnauts (157★)

The upstream's stars are frozen — archived 2023-09, no issues/PRs possible.
ballle98's master contains ~2 years of post-archive fixes (CEC-adapter-on-vc4
fix #19, install-docs #17, PR #20) that a michaelarnauts fork would lack or
duplicate. We sit in the *same* fork network (michaelarnauts → ballle98 →
addadi), so we remain discoverable from the archived repo's fork tree, while
keeping a live upstream we can PR into. Traction comes from PRing ballle98
(paho-2 fix + Markkuuss's keypress), not from the archived parent.


## Ideas to borrow (backlog → bd tickets)

| Idea | Source | Value for us |
|---|---|---|
| **Home Assistant MQTT discovery** — publish `homeassistant/…/config` payloads for TV/AVR power, volume, mute | RobertMe & splattner cec2mqtt | entities auto-appear in HA instead of hand-built automations; biggest UX win |
| **CEC key-press receive → MQTT** (`media/cec/…/key`) | splattner; our `_on_key_press_callback` is debug-only today | TV-remote presses as HA automation triggers (e.g. activity switching) |
| **Active-source get/set** (`SetActiveSource`) | splattner | input switching from HA, not just power |
| **Adapter auto-detect** (`DetectAdapters`) when `port` unset | theodorx7 | replaces the Linux/RPI guessing in hdmicec.py |
| **CI: multi-arch Docker images, signed releases + SBOM** | splattner | repo already has a Dockerfile; add GH Actions workflow |
| Volume scale normalization (`volume_correction` currently 1) | internal observation (published `127`) | AVR-relative volume instead of raw 0–127 CEC scale |

## GitHub fork decision — EXECUTED 2026-11-21

**Done:** fork created at `github.com/addadi/cec-mqtt-bridge`, all 4
modernization commits pushed (master = `668d5a7`), remotes normalized
(`origin` = addadi fork over SSH, `upstream` = ballle98 over HTTPS).
Key `id_rsa.git` verified registered on the account. Optional follow-up:
open an upstream PR to ballle98 with the paho-mqtt 2.x migration.
