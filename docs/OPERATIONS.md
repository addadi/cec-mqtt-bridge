# Operations runbook — cec-mqtt-bridge on mozart

Single canonical deployment of the CEC↔MQTT bridge. Repo:
`~/src/cec-mqtt-bridge-ballle98` (fork lineage: michaelarnauts → ballle98 →
local modernization, see `ECOSYSTEM-2026-11.md`).

## Runtime architecture (as of 2026-11-21)

```
/etc/systemd/system/cec-mqtt-bridge.service   (live unit, enabled, Restart=always)
  └─ ExecStart ~/.local/bin/cec-mqtt-bridge   (USER-OWNED SHIM — not a pip script!)
       └─ exec env -u PYTHONPATH ~/venvs/cec-mqtt-bridge/bin/cec-mqtt-bridge -c -f <repo>/config.ini
            └─ venv: python3.14 --system-site-packages
                 ├─ paho-mqtt 2.2.0            (venv)
                 ├─ cec (libcec python)        (system, /usr/lib/python3.14/site-packages)
                 └─ cec_mqtt_bridge            (editable → ~/src/cec-mqtt-bridge-ballle98/src)
```

- The `env -u PYTHONPATH` in the shim neutralizes the unit's stale
  `PYTHONPATH=~/.local/lib/python3.13/site-packages` (root cause of the hybrid
  import during the 2026-11 incident — see `INCIDENT-2026-11-21.md`).
- Editable install ⇒ code changes in the repo take effect on the next restart.

## Everyday operations

### Reload with new code (NO sudo needed)

```bash
git -C ~/src/cec-mqtt-bridge-ballle98 pull   # or commit locally
kill "$(systemctl show -p MainPID --value cec-mqtt-bridge)"   # systemd revives in 10 s
```

If dependencies changed: `~/venvs/cec-mqtt-bridge/bin/pip install -e ~/src/cec-mqtt-bridge-ballle98` first.

### Verify health

```bash
systemctl status cec-mqtt-bridge --no-pager          # active, uptime growing
journalctl -u cec-mqtt-bridge -n 30 --no-pager       # no tracebacks
# Expect on start: "Connected successfully" + "Connected to HDMI-CEC with ID <n>"
```

MQTT-side check (broker `homeassistant.brahms:1883`, creds in `config.ini`):

```bash
~/venvs/cec-mqtt-bridge/bin/python - <<'EOF'
import paho.mqtt.client as mqtt, time
def on_connect(c,u,f,rc,p): c.subscribe("media/#"); print("connected:", rc)
def on_message(c,u,m): print(m.topic, "=", m.payload.decode(), "(retained)" if m.retain else "")
c = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2, client_id="healthcheck")
c.username_pw_set("mosquitto", open("/dev/null").read() or "")  # use creds from config.ini
c.on_connect, c.on_message = on_connect, on_message
c.connect("homeassistant.brahms", 1883, 60); c.loop_start(); time.sleep(5); c.loop_stop()
EOF
```

Pass: `media/bridge/status = online` retained + device topics present.

### Config

`config.ini` in the repo (git-tracked; contains broker creds — treat the repo
as private). CEC `port=Linux` (kernel `/dev/cec0`), `devices=0..15`,
`refresh=0` (event-driven; set >0 for periodic polling, min 10 s).
Every key can be overridden via env `SECTION_KEY` (e.g. `MQTT_BROKER`).

## Kodi coexistence

`kodi.service` also uses libcec. On Linux the kernel CEC device (`/dev/cec0`)
supports multiple openers (receives fanned out, transmits serialized), so both
can run. Boot-order races are covered by the bridge's CEC open retry
(6 × 10 s). If the adapter genuinely cannot be opened, the bridge exits and
systemd restarts it every 10 s — check `journalctl -u kodi` for who holds it.

## Optional hardening (needs sudo / human)

The repo ships a modernized unit (`cec-mqtt-bridge.service` at repo root)
pointing directly at the venv (no PYTHONPATH, no shim). To adopt:

```bash
sudo cp ~/src/cec-mqtt-bridge-ballle98/cec-mqtt-bridge.service /etc/systemd/system/
sudo systemctl daemon-reload && sudo systemctl restart cec-mqtt-bridge
rm ~/.local/bin/cec-mqtt-bridge    # shim no longer needed
```

## Known-stale artifacts (removal pending user nod)

- `~/.local/lib/python3.13/site-packages/cec_mqtt_bridge{,-0.0.1.dist-info}` —
  the stale `pip --user` install from the incident. Nothing references it.
