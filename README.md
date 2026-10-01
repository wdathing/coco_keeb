# CoCo Pico Keyboard Interface

A Raspberry Pi Pico board that connects to the keyboard matrix of a **TRS-80 Color
Computer 3**. Its 16-pin keyboard header (J1) is wired pin-for-pin to the CoCo 3
keyboard connector, so the same board can sit on either side of that connector.
Which firmware you flash decides which way the keystrokes go.

Designed in KiCad. Built and tested.

## Two firmware loads, two directions

| | **1. QMK `coco`** | **2. usb-to-coco** |
|---|---|---|
| Direction | Real CoCo keyboard **→** PC (USB) | Modern USB / Bluetooth keyboard **→** real CoCo |
| J1 connects to | The CoCo 3 **keyboard's** ribbon cable | The CoCo 3 **motherboard's** keyboard connector |
| Pico | Pico (RP2040) | Pico 2 W (RP2350 + Bluetooth) |
| What the Pico does | Scans the matrix and acts as a USB HID keyboard | Acts as a USB / Bluetooth host and emulates the matrix |
| Repository | [wdathing/qmk_firmware → `keyboards/coco`](https://github.com/wdathing/qmk_firmware/tree/master/keyboards/coco) | [wdathing/usb-to-coco](https://github.com/wdathing/usb-to-coco) |
| Build system | QMK (`make coco:default`) | Pico SDK 2.3.0 / CMake |

Both loads use the **same pin assignment** for the matrix (rows on GP0–GP6, columns
on GP8–GP15) and share the same key layout. They are mirror images of each other:
QMK reads the CoCo keyboard and sends modern keycodes, while usb-to-coco takes modern
keycodes and reproduces CoCo key presses.

Both also handle the places where the two keyboards disagree: they remap shifted
symbols so the character on the keycap you're pressing is the one that appears.

---

### 1. QMK `coco`: use a CoCo 3 keyboard on a PC

Plug the original CoCo 3 keyboard into J1 and the Pico's USB port into a PC, a
Raspberry Pi, or a CoCo emulator such as
[XRoar](https://github.com/wdathing/xroar-waveshare-rp2350-pizero). The board shows
up as a standard USB keyboard.

- **CoCo shifted symbols.** On the CoCo, `Shift+2` is `"`, `Shift+7` is `'`,
  `Shift+8` is `(`, `Shift+:` is `*`, `Shift+;` is `+`, and so on. The firmware sends
  whatever is printed on the CoCo keycap, not the PC character in that position.
- **Two layouts, switchable at runtime:**
  - `Ctrl` + `Alt` + `0`: **CoCo layout** (CoCo symbols, the default)
  - `Ctrl` + `Alt` + `1`: **PC layout** (plain PC symbols)

  The selected layout is saved in EEPROM and survives unplugging.
- **Windows / GUI key.** The CoCo has no Windows key, so tapping `Ctrl` + `Alt` together
  and releasing them, with no other key in between, sends one. Holding them still
  works as normal `Ctrl` and `Alt`.
- **Special keys.** `CLEAR` sends `Home` and `BREAK` sends `Esc`. `ALT`, `CTRL`, `F1`,
  and `F2` pass straight through.
- **Joystick.** An Atari-style digital joystick on the DE9 (J2) sends the arrow keys,
  and its fire button sends `Space`:

  | DE9 pin | Function | Pico GPIO | Key sent |
  |---|---|---|---|
  | 1 | Up | GP27 | ↑ |
  | 2 | Down | GP20 | ↓ |
  | 3 | Left | GP21 | ← |
  | 4 | Right | GP22 | → |
  | 6 | Fire | GP26 | Space |
  | 8 | Ground | — | — |

- NKRO, bootmagic, extra keys, and mouse keys are enabled.

**Build and flash**, from a checkout of the QMK fork:

```bash
make coco:default          # builds coco_default.uf2
make coco:default:flash    # or hold BOOTSEL, plug in, and copy the .uf2 over
```

To enter the bootloader without the BOOTSEL button, hold the top-left matrix key
(`@`) while plugging in.

---

### 2. usb-to-coco: use a modern keyboard on a real CoCo 3

Remove the CoCo's own keyboard and plug J1 into the motherboard's keyboard
connector. The Pico 2 W pretends to be the keyboard: it watches which column the
CoCo is scanning and pulls the matching rows low for whatever keys are held. The CoCo
can't tell the difference, so nothing on the CoCo side needs to change.

- **USB keyboards** plug into the Pico's own USB port, which runs in host mode. Use an
  OTG adapter, and power the board through J5.
- **Bluetooth keyboards** work too, both Classic and BLE (tested with an 8BitDo Retro
  Mechanical keyboard and an Anker compact keyboard).
  - Press the **pairing button (SW1)** to open a 60-second pairing window. The Pico
    scans for nearby keyboards and connects to the first one it finds.
  - The last keyboard is remembered and reconnects automatically on power-up.
- **Status LEDs:**
  - **D1** lights while the CoCo is actively scanning the keyboard, so you can confirm
    the CoCo is alive and connected.
  - **D2** lights while any key is held.
- **Keycap-accurate symbols.** On a modern keyboard, `Shift+2` types `@`, `Shift+;`
  types `:`, and the `'`/`"` key types quotes. Each one is translated to the CoCo key
  (with or without CoCo SHIFT) that produces that character.
- **Special keys.** `Esc` and `Pause` send BREAK, `Home` sends CLEAR, `Backspace`
  sends ← (the CoCo's erase-left), and `` ` `` sends @.

**Build and flash** with the Pico SDK (or the Raspberry Pi Pico VS Code extension):

```bash
cmake -B build -G Ninja && ninja -C build    # -> build/usb-to-coco.uf2
```

A `-DDEBUG_KEY_LOG=ON` build is available for bring-up without a CoCo. It logs
decoded keys over UART instead of driving the matrix, because GP0/GP1 are both matrix
rows and UART0. See the
[usb-to-coco README](https://github.com/wdathing/usb-to-coco#debug-build) for details.

---

## Key matrix

The scan order matches the CoCo's own PIA keyboard scan: columns are strobed through
`$FF02` (PB0–PB7), and rows are read through `$FF00` (PA0–PA6). J1 matches CN2 on the
CoCo 3 motherboard pin-for-pin, as documented in the Tandy CoCo 3 Service Manual,
Figure 5-9. Pin 3 is unused on both.

- **Rows** (7): GP0 – GP6
- **Columns** (8): GP8 – GP15

```
       col0   col1   col2   col3   col4   col5   col6   col7
row0    @      A      B      C      D      E      F      G
row1    H      I      J      K      L      M      N      O
row2    P      Q      R      S      T      U      V      W
row3    X      Y      Z      Up     Down   Left   Right  Space
row4    0      1      2      3      4      5      6      7
row5    8      9      :      ;      ,      -      .      /
row6   Enter  Clear  Break  Alt    Ctrl   F1     F2     Shift
```

> The CoCo keyboard is a **diode-less** matrix, so pressing certain combinations of
> three or more keys at once can produce ghost keypresses. This comes from the
> original keyboard, not from either firmware.

## Hardware

| Ref | Part | Pico GPIO | Used by |
|---|---|---|---|
| U1 | Raspberry Pi Pico / Pico 2 W | — | both |
| J1 | 1×16 header: CoCo keyboard matrix | GP0–6, GP8–15 | both |
| J3 | 1×16 header, in parallel with J1 | GP0–6, GP8–15 | — |
| J2 | DE9 male, right angle: joystick | GP20–22, GP26–27 | QMK |
| J6 | 2×5 header: remote DE9 (mirrors J2) | same as J2 | QMK |
| D1 + R1 | 3 mm LED, 1 kΩ: CoCo scan activity | GP19 | usb-to-coco |
| D2 + R2 | 3 mm LED, 1 kΩ: key held | GP18 | usb-to-coco |
| SW1 | Right-angle tactile switch: Bluetooth pairing | GP7 | usb-to-coco |
| J4 | 1×4 header: optional SSD1306 I²C OLED | GP16 / GP17 | not yet used |
| J5 | 1×2 header: power in (VSYS, GND) | — | usb-to-coco (USB-host mode) |

DE9 pins 1 and 6 go to the Pico's ADC-capable pins (GP27 and GP26), so J2 can also
carry a CoCo analog joystick in the future.

## Repository contents

| Path | What it is |
|---|---|
| `Coco_pico_keyb.kicad_sch` / `.kicad_pcb` / `.kicad_pro` | KiCad schematic, layout, and project |
| `Coco_pico_keyb.step` | 3D model of the assembled board |
| `Coco_pico_keyb.dsn` / `.ses` / `.rules` | Freerouting autorouter exchange files |
| `production/` | JLCPCB fabrication outputs (Gerber zip, BOM, positions, IPC netlist) |
| `Coco_pico_keyb-backups/` | KiCad project backups |

The `production/` files were generated with the
[JLC Plugin for KiCad](https://github.com/bennymeg/JLC-Plugin-for-KiCad). Upload
`production/Coco_pico_keyb.zip` to JLCPCB to order boards.

## Related project

[**xroar-waveshare-rp2350-pizero**](https://github.com/wdathing/xroar-waveshare-rp2350-pizero)
is a CoCo emulator for the Waveshare RP2350-PiZero that takes USB keyboard input.
Paired with the QMK load, it lets an original CoCo 3 keyboard drive the emulator.
