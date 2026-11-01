# Project instructions: RaspberryPiPicoBase

High-level reference for every Claude session on this repository. Keep it
current at the end of each session.

## Purpose

A base KiCad 10 design of a custom RP2354A board (bare chip with 2 MB
in-package flash, not a Pico module) containing everything required for a
stable implementation: crystal, regulators, decoupling, USB-C, BOOTSEL/RUN
buttons, SWD debug and GPIO headers. The owner decided (2026-11-01) to use
only RP235x parts with built-in flash, so the external QSPI flash circuit was
removed. The board is meant to be assembled by JLCPCB, so the BOM is
organised around their parts library (Basic parts wherever a drop-in exists).
The owner forks this repository to start new projects, so the design must
stay minimal, clean and self-contained.

## Origin and licence

Derived from the official Raspberry Pi "RP2350A QFN-60 Minimal Design Example"
(RPI-RP2350A-MINIMAL, revision R4-S1, July 2026, MIT). The original schematic
PDF and BOM are kept in docs/reference/. Attribution lives in NOTICE.md
(Raspberry Pi design, KiCad library footprints/models, EasyEDA/LCSC 3D models)
and in the title blocks.

## Repository structure

- RaspberryPiPicoBase.kicad_pro / .kicad_sch / .kicad_pcb at the repo root.
  KiCad files use CRLF line endings; preserve them when editing as text.
- lib/RaspberryPiPicoBase.kicad_sym and lib/RaspberryPiPicoBase.pretty/ hold
  every symbol and footprint. Footprints carry their STEP models as embedded
  files (kicad-embed://); the board keeps the same models embedded at root
  level. Project-local sym-lib-table and fp-lib-table register them under the
  nickname "RaspberryPiPicoBase". Nothing depends on global KiCad libraries.
- Symbol names are the Raspberry Pi parts-database keys (for example
  "Capacitors | MLCC | M1005/RP-01BJ9Y-KRGF41"), kept for traceability even
  where the part inside was swapped. New symbols follow the same style
  ("Connectors | USB/HRO-TYPE-C-31-M-12", "Resistors | Chip |
  M1005/UNIROYAL-0402WGF5101TCE").
- Every BOM symbol (instance, schematic cache and library) carries the fields
  manufacturer, manufacturer_pn, lcsc_pn and a Datasheet link to the JLCPCB
  part page. Footprints on the board carry the same fields (KiCad 10 DRC
  parity compares them).
- bom/bom.csv (JLCPCB BOM, grouped, DNP excluded) and bom/cpl.csv (placement)
  are generated with kicad-cli; the commands are in docs/BOM.md.
- docs/BOM.md: sourcing table with LCSC codes, Basic/Extended status, the
  alternatives that were considered and the U1 fallback.
- docs/reference/: original schematic PDF and BOM (markdown).
- .mcp.json: Claude Code MCP config for kicad-mcp-pro 4.5.0 (uvx, profile
  "full", mode "write", PYTHONUTF8=1).
- .gitignore excludes KiCad backups, .kicad_prl, generated outputs, the MCP
  render folder (output/) and the editor local history (.history/).

## Design state (2026-11-01)

- ERC (severity-all): 0 violations. DRC (severity-all, schematic parity,
  refilled zones): 0 violations, 0 unconnected, 0 parity issues, apart from 4
  excluded lib_footprint_mismatch warnings on the mounting holes (KiCad false
  positive, exclusions stored in the .kicad_pro).
- Changes versus the Raspberry Pi original:
  - Project renamed, libraries made project-local, title blocks, LDO tab pin
    passive, PWR_FLAGs on +1V1, VREG_AVDD, VBUS and GND, GND #PWR23 on grid.
  - U1 is RP2354A (same QFN-60 footprint); external flash removed (U3, C2, R1,
    R9, R10), no-connects on U1 pins 55-59, QSPI_SS kept for BOOTSEL. Flash
    symbol/footprint still in lib/ for forks that need it.
  - USB: Micro-USB replaced by USB-C Korean Hroparts TYPE-C-31-M-12 (KiCad
    footprint USB_C_Receptacle_HRO_TYPE-C-31-M-12, 16 pin, USB 2.0) with 5.1 k
    CC pull-downs R11/R12. Placed flush at the top edge (0.8 mm protrusion),
    D+/D- rerouted as a 0.8 mm pair to R7/R8, VBUS bridged over the pad row.
  - Parts swapped to JLCPCB Basic: all MLCCs (Samsung, Fenghua), 1 k and 33 R
    resistors (Uniroyal), LDO AMS1117-3.3, crystal YXC X322512MSB4SI (20 pF,
    with C3/C4 changed to 33 pF), switches XKB TS-1187A-B-A-B (5.1 mm, new
    footprint xkb_ts-1187a-b-a-b built from the datasheet; SW1 and R6 moved
    left of the USB-C, SW2 and R4 moved near J3). Headers J2/J3 point at
    CONNFLY DS1021-2x18SF11-B but the owner intends to change them later.
  - J4 debug connector is now a JST GH SM03B-GHS-TB (footprint
    jst_sm03b-ghs-tb from the KiCad library with mounting pads numbered 4/5)
    and is marked DNP; SWD/SWCLK are also on J3. Board title silkscreen moved
    to the bottom edge.
  - Raspberry Pi BOM directive symbols (VARIANT, SCH_BOM_REV, PCB_PWB_REV) are
    excluded from the BOM but still on the sheet.
- Still from the original: Datasheet fields of unchanged parts point to
  Raspberry Pi internal paths; Raspberry Pi silkscreen logo on the board.
- 3D models: all footprints have embedded models. USB-C and switch models are
  EasyEDA/LCSC models (USB-C needs offset y = 1.1 mm), GH model from the
  KiCad library. The renderer hides DNP parts, so J4 does not show in renders.
- Part status per docs/BOM.md: Extended parts are R7/R8 (27 R, no Basic
  exists), U1, L1, J1, J2/J3. Crystal ESR (80 R max) is above the 50 R of the
  Raspberry Pi qualified ABM8-272-T3: verify oscillator start-up on the first
  boards.

## Tooling and environment facts

- KiCad 10.0.8 at C:\Program Files\KiCad\10.0 (kicad-cli, bundled Python 3.11
  with pcbnew SWIG). uv, Python 3.14 (has compression.zstd) and Node available.
- Command-line checks used after every edit: `kicad-cli sch erc
  --severity-all`, `kicad-cli pcb drc --severity-all --schematic-parity
  --refill-zones --save-board` (refills zones and rewrites the board with
  KiCad's own writer), `kicad-cli sch export netlist` to verify connectivity,
  `kicad-cli pcb render` (optionally `--side left`, `--perspective`) to check
  models and placement.
- The SWIG pcbnew bindings of 10.0.8 are unreliable for footprint/field/pad
  edits (raw SwigPyObjects, wx asserts that hang). All design edits in this
  repo were done as text edits of the KiCad files with Python, then verified
  with kicad-cli. See the Claude memory notes for format details (net names
  instead of numbers, absolute text angles, pad angles include footprint
  rotation, embedded-file checksum = MurmurHash3 "V1" variant).
- KiCad 10 IPC API: the owner has enabled it (kicad_get_server_info reports
  livePcbContext and ipcDocumentLoaded true), but the MCP live tools returned
  inconsistent results in this session, so the file-based workflow was used
  for both schematic and board. Whenever files are edited on disk while KiCad
  has them open, the owner must reload (File > Revert or reopen the project)
  before saving in KiCad.
- Part data sources: jlcpcb.com/partdetail/<Cxxxx> shows Basic/Extended as a
  label next to the image gallery; the LCSC API
  wmsc.lcsc.com/ftps/wm/product/detail?productCode=<Cxxxx> gives live LCSC
  stock, USD price ladder and smtLabel BASE/EXPAND; the unofficial mirror
  jlcsearch.tscircuit.com is a stale snapshot. JLCPCB's own assembly stock is
  only visible on their site (it differs from LCSC stock).

## Working agreements with the owner

- Ask before changing behaviour or features; answer questions first.
- Remind the owner to push to GitHub before large changes.
- Never commit; at session end update this file, README.md and (if needed)
  HANDOFF.md, then propose a commit message.
- The owner wants to keep KiCad open while the agent edits: tell them to
  reload the project before saving when files were changed on disk.
