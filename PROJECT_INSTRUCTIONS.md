# Project instructions: RaspberryPiPicoBase

High-level reference for every Claude session on this repository. Keep it
current at the end of each session.

## Purpose

A base KiCad 10 design of a custom RP2354A board (bare chip with 2 MB
in-package flash, not a Pico module) containing everything required for a
stable implementation: crystal, regulators, decoupling, USB, BOOTSEL/RUN
buttons, SWD debug and GPIO headers. The owner decided (2026-11-01) to use
only RP235x parts with built-in flash, so the external QSPI flash circuit was
removed from schematic and board. The owner forks this repository to start new
projects, so the design must stay minimal, clean and self-contained.

## Origin and licence

Derived from the official Raspberry Pi "RP2350A QFN-60 Minimal Design Example"
(RPI-RP2350A-MINIMAL, revision R4-S1, July 2026, MIT). The original schematic
PDF and BOM are kept in docs/reference/. Attribution lives in NOTICE.md and
in the title blocks. The routed 2-layer board was kept as published.

## Repository structure

- RaspberryPiPicoBase.kicad_pro / .kicad_sch / .kicad_pcb at the repo root.
  KiCad files use CRLF line endings; preserve them when editing as text.
- lib/RaspberryPiPicoBase.kicad_sym and lib/RaspberryPiPicoBase.pretty/ hold
  every symbol and footprint (extracted from the original embedded copies).
  Footprints carry their STEP models as embedded files; the board also keeps
  the same models embedded. Project-local sym-lib-table and fp-lib-table
  register them under the nickname "RaspberryPiPicoBase". Nothing depends on
  the user's global KiCad libraries.
- Symbol names are the Raspberry Pi parts-database keys (for example
  "Capacitors | MLCC | M1005/RP-01BJ9Y-KRGF41"); they were kept for
  traceability to the reference BOM.
- .mcp.json: Claude Code MCP config for kicad-mcp-pro 4.5.0 (uvx, profile
  "full", mode "write", project dir ".", PYTHONUTF8=1 for Windows consoles).
- docs/reference/: original schematic PDF and BOM (markdown).
- .gitignore excludes KiCad backups, autosaves, .kicad_prl and generated
  fabrication outputs.

## Design state

- ERC (severity-all): 0 violations. DRC (severity-all, schematic parity):
  0 violations apart from 4 excluded lib_footprint_mismatch warnings on the
  mounting holes (MH1-MH4). Those are a KiCad false positive: the footprint
  contains a full-circle track arc that KiCad compares differently between
  board and library even after re-importing from the library. The exclusions
  are stored in the .kicad_pro (board.design_settings.drc_exclusions);
  kicad-cli still lists them in JSON output with "excluded": true.
- Changes versus the Raspberry Pi original: title blocks; project renamed;
  libraries made project-local; LDO tab pin (U2 pin 4) made passive to clear a
  power-output conflict; PWR_FLAG added on +1V1 and VREG_AVDD; GND symbol
  #PWR23 moved onto the 1.27 mm grid (stub wire removed); U1 changed to
  RP2354A (value, description, manufacturer_pn; same footprint); external
  flash circuit removed (U3, C2, R1, R9, R10, their wires, labels, tracks and
  the net declarations), no-connect flags on U1 pins 55-59 (QSPI data/clock),
  QSPI_SS kept wired to the BOOTSEL circuit (R6, SW1); PCB silkscreen title
  changed to "Raspberry Pi Pico Base / RP2354A rev 0.1". The U1 pads 55-59
  carry KiCad's "unconnected-(U1-...-PadNN)" nets so schematic parity passes.
- The flash symbols/footprints are still available in lib/ if a fork needs
  external QSPI flash again (see docs/reference for the original circuit).
- Still as in the original: Micro-USB B connector (not USB-C), Raspberry Pi
  BOM directive symbols (VARIANT, SCH_BOM_REV, PCB_PWB_REV) on the sheet, and
  Datasheet fields pointing to Raspberry Pi internal paths. Candidates for
  cleanup if the owner wants.

## Tooling and environment facts

- KiCad 10.0.8 at C:\Program Files\KiCad\10.0 (kicad-cli, bundled Python 3.11
  with pcbnew SWIG). uv, Python 3.14 and Node are available.
- Command-line checks: kicad-cli sch erc --severity-all, kicad-cli pcb drc
  --severity-all --schematic-parity (JSON reports via --format json).
- KiCad 10 IPC API covers the PCB editor only (kicad-python 0.8.0 has 41 board
  commands, 0 schematic commands). Schematic edits by any MCP server are file
  edits; the open Schematic Editor does not detect them and saving in KiCad
  overwrites them. Workflow: schematic closed while editing, or File > Revert
  afterwards. PCB edits are live over IPC once "Enable KiCad API" is on in
  KiCad preferences (it was off in the owner's config at setup time; the
  owner must enable it and restart KiCad).
- kicad-mcp-pro hides its live PCB editing tools until an IPC connection with
  an open board exists; in write mode without IPC it exposes 290 tools
  (schematic authoring, ERC/DRC, exports, library tools).

## Working agreements with the owner

- Ask before changing behaviour or features; answer questions first.
- Remind the owner to push to GitHub before large changes.
- Never commit; at session end update this file, README.md and (if needed)
  HANDOFF.md, then propose a commit message.
- The owner wants to keep KiCad open while the agent edits: honour the
  schematic workflow above and prefer live IPC for PCB work.
