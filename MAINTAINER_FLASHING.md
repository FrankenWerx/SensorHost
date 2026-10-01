# SensorHost — Maintainer Flashing Guide

Step-by-step for building and shipping a new board.

---

## Prerequisites

- ESPHome installed — CLI (`pip install esphome`) or via the Home Assistant add-on
- A data-capable USB-C cable (charge-only cables will not work)
- The repo files on your machine (clone or local copy)

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
2. Bump the `version:` field in the variant YAML(s) you changed:
   ```yaml
   project:
     name: "FrankenWerx.sensorhost"
     version: "1.1.0"   # ← increment this
   ```
3. Commit and push to `main` on GitHub.

Each recipient's ESPHome dashboard will show an **Update available** indicator. They click it, it pulls the new config from GitHub, recompiles, and flashes over WiFi. You don't need access to their HA instance.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| No COM port in Device Manager | Try bootloader mode: hold BOOT, plug in, release BOOT |
| `esphome: command not found` | Run `pip install esphome` or use the HA add-on |
| Compile error about `packages:` | Ensure you have a recent ESPHome version (`esphome version` should be 2023.x or later) |
| Board not appearing in HA after provisioning | Confirm ESPHome **add-on** is installed, not just the integration |
| AP not broadcasting after flash | Check serial output; board may have picked up a cached WiFi credential — re-flash to clear |
