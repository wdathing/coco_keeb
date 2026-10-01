# CoCo Pico Keyboard Interface

A small interposer PCB that turns a real **TRS-80 Color Computer 3 keyboard** into a
**USB keyboard**, using a Raspberry Pi Pico (RP2040). The CoCo keyboard's ribbon
connector plugs into the board, the Pico scans the original 7×8 key matrix, and the
result enumerates on any PC (or CoCo emulator) as a standard USB HID keyboard.

The board also has a DE9 port for an Atari-style digital joystick, so a classic stick
can be used alongside the keyboard.

Designed in KiCad. Built, flashed, and fully tested.

## Firmware

There are two firmware loads for this board. Both use the **same pin mapping**, so
either one can be flashed onto the same hardware without changes.

| | **QMK** (recommended) | **KMK** |
|---|---|---|
| Language / runtime | C, compiled to a `.uf2` | Python on CircuitPython |
| Repository | [wdathing/qmk_firmware → `keyboards/coco`](https://github.com/wdathing/qmk_firmware/tree/master/keyboards/coco) | [wdathing/kmk_firmware_coco → `code.py`](https://github.com/wdathing/kmk_firmware_coco/blob/main/code.py) |
| CoCo symbol mapping | Yes: shifted keys send the CoCo's characters | No: plain PC symbols |
| CoCo / PC layout switch | Yes, saved across power cycles | No |
| DE9 joystick | Yes | No |
| Status | Main, full-featured load | Earlier, minimal bring-up load |

### 1. QMK: `keyboards/coco`

This is the main firmware. It goes beyond a plain matrix-to-USB conversion so that the
keyboard behaves like a CoCo keyboard:

- **CoCo shifted symbols.** On the CoCo, `Shift+2` is `"`, `Shift+7` is `'`,
  `Shift+8` is `(`, `Shift+:` is `*`, `Shift+;` is `+`, and so on. The firmware rewrites
  those keys so the host gets the character printed on the CoCo keycap, not the PC
  one.
- **Two layouts, switchable at runtime:**
  - `Ctrl` + `Alt` + `0`: **CoCo layout** (CoCo symbol mapping, the default)
  - `Ctrl` + `Alt` + `1`: **PC layout** (plain PC symbols)

  The selected layout is stored in EEPROM and survives unplugging.
- **Special keys.** `Clear` is mapped to `Home` and `Break` to `Esc`. `Alt`, `Ctrl`,
  `F1`, and `F2` pass straight through.
- **DE9 joystick.** An Atari-style digital stick on J2 sends arrow keys, and its fire
  button sends `Space`:

  | DE9 pin | Function | Pico GPIO | Key sent |
  |---|---|---|---|
  | 1 | Up | GP27 | ↑ |
  | 2 | Down | GP20 | ↓ |
  | 3 | Left | GP21 | ← |
  | 4 | Right | GP22 | → |
  | 6 | Fire | GP26 | Space |
  | 8 | Ground | — | — |

- NKRO, bootmagic, extra keys, and mouse keys are enabled.

**Build and flash**, from a QMK checkout of the fork above:

```bash
make coco:default          # builds coco_default.uf2
make coco:default:flash    # or hold BOOTSEL on the Pico, plug it in, and copy the .uf2 over
```

You can also enter the bootloader by holding the top-left matrix key (`@`) while
plugging in the keyboard.

### 2. KMK: `code.py`

A CircuitPython / [KMK](https://github.com/KMKfw/kmk_firmware) version. It was the
first firmware brought up on this hardware and is useful for quick experiments: edit
`code.py` on the Pico's USB drive and the change takes effect immediately, with no
build step.

It maps the matrix directly to PC keys, with no CoCo symbol rewriting, no layout
switch, and no joystick support.

**Install:**

1. Flash [CircuitPython](https://circuitpython.org/board/raspberry_pi_pico/) onto the Pico.
2. Copy the `kmk/` folder and `code.py` from the repository above onto the `CIRCUITPY` drive.

## Key matrix

Both firmwares scan the matrix in the same order as the CoCo's own PIA keyboard scan
(columns strobed through `$FF02`, rows read through `$FF00`). The diode direction is
COL2ROW.

- **Columns** (8): GP8 – GP15
- **Rows** (7): GP0 – GP6

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
> three or more keys can produce ghost keypresses. This is a limitation of the
> original keyboard, not of the firmware.

## Hardware

| Ref | Part | Notes |
|---|---|---|
| U1 | Raspberry Pi Pico | SMD / through-hole footprint |
| J1, J3 | 1×16 pin headers | Pico headers / CoCo keyboard connector |
| J2 | DE9 male, right angle | Atari-style digital joystick |
| J6 | 2×5 header | Remote DE9 (for a panel-mounted joystick connector) |
| J4 | 1×4 header | Optional SSD1306 I²C OLED display |
| J5 | 1×2 header | Power in |
| D1, D2 + R1, R2 | 3 mm LEDs, 1 kΩ | Status LEDs |
| SW1 | Right-angle tactile switch | "Pairing" button |

The OLED header, status LEDs, and pairing button are provided on the board but are
**not used by either firmware yet**.

## Repository contents

| Path | What it is |
|---|---|
| `Coco_pico_keyb.kicad_sch` / `.kicad_pcb` / `.kicad_pro` | KiCad schematic, layout, and project |
| `Coco_pico_keyb.step` | 3D model of the assembled board |
| `Coco_pico_keyb.dsn` / `.ses` / `.rules` | Freerouting autorouter exchange files |
| `production/` | JLCPCB fabrication outputs (Gerbers zip, BOM, CPL/positions, IPC netlist) |
| `Coco_pico_keyb-backups/` | KiCad project backups |

The `production/` files were generated with the
[JLC Plugin for KiCad](https://github.com/bennymeg/JLC-Plugin-for-KiCad) (Fabrication
Toolkit). Upload `production/Coco_pico_keyb.zip` to JLCPCB to order boards.

## Related project

[**xroar-waveshare-rp2350-pizero**](https://github.com/wdathing/xroar-waveshare-rp2350-pizero):
a CoCo emulator for the Waveshare RP2350-PiZero that takes USB keyboard input. Together
with this board, it lets a real CoCo 3 keyboard drive the emulator.
