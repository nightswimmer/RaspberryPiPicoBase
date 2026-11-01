# Handoff

## Left to do

1. **Enable the KiCad API (owner action).** In KiCad 10: Preferences > Plugins >
   "Enable KiCad API", then restart KiCad. It was off at setup time
   (%APPDATA%\kicad\10.0\kicad_common.json, api.enable_server=false). KiCad was
   running during the setup session, so the setting was not edited on disk.

2. **Verify live PCB editing through the MCP.** With the API enabled and
   RaspberryPiPicoBase.kicad_pcb open in the PCB editor (opened from the
   project manager, not a standalone pcbnew), start a Claude Code session in
   the repo and call `kicad_get_server_info`. Expect
   `kicad.livePcbContext: true` and `ipcDocumentLoaded: true`. The tool list
   should then also contain the live PCB editing tools that kicad-mcp-pro
   hides while no IPC board is connected (without IPC it exposes 290 tools in
   the `full` profile, none of which move or route footprints). If they still
   do not appear, check `uvx kicad-mcp-pro doctor` with PYTHONUTF8=1 set.

3. **First GUI open.** The project has never been opened in the KiCad GUI
   after the conversion. Open it once, confirm the schematic and board load
   with the project-local libraries (sym-lib-table / fp-lib-table in the
   repo), that the 3D viewer shows the embedded models, and that the two new
   PWR_FLAG symbols (+1V1 near L1, VREG_AVDD near R3) look acceptable. KiCad
   may ask to create global library tables on first launch of 10.0; choose the
   default tables, this project does not depend on them. Also check the area
   where the flash used to be (board x 84-100 mm, y 76-92 mm, left of U1):
   the footprints and tracks were removed by script and the zones refilled,
   so run Tools > Update PCB from Schematic once in the GUI to confirm it
   reports nothing to change, and look at the empty area for leftover
   silkscreen or odd zone shapes.

4. **Decide what to do with the freed board area** (optional): shrink the
   outline, or leave room for the user's own circuitry.

## Candidate follow-ups (not decided with the owner yet)

- Replace the Micro-USB B connector with USB-C (needs 5.1 kΩ CC pull-downs
  and a new footprint; stock KiCad library has USB_C_Receptacle_USB2.0_16P
  and GCT USB4105 footprints).
- Remove the Raspberry Pi BOM-directive symbols (VARIANT, SCH_BOM_REV,
  PCB_PWB_REV) and the Raspberry Pi silkscreen text / logo on the board.
- Clear Datasheet fields that point to Raspberry Pi internal paths
  (file:///C:\rpi-pdb\...) in lib/RaspberryPiPicoBase.kicad_sym, the
  schematic cache and the footprints (keep schematic cache and library
  identical, or ERC reports lib_symbol_mismatch).
- Optionally rename the database-style symbol names to short KiCad-style
  names; the same rename must be applied to the schematic's lib_symbols
  cache, the lib_id references and the library file.
- Reduce permission prompts: add uvx and the kicad-cli path to
  .claude/settings.json allow list (or run /fewer-permission-prompts).
