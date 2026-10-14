# SensorHost — Maintainer Flashing Guide

Step-by-step for building and shipping a new board.

---

## Prerequisites

- ESPHome installed — CLI (`pip install esphome`) or via the Home Assistant add-on. ESPHome 2026.7 and later need **Python 3.12+**; on an older Python, pip silently stops at 2026.6.x. Build with the same ESPHome version your Home Assistant add-on runs.
- A data-capable USB-C cable (charge-only cables will not work)
- The repo files on your machine (clone or local copy)
- **Windows only:** set a short build folder once, or ESP-IDF builds fail with "filename too long": `setx ESPHOME_BUILD_PATH C:\esphome-build` (then reopen your terminal). Builds land in `C:\esphome-build\<device name>`; switching between the C3 and C6 variants triggers a clean rebuild. Do not put `build_path:` in the shared YAMLs — every adopted config inherits it, and boards end up sharing one build folder.
- **Windows, if your user folder name contains a space:** ESP-IDF builds fail with `ninja: error: 'C:/Users/<first word>', needed by ... missing`. ESPHome downloads ESP-IDF under your user folder by default, so move it once: `setx ESPHOME_ESP_IDF_PREFIX C:\esphome-idf`, reopen your terminal, then `esphome clean <config>.yaml` before rebuilding.

---

## Step 1 — Choose the hotspot password

No `secrets.yaml` is needed. The "SensorHost Setup" fallback hotspot password defaults to `sensorhost`, which is set in `sensorhost-v3-base.yaml`. Because the repo is public, treat that default as public. The hotspot only runs while the board can't reach its WiFi.

To give a board its own password, add `-s ap_password YourPassword` to the flash command in Step 4, e.g.:

```bash
esphome -s ap_password YourPassword run sensorhost-v3-c6.yaml
```

> **Note:** You do NOT need `wifi_ssid` or `wifi_password` either. The template has no pre-configured WiFi — the board will boot straight into provisioning mode so the recipient can enter their own credentials.

If you use a non-default password, write it down. You will need to communicate it to the recipient (or include it on the printed setup card).

---

## Step 2 — Identify which variant to flash

The v3 PCB accepts either a C3 or C6 XIAO module — pick the config that matches the chip you installed.

| Chip installed | Config file |
|----------------|-------------|
| ESP32-C6 | `sensorhost-v3-c6.yaml` |
| ESP32-C3 | `sensorhost-v3-c3.yaml` |

> The v2 PCB only accepts a C3. Use `sensorhost-v3-c3.yaml` for it as well.

---

## Step 3 — Connect the board

1. Plug the board into your PC via USB-C.
2. Windows 11 should recognize it automatically — no driver install needed.
3. Open Device Manager and confirm a new **COM port** appears under "Ports (COM & LPT)". Note the port number (e.g., COM4).

**If no COM port appears:** The board may need to be put into bootloader mode. Unplug, hold the **BOOT** button on the board, plug in while holding BOOT, then release. Try again.

---

## Step 4 — Flash

**From the CLI** (run from the directory containing the YAML files):

```bash
esphome run sensorhost-v3-c6.yaml
```

ESPHome will:
1. Pull `sensorhost-v3-base.yaml` via the `packages:` reference
2. Compile the firmware (takes 2–5 minutes on first build; subsequent builds are faster)
3. Auto-detect the COM port and flash
4. Reboot the board

If ESPHome can't find the port automatically, specify it manually:

```bash
esphome run sensorhost-v3-c6.yaml --device COM4
```

**From the Home Assistant ESPHome add-on:**
Upload the YAML files to your ESPHome config folder, open the device in the dashboard, and click **Install → Plug into this computer**.

---

## Step 5 — Verify the flash

Watch the serial output after flashing. You should see:

- Boot messages from the ESP32
- `[W] [wifi:xxx] WiFi connection failed — starting AP mode`
- A line confirming the AP is broadcasting: `AP SSID: SensorHost Setup`
- The NeoPixel LED on the board should pulse or light up

At this point the board is ready to ship. You can disconnect USB-C.

**Optional quick test on your bench:**
Connect your phone to the "SensorHost Setup" WiFi, enter your own home WiFi credentials, and confirm the board connects. Then factory reset via the ESPHome dashboard (or power-cycle and it will clear saved credentials after re-flash) before shipping.

---

## Step 6 — Prepare to ship

Checklist before the board leaves your hands:

- [ ] Board tested and booting cleanly
- [ ] Recipient confirmed to have **Home Assistant** running
- [ ] Recipient confirmed to have the **ESPHome add-on** installed (Settings → Add-ons in HA — not just the ESPHome integration)
- [ ] AP password communicated to recipient (written on the box, in a note, or texted)
- [ ] Printed or digital copy of the recipient setup guide included

---

## Releasing firmware updates

When you want to push an update to all boards in the field:

1. Make your changes in `sensorhost-v3-base.yaml` (for sensor/config changes) or in the relevant variant YAML (for chip-specific changes).
2. Bump the `version:` field under `project:` in `sensorhost-v3-base.yaml` — without a bump, dashboards never offer the update:
   ```yaml
   project:
     name: "FrankenWerx.sensorhost"
     version: "1.1.0"   # ← increment this
   ```
3. Commit and push to `main` on GitHub.

Each recipient's ESPHome dashboard will show an **Update available** indicator. They click it, it pulls the new config from GitHub, recompiles, and flashes over WiFi. You don't need access to their HA instance.

**GitHub packages are cached for a day.** A dashboard that built within the last 24 hours reuses its copy of the repo and logs `Skipping update for https://github.com/FrankenWerx/SensorHost.git@main, will refresh on the next run after ...`, so the install silently ships the old config. To pull a change immediately, replace the one-line `github://` package in the device YAML with the long form and install again:

```yaml
packages:
  FrankenWerx.sensorhost:
    url: https://github.com/FrankenWerx/SensorHost
    ref: main
    files: [sensorhost-v3-c6.yaml]   # or sensorhost-v3-c3.yaml
    refresh: always
```

### Flashing a board in place (OTA from the CLI)

A board that is already on WiFi can be reflashed from your PC without unplugging it — point ESPHome at its IP:

```bash
esphome run sensorhost-v3-c6.yaml --device 192.168.x.x
```

Flashing the repo config this way removes any API encryption key the dashboard added, so Home Assistant must re-add or reconfigure the device afterward. The board keeps its saved WiFi.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| No COM port in Device Manager | Try bootloader mode: hold BOOT, plug in, release BOOT |
| `esphome: command not found` | Run `pip install esphome` or use the HA add-on |
| Compile error about `packages:` | Ensure you have a recent ESPHome version (`esphome version` should be 2023.x or later) |
| Board not appearing in HA after provisioning | Confirm ESPHome **add-on** is installed, not just the integration |
| AP not broadcasting after flash | Check serial output; board may have picked up a cached WiFi credential — re-flash to clear |
| Dashboard shows the board **Offline** while Home Assistant still gets data | The dashboard finds boards by mDNS, which does not cross subnets/VLANs (or escape a metal enclosure). Add `use_address: <board IP>` under `wifi:` in the device YAML, or flash from the CLI with `--device <board IP>` |
| Some sensors stop updating while the LD2410 keeps going | The board's polling timers have stalled (seen on a C6 built with ESPHome 2026.6.2). Press **Reboot**, then rebuild with a current ESPHome |
