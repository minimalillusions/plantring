# MakerWorld listing: PlantRing

This is the text for the PlantRing listing on MakerWorld. Each section below matches a field on MakerWorld or a
part of the description.

**Before you publish:**

- Search this file for `TODO` and replace every placeholder with your own details: print settings, printed parts,
  photos and links.
- Copy from the rendered view of this file on GitHub, not from the raw Markdown. That way headings, lists and bold
  text paste as formatting.
- The firmware facts here (names, defaults, links) match the [README](../README.md). If you change one, change the
  other too.

---

## Model name

PlantRing – smart grow light ring for houseplants (ESP32-C3, WS2812B)

Alternatives:

- PlantRing – 24-LED grow light with day/night timer, no app needed
- PlantRing – small plant light with its own web page and Home Assistant support

## Short description

A small grow light for a houseplant: a 3D-printed holder for a 24-LED WS2812B ring and an ESP32-C3 SuperMini.
The free firmware installs from your browser. The lamp has its own web page and a day/night timer, and Home
Assistant is optional.

## Description

**[TODO: optional – one or two sentences in your own words, for example why you made it and which plant it lights
at your place. Delete this line if you don't want an intro.]**

### What it is

PlantRing turns a 24-LED WS2812B ring and an ESP32-C3 SuperMini into a small smart grow light. It works on its
own: it has its own web page for switching and dimming, three color presets, and a built-in timer that switches
the light on in the morning and off in the evening. If you use Home Assistant, it finds the lamp automatically,
but you don't need it.

It's a small light. It works best for one plant, herbs or seedlings, placed fairly close above the leaves.

### Features

- **Own web page:** on/off, brightness, color, presets and timer. No app, no account, no cloud.
- **Timer (photoperiod):** on at 07:00 and off at 21:00 by default (14 hours of light). You can change both times,
  and schedules past midnight work too.
- **Presets:** Vegetative (blue-rich), Bloom (red-rich) and Full spectrum (white).
- **Safe power limit:** runs from a normal USB charger.
- **Home Assistant:** discovered automatically (optional).
- **Free, open-source firmware** (ESPHome), installed from the browser with one click. Updates come through Home
  Assistant or the installer.

### What you need

| Qty | Part | Notes |
|---:|---|---|
| 1 | ESP32-C3 SuperMini | The small board with USB-C. 2.4 GHz WiFi only. |
| 1 | WS2812B LED ring, 24 LEDs | 5 V type. **[TODO: ring size that fits the holder, e.g. outer diameter in mm]** |
| 1 | USB-C data cable | Must carry data, not only charge. You need it for the first install. |
| 1 | USB charger, 5 V, at least 1 A | 2 A is recommended. |
| 3 | Short wires | **[TODO: wire length that fits the holder]** |
| 1 | 330 Ω resistor (optional) | Goes in the data wire. |
| **[TODO]** | **[TODO: screws, inserts, glue, ... – or delete this row]** | |

Printed parts: **[TODO: list the printed parts, e.g. ring holder, arm or stand, board cover]**

Tools: a soldering iron and solder, plus heat-shrink tubing or tape to insulate the joints.

If MakerWorld offers a separate field for the bill of materials, you can copy this table there as well.

### Printing

**[TODO: replace these with your own print settings]**

- Printer and profile: **[TODO]**
- Material: **[TODO, e.g. PETG or PLA]**
- Layer height: **[TODO]**
- Walls and infill: **[TODO]**
- Supports: **[TODO: none, or where they are needed]**
- Orientation: **[TODO]**
- Print time and filament: **[TODO]**
- Notes: **[TODO: anything special, e.g. tolerances for the ring or the board]**

About heat: the LEDs and the board can get warm while the light is on. If you ever run the ring at 100% power (only
with a separate 5 V supply, see Safety), print the holder in PETG or ASA rather than PLA. PLA starts to soften at
around 60 °C.

### Assembly and wiring

**[TODO: photo of all parts laid out before assembly]**

1. Print the parts.
2. Solder three wires to the ring's pads: **5V**, **GND** and **DIN** (data in, not DOUT).
   **[TODO: wire length that fits the holder]**
3. Optional: solder the 330 Ω resistor into the data wire, close to the ring.
4. Solder the other ends to the board: 5V to **5V**, GND to **GND** (often printed as G), DIN to **GPIO4** (often
   printed as just 4).
5. Install the firmware and test the ring (next section) **before** you close anything. For the first install you
   may need the board's BOOT button.
6. **[TODO: how the ring goes into the holder]**
7. **[TODO: how the board sits in the holder and where the USB cable goes]**

| Board pin | Ring pad | Notes |
|---|---|---|
| 5V | 5V (sometimes VCC) | Power for the LEDs, passed through the board from USB |
| GND (G) | GND | Common ground |
| GPIO4 (4) | DIN (sometimes DI or IN) | Data; optional 330 Ω resistor in series |
| (nothing) | DOUT | Leave unconnected |

- Double-check 5V and GND before you plug in. Swapping them can destroy the LEDs.
- Don't use GPIO2, GPIO8 or GPIO9 for the ring (strapping pins; GPIO8 is the onboard LED).
- A full wiring diagram is in the README: https://github.com/minimalillusions/plantring#wiring

**[TODO: close-up photo of the wiring before the holder is closed]**

### Install the firmware

Installer: **https://minimalillusions.github.io/plantring/**

1. Open this page in **Chrome or Edge** on a computer.
2. Plug in the ESP32-C3 with a USB **data** cable.
3. Click **Install**, pick the port, and wait a minute or two.
4. Enter your WiFi name and password when asked (2.4 GHz).
5. Done. Open the link shown (the lamp's own web page). Home Assistant finds it automatically.

Three things trip people up:

- **Browser:** Chrome or Edge on a computer. Firefox 151 or newer also works. Safari and phones/tablets can't
  install firmware this way.
- **Data cable:** many USB cables can only charge. With one of those, no port shows up. This is the most common
  problem by far.
- **2.4 GHz WiFi:** the ESP32-C3 can't use 5 GHz networks. Most routers send 2.4 GHz and 5 GHz under the same name,
  which is fine.

If something goes wrong, see the troubleshooting section:
https://github.com/minimalillusions/plantring#troubleshooting

### Using it

- **Web page:** open `http://plantring-xxxxxx.local` or the lamp's IP address. `xxxxxx` stands for six characters
  from the board's MAC address. The installer shows the exact name and the link.
- **Set your time zone first:** the **Timezone** setting starts at `Europe/Berlin`.
- **Timer:** with **Schedule** on, the light switches on at **Lights on** (default 07:00) and off at
  **Lights off** (default 21:00).
- **Manual changes win until the next scheduled time.** If you switch the light off in the afternoon, it stays off
  until the next **Lights on** time.
- The lamp needs internet access once after it starts, to learn the time. Until then the timer waits, but the light
  and the web page work normally.
- **Presets:** **Preset: Vegetative** (blue-rich, for leafy growth and seedlings), **Preset: Bloom** (red-rich, for
  flowering plants) and **Preset: Full spectrum** (white, looks natural in a living room). Presets only change the
  color; the brightness stays as you set it.
- **Why not always purple?** Leaves absorb red and blue light most readily, and red + blue needs less power. But
  plants use green light too, and purple looks odd in a living room. White is a perfectly good choice.
- **Home Assistant:** the lamp appears under **Settings → Devices & services → Discovered**. Click **Add**.
- **Customizing** (LED count, data pin, power limit): take control of the lamp in ESPHome Device Builder. See
  https://github.com/minimalillusions/plantring#customizing

### Safety

- **Power limit:** the ring is powered through the little ESP32 board. Full white on all 24 LEDs would draw about
  1.44 A, which is too much for the board for hours every day. So the firmware limits the LED power to 80%.
  Because of the brightness curve, that means only about 0.77 A at full white (about 0.5 A with red + blue), while
  it still looks like roughly 80% brightness to your eye.
- **Only raise the limit to 100%** if the ring gets its 5 V directly from a power supply, not through the board.
  The README explains how: https://github.com/minimalillusions/plantring#power-and-safety
- Use a decent USB charger with at least 1 A (2 A recommended).
- After the first hour, feel the board, cable and ring. Warm is fine, hot is not.
- Keep water away from the electronics. Don't mist the lamp, and place it so watering can't drip onto it.
- Don't look straight into the ring at full brightness.
- This is a DIY project without any certification.

### Links

- Firmware, documentation and source code: https://github.com/minimalillusions/plantring
- Web installer: https://minimalillusions.github.io/plantring/
- Firmware license: MIT. 3D model license: **[TODO: the license you choose on MakerWorld]**

## Tags

grow light, plant light, houseplants, LED ring, WS2812B, ESP32-C3, ESPHome, Home Assistant, smart home, DIY

## Photos to add

- **[TODO: cover photo – the lamp switched on above a plant]**
- **[TODO: the lamp in Full spectrum (white) and in a colored preset, side by side]**
- **[TODO: all parts laid out before assembly]**
- **[TODO: close-up of the wiring before the holder is closed]**
- **[TODO: screenshot of the lamp's web page]**
