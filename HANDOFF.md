# Handoff

## Left to do

1. **Reload and review in the KiCad GUI (owner action).** The schematic,
   board and libraries were edited on disk while KiCad had them open. Reopen
   the project (or File > Revert in both editors) before saving anything.
   Then look at: the Power block of the schematic (USB-C symbol J1, R11/R12,
   label spacing), the USB-C area on the board (connector protrudes 0.8 mm
   past the edge), SW1/R6/BOOTSEL left of the USB-C, SW2/R4 near J3, and J4
   with the title silkscreen at the bottom edge. Run Tools > Update PCB from
   Schematic once; it should report nothing to change. Check the 3D viewer
   (J4 shows only if "show DNP" is enabled).

2. **Before ordering at JLCPCB**, re-check stock on jlcpcb.com for the
   Extended parts (RP2354A C41378174 had 88 in stock on 2026-11-01, the
   inductor C42411119 is low) and decide whether J2/J3 headers are assembled
   or replaced; the owner intends to swap them. Confirm the USB-C and switch
   rotations in bom/cpl.csv against JLCPCB's preview.

3. **First boards**: verify crystal start-up (the Basic YXC crystal has 80 Ω
   max ESR versus 50 Ω for the Raspberry Pi qualified part) and the AMS1117
   stability with the 10 uF ceramic output cap.

4. **Decide what to do with the freed board area** (left of U1, x 84-100 mm,
   y 84-92 mm): shrink the outline or leave room for the user's circuitry.

## Candidate follow-ups (not decided with the owner yet)

- Remove the Raspberry Pi BOM-directive symbols (VARIANT, SCH_BOM_REV,
  PCB_PWB_REV; already excluded from the BOM) and the Raspberry Pi logo on
  the board silkscreen.
- Clear the remaining Datasheet fields that point to Raspberry Pi internal
  paths (file:///C:\rpi-pdb\...) for parts that were not swapped (U1, L1,
  J2/J3 symbol, mounting holes), keeping schematic cache, library and
  footprint fields identical.
- Optionally rename the database-style symbol names to short KiCad-style
  names (same rename in the schematic lib_symbols cache, lib_id references
  and the library file).
- Remove the unused library entries (Amphenol Micro-USB, Wurth switch, JST SH
  footprints and symbols, flash SOIC) if forks will not need them.
- Reduce permission prompts: allow uvx and the kicad-cli path in
  .claude/settings.json (or run /fewer-permission-prompts).
- Delete the untracked `output/` (MCP render scratch) and `.history/`
  folders; both are now in .gitignore.
