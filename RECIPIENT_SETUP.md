# SensorHost — Setup Guide

Welcome! This sensor monitors room presence, temperature, humidity, and air pressure, and sends all of that data to your Home Assistant automatically.

Setup takes about 5 minutes and you won't need to touch it again after that.

---

## Before you start

You'll need:

- The SensorHost board
- A USB-C power adapter or cable (any phone charger works)
- Your home WiFi name and password
- Home Assistant running on your network
- The **ESPHome add-on** installed in Home Assistant

**Check that ESPHome is installed:**
In Home Assistant, go to **Settings → Add-ons** and look for **ESPHome** in your installed add-ons list. If it's not there, click **Add-on Store**, search for ESPHome, install it, and start it before continuing.

---

## Step 1 — Power on the board

Plug the board into any USB-C power source. The LED will light up. Give it about 30 seconds to start up.

---

## Step 2 — Connect the board to your WiFi

Choose the method that matches your phone or computer:

### Option A — Android phone with Chrome (easiest)

1. Open **Google Chrome** on your Android phone.
2. Go to **[improv-wifi.com](https://www.improv-wifi.com)**.
3. Click **Connect device to WiFi**.
4. Select **SensorHost** from the Bluetooth device list.
5. Enter your home WiFi name and password when prompted.
6. Done — the board will connect automatically.

### Option B — Any phone or computer

1. Open your WiFi settings.
2. Connect to the network named **"SensorHost Setup"**.
   - Password: `sensorhost` *(unless a different one is written on the box or was sent to you)*
3. A setup page should open automatically. If it doesn't, open a browser and go to **192.168.4.1**.
4. Enter your home WiFi name and password and click Save.
5. The board will disconnect from the setup network and connect to your home WiFi. Reconnect your phone to your normal WiFi.

---

## Step 3 — Add it to Home Assistant

1. Open **Home Assistant**.
2. Go to **Settings → Add-ons → ESPHome** and click **Open Web UI**.
3. You should see a **"Discovered"** device called **SensorHost** with an **Adopt** button.

   > If it doesn't appear within 2 minutes, try refreshing the page.

4. Click **Adopt**. ESPHome will download the configuration from GitHub and set up the device. This takes a minute or two.
5. Once adopted, the device will appear in your ESPHome dashboard with a green **Online** status.

---

## Step 4 — Done

Your sensors are now live in Home Assistant. To add them to a dashboard:

1. Go to **Settings → Devices & Services → ESPHome**.
2. Find your SensorHost device and click it.
3. Click **Add to Dashboard** on any sensor you want to display.

The board will receive automatic firmware updates when new versions are available — you'll see an **Update** button appear in the ESPHome dashboard when one is ready. Just click it and it handles the rest.

---

## What the sensors show

| Sensor | What it measures |
|--------|-----------------|
| Presence Detected | Whether someone is in the room (yes/no) |
| Moving Target | Active movement detected |
| Still Target | Someone sitting still detected |
| Temperature | Room temperature (°F) |
| Humidity | Relative humidity (%) |
| Barometric Pressure | Air pressure (hPa) |
| Light Sensor | Ambient light level |
| Distance Detection | How far away the detected person is |

---

## Troubleshooting

**"SensorHost Setup" WiFi doesn't appear**
Wait 60 seconds after powering on and try again. If it still doesn't appear, unplug the board for 10 seconds and plug it back in.

**Setup page doesn't open automatically**
Open a browser manually and go to `192.168.4.1`.

**Board doesn't appear in ESPHome after connecting to WiFi**
Make sure your phone/computer is back on your home WiFi (not still on "SensorHost Setup"). Refresh the ESPHome dashboard. If it still doesn't show, confirm the ESPHome add-on (not just the ESPHome integration) is installed and running.

**"Adopt" button never appears**
The board may have connected to your WiFi but ESPHome isn't running. Go to Settings → Add-ons → ESPHome and make sure it shows as **Running**.
