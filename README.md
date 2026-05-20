# IIDX Display
# WIP

DIY 9-digit 16-segment display for beatmania IIDX and general PC use.

Reads the ticker from [spice2x](https://github.com/spice2x/spice2x.github.io) via its
TCP/JSON API (`ticker_get`) and displays it on 9x 16-segment digits.

## Hardware

- **MCU:** Waveshare RP2040 Zero
- **Drivers:** 2x HOLTEK HT16K33 (I2C LED matrix driver)
- **Displays:** 9x Kingbright PSC08-11SURKWA (0.8", red, common cathode, 16-segment)
- **Decimals:** 9x individual GPIO-driven decimal points

### Pinout

| Func | RP2040 GPIO |
|------|-------------|
| I2C SDA | GPIO0 |
| I2C SCL | GPIO1 |
| DP1-DP9 | GPIO2-GPIO10 |

### I2C Layout

Both HT16K33 share the same I2C bus. U1 drives displays 1-5 (COM0-COM4) and all 16
segment lines. U2 drives displays 6-9 (COM1-COM4) with ROW0-2 paralleled to U1 for
segments A/B/C (extra current drive).

U2 address is set via a diode+resistor network on the AD/COM0 pin.

### PCB
### Untested


KiCad design files are in `pcb/`. Footprint library: `pcb/iidx-library.pretty/`.

## Architecture

```
┌──────────┐    TCP/JSON     ┌──────────────┐   USB Serial   ┌──────────────────┐
│  spice2x  │ ─────────────→ │ pc-bridge/   │ ─────────────→ │ RP2040 Zero      │
│  (-api)   │  ticker_get()  │ bridge.py    │  9 raw bytes   │   2× HT16K33     │
└──────────┘                 └──────────────┘                │   9× 16seg DPs   │
                                                              └──────────────────┘
```

The game ticker is 9 characters (`IIDXIO_LED_TICKER[10]` in spice2x's
`games/iidx/iidx.h`). The PC bridge polls `ticker_get()` via the TCP API and forwards
the string over USB serial to the Pico.

### Project Structure

```
├── pcb/                  KiCad design files
├── firmware/             Pico SDK C/C++ source
│   └── src/
│       ├── main.c        init, I2C, USB serial loop
│       ├── ht16k33.c     HT16K33 driver
│       ├── display.c     display buffer, font mapping
│       └── font_16seg.h  ASCII → 16-segment lookup
└── pc-bridge/
    └── bridge.py         spice2x API client → serial
```

## Firmware
## TODO

C/C++ using the Raspberry Pi Pico SDK. Source in `firmware/`.

### Building

```bash
cd firmware
mkdir build && cd build
cmake ..
make
```

## PC Bridge

Python script that connects to spice2x's TCP API, polls `ticker_get()`, and forwards
the 9-character string over USB serial.

### spice2x Setup

Enable the API: `-api 1337 -apipass yourpass`

API request format (null-terminated JSON):

```json
{"id":1,"module":"iidx","function":"ticker_get","params":[]}
```

Response includes the 9-char ticker string in the `data` field.

## License

MIT
