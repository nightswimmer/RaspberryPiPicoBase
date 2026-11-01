# Raspberry Pi Pico Base

A KiCad 10 base design for a custom **RP2354A** board (the RP2350 variant with
2 MB of flash inside the package): everything the chip needs to boot and run
reliably, and nothing else. Fork it, add your own circuitry to the free GPIO,
and go.

The design is derived from the official Raspberry Pi
*RP2350A QFN-60 Minimal Design Example* (revision R4-S1, July 2026, MIT
licence, see [NOTICE.md](NOTICE.md)). The external QSPI flash section was
removed because the RP2354A has flash on board; the rest of the schematic and
the routed two-layer board were kept. The project was made self-contained
(project-local symbol and footprint libraries with embedded 3D models) and
cleaned up so that ERC and DRC pass with zero violations.

## What is on the board

| Block | Parts |
|---|---|
| MCU | RP2354A (QFN-60, 2 MB internal flash), 12 MHz crystal (ABM8-272-T3) with 15 pF load caps, 1 kΩ series resistor on XOUT. QSPI data/clock pins unused; QSPI_SS kept for BOOTSEL |
| Power | NCP1117 3.3 V LDO from USB VBUS, on-chip switching regulator with 3.3 µH inductor for the 1.1 V core, 100 nF decoupling on every supply pin, 33 Ω / 4.7 µF filter on VREG_AVDD |
| USB | Micro-USB B connector, 27 Ω series resistors on D+/D- |
| Buttons | BOOTSEL and RUN tactile switches |
| Debug | 3-pin JST-SH SWD connector (Raspberry Pi Debug Probe pinout) |
| I/O | Two 2x18 2.54 mm pin headers breaking out all 30 GPIO, 3V3, 1V1 and GND |
| Mechanical | 4 x 2.75 mm mounting holes |

The reference BOM with manufacturer part numbers is in
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
kicad-cli pcb drc --severity-all --schematic-parity RaspberryPiPicoBase.kicad_pcb
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
   This lets the server edit the board live inside the open PCB editor.

How edits reach KiCad, and what that means for your workflow:

- **PCB:** with the board open in the PCB editor, edits go through KiCad's IPC
  API and appear immediately, one undo step per operation. You can keep the
  editor open.
- **Schematic:** KiCad 10 has no schematic IPC API (it arrives with KiCad 11),
  so every tool edits the `.kicad_sch` file on disk. The open Schematic Editor
  does not notice this. Either close the schematic while Claude edits it, or
  use *File > Revert* afterwards, and never save in KiCad before reverting or
  you will overwrite the edit.

## Licence

The project files in this repository are MIT licensed (see [LICENSE](LICENSE)).
The underlying design is copyright Raspberry Pi Ltd, MIT licensed (see
[NOTICE.md](NOTICE.md)).
