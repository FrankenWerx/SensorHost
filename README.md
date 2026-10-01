# FrankenWerx SensorHost

Custom ESPHome presence + environment sensor built on the Seeed XIAO ESP32-C3 or ESP32-C6.

**Sensors:** LD2410C mmWave presence · AHT20 temp/humidity · BMP280 barometric pressure · ambient light

---

## For recipients — getting your board online

**What you need:** The board, a USB-C power source, and your home WiFi password.

**Prerequisite:** The ESPHome add-on must be installed in your Home Assistant instance (not just the ESPHome integration). If you're not sure, check Settings → Add-ons in Home Assistant and look for ESPHome.

### Step 1 — Power on the board

Plug in via USB-C. The status LED will light up.

### Step 2 — Connect to the setup network

**On Android (easiest):** Open Chrome, go to [improv-wifi.com](https://www.improv-wifi.com/), and follow the Bluetooth prompts to enter your WiFi credentials.

**On any device:** Look for a WiFi network called **"SensorHost Setup"** in your WiFi settings. Connect to it (password provided separately). Your device should automatically open a setup page — if it doesn't, open a browser and go to `192.168.4.1`. Enter your home WiFi network name and password.

### Step 3 — Done

The board will connect to your WiFi and appear in your Home Assistant ESPHome dashboard within a minute or two as a discovered device. Click **Adopt** to add it — this pulls the latest configuration template from this GitHub repo and creates a fully editable entry in your dashboard.

---

## For the maintainer — bench flashing

### Prerequisites

- ESPHome installed (CLI or add-on)
- `secrets.yaml` with at minimum:

```yaml
ap_fallback_password: "YourSharedAPPassword"
```

### Flash a board

The v3 PCB accepts either chip — use the config that matches what's installed:

```bash
# C6 chip
esphome run sensorhost-v3-c6.yaml

# C3 chip (v3 PCB or v2 PCB)
esphome run sensorhost-v3-c3.yaml
```

Every board gets a unique name based on its MAC address (e.g., `sensorhost-aabbcc`). No per-board edits needed.

### Releasing an update

1. Make sensor/config changes in `sensorhost-v3-base.yaml` (both chip variants pick these up automatically)
2. For chip-specific changes, edit the relevant variant file (`sensorhost-v3-c6.yaml` or `sensorhost-v3-c3.yaml`)
3. Bump `project: version:` in the variant file(s) you changed (e.g., `1.0.0` → `1.1.0`)
4. Commit and push to `main`

Each household's ESPHome dashboard independently checks this repo via `dashboard_import`. When the version in the repo is newer than what's installed, their dashboard shows an update available — they can apply it with one click. No access to their instance needed.

---

## Hardware

| PCB revision | Accepted chips | Config file |
|--------------|---------------|-------------|
| v3 (current) | ESP32-C6 | `sensorhost-v3-c6.yaml` |
| v3 (current) | ESP32-C3 | `sensorhost-v3-c3.yaml` |
| v2 | ESP32-C3 only | `sensorhost-v3-c3.yaml` |

v2 and v3 PCBs are nearly identical — the only differences are the onboard LED component and an added capacitor and resistor. All sensor GPIO assignments are the same across both revisions.

Shared sensor config lives in `sensorhost-v3-base.yaml`. Both variant files import it via `packages:`.

## Notes

- C6 adds 802.15.4 / Matter-Thread potential over the C3
- Verify C6 GPIO strapping pin behavior with the LD2410C before committing to C6 for a production run
