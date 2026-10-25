# PlantRing

A small smart grow light for houseplants: a 24-LED WS2812B ring, an ESP32-C3 SuperMini and free
[ESPHome](https://esphome.io) firmware. It works on its own, with its own web page and a built-in
day/night schedule. Home Assistant is optional; if you use it, it finds the lamp automatically.

![PlantRing grow light above a houseplant](docs/images/plantring.jpg)

*Photos will be added soon.*

<!-- TODO(author): save the photo as docs/images/plantring.jpg (it shows up above automatically), then delete the "Photos will be added soon" line. -->

The 3D-printable holder is published on MakerWorld (link will be added here).
<!-- TODO(author): add the MakerWorld link to the line above. -->

**Install the firmware from your browser:** <https://minimalillusions.github.io/plantring/>

## Features

- **Works standalone.** The lamp has its own web page. No app, no account, no cloud.
- **Built-in schedule (photoperiod).** Lights on at 07:00 and off at 21:00 by default (14 hours of light).
  You can change both times, and schedules that run past midnight work too.
- **Three presets:** Vegetative (blue-rich), Bloom (red-rich) and Full spectrum (white). You can also pick any
  color and brightness, or use a few effects (Rainbow, Color Wipe, Twinkle, Pulse).
- **Safe by default.** A built-in power limit keeps the current low enough to run the whole ring from a normal USB
  charger through the little ESP32 board (see [Power and safety](#power-and-safety)).
- **Home Assistant (optional).** Discovered automatically via ESPHome; every control shows up as an entity.
- **Easy setup.** One-click installer in the browser. WiFi is set up right there, or through the lamp's own
  hotspot, or via Bluetooth.
- **Updates.** New firmware versions can be installed from Home Assistant or with the installer.
- **Open source** (MIT) and easy to customize with ESPHome.

## What you need

| Qty | Part | Notes |
|---:|---|---|
| 1 | ESP32-C3 SuperMini | The small board with USB-C and a blue onboard LED. 2.4 GHz WiFi only. |
| 1 | WS2812B LED ring, 24 LEDs | 5 V type, with pads for 5V, GND, DIN (data in) and DOUT. |
| 1 | USB-C **data** cable | Must carry data, not only charge. You need it for the first install. |
| 1 | USB charger, 5 V, at least 1 A | 2 A is recommended. A decent phone charger is fine. |
| 3 | Short wires | For 5V, GND and data. |
| 1 | 330 Ω resistor (optional, recommended) | Goes in the data wire and makes the signal more robust. |
| 1 | 3D-printed holder | From MakerWorld (see above). |

Tools: a soldering iron and solder, plus heat-shrink tubing or tape to insulate the joints.

## Wiring

There are only three wires. The ring gets its power **through the board**, straight from USB.

| ESP32-C3 SuperMini pin | LED ring pad | What it is for |
|---|---|---|
| `5V` | `5V` (sometimes `VCC` or `+5V`) | Power for the LEDs (USB 5 V, passed through the board) |
| `GND` (often printed as `G`) | `GND` | Common ground |
| `GPIO4` (often printed as just `4`) | `DIN` (sometimes `DI` or `IN`) | LED data. Optional 330 Ω resistor in this wire, close to the ring |
| (nothing) | `DOUT` (sometimes `DO` or `OUT`) | Leave unconnected (only used to chain more LEDs) |

```text
  USB charger, 5 V (min. 1 A, 2 A recommended)
          |
          | USB-C cable
          v
  +---------------------+                      +------------------------+
  | ESP32-C3 SuperMini  |                      | WS2812B ring, 24 LEDs  |
  |                     |                      |                        |
  |              5V  o--+----------------------+--o 5V   (VCC / +5V)    |
  |             GND  o--+----------------------+--o GND                 |
  |           GPIO4  o--+-----[ 330 ohm ]------+--o DIN  (DI / IN)      |
  |                     |      (optional)      |    DOUT: not connected |
  +---------------------+                      +------------------------+
```

Good to know:

- Use the **DIN** pad (data in), not DOUT. Many rings have small arrows that show the data direction.
- Double-check 5V and GND before you plug in. Swapping them can destroy the LEDs.
- Don't use GPIO2, GPIO8 or GPIO9 for the ring. These are "strapping" pins that the ESP32-C3 reads at power-up.
  GPIO8 also drives the onboard blue LED, which PlantRing uses as its status light.
- Pin labels differ between SuperMini batches, and look-alike boards (for example "SuperMini Plus" versions) use a
  different layout. Check the labels on your own board before soldering.
- Install the firmware and test the ring **before** you close the holder. For the first install you may need the
  board's BOOT button.

## Power and safety

### Why the power is limited to 80%

- Each WS2812B LED draws up to about **60 mA** at full white. 24 LEDs × 60 mA ≈ **1.44 A**.
- In PlantRing all of that current would flow through the ESP32-C3 SuperMini: USB socket → board → `5V` pin → ring.
  That tiny board (and many thin USB cables) isn't made to carry about 1.5 A for 14 hours a day. Too much current
  causes heat and voltage drops, random restarts, and at worst a damaged board or cable.
- So the firmware limits each color channel to **80%**. This is the `max_power: 80%` setting.

**Why 80% saves almost half the current:** the firmware uses a gamma curve (ESPHome's default of 2.8), so the
brightness slider feels even to your eye. The 80% limit is applied *before* that curve, so the LEDs end up with
0.8^2.8 ≈ **54%** of their full current. To your eye, the brightest setting still looks like roughly 80% brightness.
Your plant (and the power supply) get a little over half.

| At full brightness | With the default limit (80%) | Without limit (100%) |
|---|---:|---:|
| Full spectrum (white) | about 0.77 A | about 1.44 A |
| Red + blue only (maximum) | about 0.51 A | about 0.96 A |
| Preset: Vegetative | about 0.39 A | about 0.72 A |
| Preset: Bloom | about 0.29 A | about 0.54 A |

These are worst-case estimates for the LEDs only; the ESP32 itself needs a little more on top. That's why the
charger should deliver at least 1 A, and 2 A gives you comfortable headroom.

### When 100% is OK

Only raise `max_power` to `100%` if the ring gets its 5 V **directly from a power supply**, not through the board:

```text
  5 V power supply (2 A or more)
    +5V ---+-------------------------------> ring 5V
           +---> board 5V
    GND ---+-------------------------------> ring GND
           +---> board GND
  board GPIO4 -----[ 330 ohm ]-------------> ring DIN
```

- Use short, reasonably thick wires for +5V and GND (for example 22 AWG / 0.34 mm² or thicker).
- The board and the ring must share GND, otherwise the data signal doesn't work.
- If the board is powered through its `5V` pin like this, unplug that supply before you connect the board to a
  computer with USB.
- Optional: a 500–1000 µF capacitor (6.3 V or more) across 5V and GND at the ring smooths the supply.
- The web installer always installs the safe 80% version. To change the limit, you build your own firmware; see
  [Customizing](#customizing).

### General safety

- Use a decent USB charger. Very cheap ones often can't really deliver what's printed on them.
- After the first hour, feel the board, cable and ring. Warm is fine, hot is not.
- PLA starts to soften at around 60 °C. If you raise the power limit, check the holder after an hour of full white,
  or print it in PETG or ASA.
- Keep water away from the electronics. Don't mist the lamp, and place it so watering can't drip onto it.
- Don't look straight into the ring at full brightness.
- This is a DIY project without any certification. Don't leave a modified (100%) build running unattended until
  you have checked that nothing gets hot.

## Install the firmware

Open the installer: **<https://minimalillusions.github.io/plantring/>**

1. Open this page in **Chrome or Edge** on a computer.
2. Plug in the ESP32-C3 with a USB **data** cable.
3. Click **Install**, pick the port, and wait a minute or two.
4. Enter your WiFi name and password when asked (2.4 GHz).
5. Done. Open the link shown (the lamp's own web page). Home Assistant finds it automatically.

Three things trip people up:

- **Browser:** use Chrome or Edge on a Windows, macOS or Linux computer. Firefox 151 or newer also works (it asks
  for an extra permission first). Safari, iPhones, iPads and Android phones can't install firmware this way.
- **Data cable:** many USB cables can only charge. With one of those, no port shows up. This is by far the most
  common problem.
- **2.4 GHz WiFi:** the ESP32-C3 can't use 5 GHz networks. Most routers send 2.4 GHz and 5 GHz under the same name,
  which is fine.

<details>
<summary>What you'll see in the installer, step by step</summary>

1. Click **Install** on the page. The browser lists the available ports. The board usually shows up as something
   like "USB JTAG/serial debug unit" or as a COM / usbmodem / ttyACM port. Select it and click **Connect**.
2. In the dialog, choose **Install** and confirm. A new installation erases the board first; that's normal.
3. Wait while it says "Installing" and keep the page visible (it's faster that way).
4. After "Installation complete!" click **Next**. Under "Configure Wi-Fi", pick your network, enter the password and
   click **Connect**.
5. When it says "Device connected to the network!", click **Visit Device** to open the lamp's web page (bookmark it).
   If you use Home Assistant, you can also click **Add to Home Assistant**, or just wait until it's discovered.

</details>

## First setup

1. Open the lamp's web page: use **Visit Device** in the installer, or go to `http://plantring-xxxxxx.local`.
2. Set **Timezone** to where you live. The default is `Europe/Berlin`.
3. Check **Lights on** (default 07:00) and **Lights off** (default 21:00). **Schedule** is on by default.
4. Pick a preset (see below) and set the brightness on **Light**.
5. Mount the lamp above your plant. Light gets weaker quickly with distance, so keep it fairly close to the leaves.

**About the name:** every lamp gets its own name, `plantring-` followed by six characters from the board's MAC
address, for example `plantring-a1b2c3`. That way, several lamps on one network don't get in each other's way.
You'll find the exact name in the installer, in your router's device list, or in Home Assistant. In this README,
`plantring-xxxxxx` stands for your lamp's name.

### Other ways to set up WiFi

If you skipped the WiFi step, or the lamp moves to a new network:

- **Installer again:** plug the lamp into your computer, open the installer page, click **Install** and choose
  **Connect to Wi-Fi** (or **Change Wi-Fi**).
- **The lamp's own hotspot:** if the lamp has no WiFi details yet, or can't reach your WiFi for a minute or two, it
  opens its own WiFi hotspot named like the lamp (for example `plantring-a1b2c3`). Connect your phone or laptop to
  it. A setup page should open by itself; if not, go to <http://192.168.4.1>. Pick your WiFi, enter the password
  and save.
- **Bluetooth (Improv):** if Home Assistant has Bluetooth, a lamp without WiFi shows up under **Discovered** as
  **Improv via BLE**, and Home Assistant can send it your WiFi details. The Bluetooth setup on
  [improv-wifi.com](https://www.improv-wifi.com/) also works in Chrome or Edge (not on iPhone/iPad).

## Using it

### The web page

Open `http://plantring-xxxxxx.local` or the lamp's IP address in any browser on your home network. The page is
stored on the lamp itself, so it works even without internet.

The page has three sections:

| Section | Control | What it does |
|---|---|---|
| Light | **Light** | On/off, brightness, color and effects |
| Light | **Preset: Vegetative** | Blue-rich light; brightness stays as it is |
| Light | **Preset: Bloom** | Red-rich light; brightness stays as it is |
| Light | **Preset: Full spectrum** | White light; brightness stays as it is |
| Schedule | **Schedule** | On: the lamp follows **Lights on** / **Lights off** |
| Schedule | **Lights on** | When the light switches on (default 07:00) |
| Schedule | **Lights off** | When the light switches off (default 21:00) |
| Schedule | **Photoperiod** | Shows the resulting day length, for example "14 h light / 10 h dark" |
| Schedule | **Timezone** | The lamp's time zone (default `Europe/Berlin`) |
| Device | **WiFi signal** | Signal strength in dBm |
| Device | **IP address** | Handy when the `.local` address doesn't work |
| Device | **Firmware** | Shows whether newer firmware is available (see [Updating](#updating)) |
| Device | **Restart** | Restarts the lamp |
| Device | **Factory reset** | Erases the WiFi details and all your settings, then restarts. Set up WiFi again afterwards (see above). |

Further down, the page has an **OTA Update** section for uploading a firmware file by hand.

The small blue LED on the board shows the status: it blinks while the lamp has no WiFi connection (or another
problem) and stays off when everything is fine.

Some phones and networks can't open `.local` names. Use the IP address instead. It's shown by the installer's
**Visit Device** link and in your router's device list. Tip: give the lamp a fixed IP in your router (a "DHCP
reservation") so your bookmark keeps working.

**Security note:** the page has no password, so anyone on your home network can control the lamp and even upload
new firmware to it. That's normal for small home devices, but never make the lamp reachable from the internet (no
port forwarding). If you want a password, see [Customizing](#customizing).

### Presets

| Preset | Color | Good for |
|---|---|---|
| **Preset: Vegetative** | Blue-rich (purple-blue) | Leafy growth, seedlings, herbs |
| **Preset: Bloom** | Red-rich (pink-red) | Flowering and fruiting plants |
| **Preset: Full spectrum** | White | Everything; looks natural in a living room |

The presets only change the color (and stop any running effect). The brightness stays where you set it. If the
light is off, pressing a preset switches it on.

**The trade-off:** leaves absorb red and blue light most readily, which is why classic grow lights glow purple.
Plants use green light too, just a little less efficiently. Red and blue also need less power than white (see the
table in [Power and safety](#power-and-safety)). But purple light looks odd in a living room, and it makes plants
look strange. If the lamp sits where you live, **Full spectrum** is a perfectly good choice.

### Schedule and time zone

- With **Schedule** on, the lamp switches the light on at **Lights on** and off at **Lights off**, every day.
  The default 07:00 to 21:00 gives 14 hours of light and 10 hours of darkness.
- Schedules past midnight work too: for example **Lights on** 18:00 and **Lights off** 06:00. If both times are the
  same, the schedule keeps the light off.
- New **Lights on** and **Lights off** times count right away: if they change whether the light should be on right
  now, the lamp switches immediately. **Photoperiod** shows the resulting day length.
- Most plants need a daily dark period too, so don't run the light around the clock.
- **Manual changes win until the next scheduled time.** The lamp only acts at the **Lights on** and **Lights off**
  times. If you switch the light off at 15:00 to take a photo, it stays off. The next scheduled event is "off" at
  21:00 (nothing changes), and then the light comes back on at 07:00 the next morning. Switch it back on by hand
  if you want it sooner.
- **After a power cut**, the light stays off until the lamp knows the time again. This usually takes a few seconds
  after it reconnects to WiFi. Then it switches to whatever the schedule says for the current time. Color and
  brightness are remembered.
- **The lamp needs internet access to learn the time** after it starts. It gets the time from public time servers
  (NTP). Until then the schedule waits, but the light and the web page work normally. If the internet goes down
  later, the lamp keeps its own clock running and the schedule carries on.
- **The time zone is the lamp's own setting.** It doesn't take the time zone from your computer or from Home
  Assistant. The list covers common time zones around the world, plus UTC. If yours is missing, pick one with the
  same local time and the same daylight-saving rules.
- Turn **Schedule** off if you'd rather switch the light only by hand or with Home Assistant automations. When you
  turn it back on, the light follows the schedule right away.

### Home Assistant

1. In Home Assistant, go to **Settings → Devices & services**.
2. Under **Discovered**, the lamp appears as an ESPHome device. Click **Add** and confirm.
3. All controls from the web page show up as entities of the device "PlantRing xxxxxx". **Lights on**,
   **Lights off** and **Timezone** are listed under *Configuration* on the device page. Use them in dashboards and
   automations.

The schedule runs on the lamp itself, so it keeps working even when Home Assistant is offline. If you'd rather
control the light from Home Assistant automations, switch the lamp's **Schedule** off.

## Customizing

The web installer always installs the standard build. To change settings such as the power limit, the LED count or
the data pin, take control of the lamp in **ESPHome Device Builder** and build your own firmware from the same
configuration.

1. Get the ESPHome Device Builder, either as a Home Assistant app (formerly called add-on) or as the desktop app
   from [desktop.esphome.io](https://desktop.esphome.io).
2. With the lamp online in the same network, it appears under **Discovered**. Click **Take control** (older
   versions call it **Adopt**). The Device Builder creates a small configuration file that loads `plantring.yaml`
   from this repository.
3. Edit that file and add your changes under `substitutions:`. Your values override the defaults.
4. Click **Install** and choose to install wirelessly.

The generated file looks roughly like this (your name and key will differ; the lines under "Your changes" are
examples you add yourself):

```yaml
substitutions:
  name: plantring-a1b2c3
  friendly_name: PlantRing a1b2c3
  # Your changes:
  num_leds: "16"
  led_pin: GPIO5
  # max_power: 100%   # ONLY if the ring has its own 5 V supply!

packages:
  minimalillusions.plantring: github://minimalillusions/plantring/plantring.yaml@main

esphome:
  name: ${name}
  name_add_mac_suffix: false
  friendly_name: ${friendly_name}

api:
  encryption:
    key: "..."

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

| Substitution | Default | What it does |
|---|---|---|
| `max_power` | `80%` | Power limit per color channel. Only use `100%` if the ring gets 5 V directly from a power supply (see [When 100% is OK](#when-100-is-ok)). |
| `led_pin` | `GPIO4` | Data pin for the ring. Free pins on the ESP32-C3 SuperMini include GPIO3, GPIO5, GPIO6, GPIO7 and GPIO10. Avoid GPIO2, GPIO8 and GPIO9. |
| `num_leds` | `"24"` | Number of LEDs on your ring. Keep the quotes. |
| `wifi_output_power` | `8.5dB` | WiFi transmit power. Many SuperMini boards connect *more* reliably at this low setting. If yours works well and the signal is weak, try higher values (up to `20dB`). |
| `name` / `friendly_name` | `plantring` / `PlantRing` | Filled in for you when you take control. Change `friendly_name` to rename the lamp. |

Good to know:

- Your own builds need ESPHome 2026.8.0 or newer.
- The Device Builder needs your WiFi name and password in its **Secrets** (`wifi_ssid` and `wifi_password`).
- Missing a time zone? The comments at `Timezone` and `apply_timezone` in `plantring.yaml` explain how to add one.
- **Enable encryption** (an option when you take control) protects the connection to Home Assistant. If the Device
  Builder runs as a Home Assistant app, Home Assistant picks up the new key by itself. Otherwise, copy the key from
  the file when Home Assistant asks for it.
- `@main` always loads the newest `plantring.yaml`, so every new build includes the latest changes. To stay on a
  fixed version, replace `@main` with a release tag from the
  [Releases](https://github.com/minimalillusions/plantring/releases) page.
- Want a password for the web page? Add this to your file, and add `web_password` with a password of your choice to
  the Device Builder's **Secrets**:

  ```yaml
  web_server:
    auth:
      username: admin
      password: !secret web_password
      type: digest
  ```

## Updating

Updates keep your WiFi details and settings. The lamp checks for new firmware every 6 hours. When there is one, the
**Firmware** entry on its web page says so. To install it:

- **Home Assistant:** the lamp's **Firmware** update shows up with your other updates. Click **Update**. The lamp
  downloads the new firmware itself (it needs internet access for that).
- **Installer:** plug the lamp into your computer, open <https://minimalillusions.github.io/plantring/>, click
  **Install** and choose **Update**. The installer always has the newest version.
- **Upload by hand (advanced):** download the newest `plantring-esp32c3.ota.bin` from
  <https://minimalillusions.github.io/plantring/firmware/plantring-esp32c3.ota.bin> (or a specific version from the
  [Releases](https://github.com/minimalillusions/plantring/releases) page) and upload it in the **OTA Update** section
  of the lamp's web page. Don't use the `.factory.bin` file for this.

**If you took control in ESPHome Device Builder,** update from the Device Builder (**Install**). Your own build has
no **Firmware** entry, because updates now come from your builds. Installing the standard firmware, for example from
the installer page, would replace your customized build and your changes would be gone.

## Troubleshooting

### The installer says my browser isn't supported, or the button is missing

Use Chrome or Edge on a computer (or Firefox 151 or newer). Safari and phones/tablets can't install firmware.

### No port shows up in the list

- It's almost always the cable. Try another cable that you know carries data, for example one you've used to copy
  files to a phone.
- Plug the board directly into the computer, not into a hub or monitor.
- The SuperMini uses the ESP32-C3's built-in USB and normally needs no extra driver. Other ESP32-C3 boards with a
  separate USB chip (CH340, CP2102) need that chip's driver.
- On Linux, your user needs access to serial ports (usually the `dialout` group).
- If the board's LED doesn't light up at all with a USB-C-to-USB-C cable, try a USB-A-to-USB-C cable. Some boards
  don't get power from USB-C-only chargers and ports.

### "Failed to initialize", or the first install fails

Unplug the board. Hold the **BOOT** button while you plug it back in, then release BOOT and click **Install** again.
(If your board has a RESET or RST button, you can also hold BOOT, press and release RESET, then release BOOT.)
After the install, unplug and replug the board once so the new firmware starts.

### The WiFi step didn't show up after installing

Unplug and replug the board, open the installer page again, click **Install** and choose **Connect to Wi-Fi**.
Or use the lamp's hotspot (see [Other ways to set up WiFi](#other-ways-to-set-up-wifi)).

### The lamp won't join my WiFi

- It must be a 2.4 GHz network. Networks that use 5 GHz only don't work.
- Check the password. It's case-sensitive.
- Move the lamp closer to the router for a test. On the web page, a WiFi signal around -70 dBm or better is fine.
  Much below -80 dBm, the connection gets unreliable.
- The blue LED on the board blinks while the lamp has no WiFi connection. If you see a WiFi network named like the
  lamp (`plantring-xxxxxx`), the lamp can't reach your WiFi and has opened its hotspot. Join it and enter your WiFi
  details again (see [Other ways to set up WiFi](#other-ways-to-set-up-wifi)).

### I can't open `plantring-xxxxxx.local`

Use the IP address instead (from the installer, your router or Home Assistant). Not all phones and networks
support `.local` names.

### The light switches at the wrong time, or not at all

- Check **Timezone**. A wrong time zone shifts everything by hours.
- Check that **Schedule** is on and that **Lights on** and **Lights off** are set correctly.
- After a restart, the lamp needs internet access to get the time. Until it has the time, the schedule doesn't run.
- Remember that manual changes stay until the next scheduled time (see
  [Schedule and time zone](#schedule-and-time-zone)).

### The LEDs flicker, show wrong colors, or only the first LED works

- Check that the data wire goes to **DIN**, not DOUT, and that GND is connected.
- Check the solder joints, and keep the data wire short. The 330 Ω resistor helps.
- If you changed `num_leds`, make sure it matches your ring.
- The ESP32 sends a 3.3 V data signal to 5 V LEDs. That works with most rings. If yours stays flaky, a level
  shifter (for example a 74AHCT125) on the data line fixes it.

### The board restarts when the light is bright

The power source or cable can't deliver enough current. A computer's USB port often isn't enough for the ring. Use a
USB charger (2 A recommended) and a short, good cable, and keep `max_power` at `80%`.

### I want to start over

Press the factory reset button on the web page. This erases the WiFi details and all settings, and the lamp then
waits for a new WiFi setup. Or reinstall with the installer: for a new installation it erases the board first.

## Repository layout

| Path | What it is |
|---|---|
| `plantring.yaml` | The core ESPHome configuration: light, presets, schedule, web page and the fallback hotspot. The ESPHome Device Builder loads this when you take control. |
| `plantring.factory.yaml` | What the web installer installs: `plantring.yaml` plus WiFi setup over USB and Bluetooth (Improv), the **Firmware** update check and the link back to this repository for the Device Builder. |
| `.github/workflows/` | GitHub Actions that build the firmware and publish the installer page to GitHub Pages. Release binaries are attached to [GitHub Releases](https://github.com/minimalillusions/plantring/releases). |
| `static/index.html` | The web installer page. It's published on GitHub Pages together with the firmware. |
| `docs/` | The text for the MakerWorld listing ([docs/makerworld.md](docs/makerworld.md)). Photos will go in `docs/images/`. |

Built with [ESPHome](https://esphome.io) and [ESP Web Tools](https://esphome.github.io/esp-web-tools/).

## Publishing a new version

For maintainers of this repository:

1. Make your changes and raise `project_version` in `plantring.yaml` (for example to `"1.1.0"`). Lamps only offer an
   update when this number changes.
2. Push to `main`. GitHub Actions builds the firmware and updates the installer page, and lamps then offer the update
   as **Firmware**.
3. Optional: publish a GitHub release tagged `v1.1.0` (the same number with a `v`). The firmware files are attached to
   the release automatically, and the release notes show up with the update in Home Assistant.

One-time setup: **Settings → Pages → Build and deployment → Source: GitHub Actions**.

## License

[MIT](LICENSE). This covers the firmware configuration, the installer and the documentation in this repository.
The 3D model on MakerWorld is published there under the license shown on its MakerWorld page.
