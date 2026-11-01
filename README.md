# Raspberry Pi Pico Base

A KiCad 10 base design for a custom **RP2354A** board (the RP2350 variant with
2 MB of flash inside the package): everything the chip needs to boot and run
reliably, and nothing else. Fork it, add your own circuitry to the free GPIO,
and order it assembled at JLCPCB.

The design is derived from the official Raspberry Pi
*RP2350A QFN-60 Minimal Design Example* (revision R4-S1, July 2026, MIT
licence, see [NOTICE.md](NOTICE.md)). The external QSPI flash section was
removed because the RP2354A has flash on board, the Micro-USB connector was
replaced by USB-C, and the bill of materials was reworked around JLCPCB's
parts library. The project is self-contained (project-local symbol and
footprint libraries with embedded 3D models) and ERC and DRC pass with zero
violations.

## What is on the board

| Block | Parts |
|---|---|
| MCU | RP2354A (QFN-60, 2 MB internal flash), 12 MHz crystal (YXC X322512MSB4SI, 20 pF) with 33 pF load caps, 1 kΩ series resistor on XOUT. QSPI data/clock pins unused; QSPI_SS kept for BOOTSEL |
| Power | AMS1117-3.3 LDO from USB VBUS, on-chip switching regulator with 3.3 µH inductor for the 1.1 V core, 100 nF decoupling on every supply pin, 33 Ω / 4.7 µF filter on VREG_AVDD |
| USB | USB-C receptacle (Korean Hroparts TYPE-C-31-M-12, USB 2.0), 5.1 kΩ CC pull-downs, 27 Ω series resistors on D+/D- |
| Buttons | BOOTSEL and RUN tactile switches (XKB TS-1187A-B-A-B, 5.1 mm) |
| Debug | 3-pin JST GH SWD connector (Raspberry Pi Debug Probe pinout), not populated by default; SWD/SWCLK are also on the J3 header |
| I/O | Two 2x18 2.54 mm pin headers breaking out all 30 GPIO, 3V3, 1V1 and GND |
| Mechanical | 4 x 2.75 mm mounting holes |

## Bill of materials for JLCPCB

Every part carries its LCSC code (`lcsc_pn` field) and the design uses JLCPCB
*Basic* parts wherever a drop-in exists; the remaining *Extended* parts are the
RP2354A, the inductor, the USB-C connector, the 27 Ω USB resistors and the pin
headers. [docs/BOM.md](docs/BOM.md) has the full sourcing table with
Basic/Extended status, the alternatives that were considered and how to
regenerate the order files. Ready-made files:

- [bom/bom.csv](bom/bom.csv): grouped BOM with designators, quantities and LCSC part numbers
- [bom/cpl.csv](bom/cpl.csv): component placement (DNP parts excluded)

The original Raspberry Pi BOM is in
[docs/reference/RPI-RP2350A-MINIMAL_R4-S1_BOM.md](docs/reference/RPI-RP2350A-MINIMAL_R4-S1_BOM.md)
and the original schematic PDF is next to it.

## Repository layout

```
RaspberryPiPicoBase.kicad_pro   KiCad 10 project
RaspberryPiPicoBase.kicad_sch   schematic (single sheet)
RaspberryPiPicoBase.kicad_pcb   routed 2-layer board, 3D models embedded
sym-lib-table / fp-lib-table    project-local library tables
lib/RaspberryPiPicoBase.kicad_sym    all symbols used by the design
lib/RaspberryPiPicoBase.pretty/      all footprints, each with its 3D model embedded
bom/                            JLCPCB BOM and placement files (generated)
docs/BOM.md                     sourcing table, part status and alternatives
docs/reference/                 original Raspberry Pi schematic PDF and BOM
.mcp.json                       KiCad MCP server for Claude Code (kicad-mcp-pro)
NOTICE.md                       third-party attribution
```

## Requirements

- KiCad 10.0.x (tested with 10.0.8 on Windows 11). No global library setup is
  needed: every symbol, footprint and 3D model lives in this repository.

## Checking the design from the command line

```
kicad-cli sch erc --severity-all RaspberryPiPicoBase.kicad_sch
kicad-cli pcb drc --severity-all --schematic-parity --refill-zones RaspberryPiPicoBase.kicad_pcb
```

Both report zero violations on the base design. The only entries are four
pre-excluded `lib_footprint_mismatch` warnings on the mounting holes, a KiCad
false positive caused by the full-circle arc inside that footprint.

## AI-assisted editing with Claude Code (MCP)

The repository ships a project-local `.mcp.json` that starts
[kicad-mcp-pro](https://github.com/oaslananka/kicad-mcp-pro) 4.5.0 through
`uvx`, in the `full` profile with write mode. Claude Code picks it up
automatically when the repository is the working directory.

Prerequisites:

1. [uv](https://docs.astral.sh/uv/) on the PATH (`uvx` downloads the server on first use).
2. KiCad 10 installed in the default location (kicad-cli is auto-detected).
3. In KiCad: *Preferences > Plugins > Enable KiCad API*, then restart KiCad.

How edits reach KiCad, and what that means for your workflow: tools edit the
`.kicad_sch`, `.kicad_pcb` and library files on disk and verify them with
`kicad-cli`. KiCad does not notice edits made on disk while a file is open, so
after an editing session use *File > Revert* (or reopen the project) before
saving anything in KiCad, otherwise the on-disk changes are overwritten.

## Licence

The project files in this repository are MIT licensed (see [LICENSE](LICENSE)).
The underlying design is copyright Raspberry Pi Ltd, MIT licensed; the
copied KiCad library footprints and 3D model are CC-BY-SA 4.0 with the KiCad
exception; the EasyEDA/LCSC 3D models are included for visualisation (see
[NOTICE.md](NOTICE.md)).
