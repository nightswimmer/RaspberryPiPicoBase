# Bill of materials (JLCPCB)

Sourcing table for the RaspberryPiPicoBase board, prepared for JLCPCB assembly.
Every BOM symbol in the schematic carries the field `lcsc_pn`; regenerate the
order files with:

```
kicad-cli sch export bom --exclude-dnp ^
  --fields "Reference,${QUANTITY},Value,Footprint,manufacturer,manufacturer_pn,lcsc_pn,Description" ^
  --labels "Designator,Quantity,Value,Footprint,Manufacturer,Manufacturer Part,LCSC Part #,Description" ^
  --group-by "Value,manufacturer_pn,Footprint" --output bom/bom.csv RaspberryPiPicoBase.kicad_sch
kicad-cli pcb export pos --format csv --units mm --side both --output bom/cpl.csv RaspberryPiPicoBase.kicad_pcb
```

Library type (Basic / Extended) as shown on jlcpcb.com on 2026-11-01. Basic
parts are pre-loaded on the JLCPCB lines and carry no per-part setup fee;
every Extended part adds a one-off fee per order, so the design uses Basic parts
wherever a drop-in part exists.

| Designators | Qty | Value / part | Package | Manufacturer part | LCSC | JLCPCB type | Note |
|---|---|---|---|---|---|---|---|
| C1, C5, C19 | 3 | 10 uF 25 V X5R | 0805 | Samsung CL21A106KAYNNNE | C15850 | Basic | was Murata GRM21BR61C106KE15L (C77075, Extended) |
| C3, C4 | 2 | 33 pF 50 V C0G (crystal load) | 0402 | FH 0402CG330J500NT | C1562 | Basic | was 15 pF Murata GRM1555C1H150JA01D (C76950, Extended); value follows the 20 pF crystal |
| C6, C7, C9 | 3 | 4.7 uF 10 V X5R | 0402 | Samsung CL05A475MP5NRNC | C23733 | Basic | was Murata GRM155R60J475ME47D (C82453, Extended) |
| C8, C10-C18 | 10 | 100 nF 16 V X7R | 0402 | Samsung CL05B104KO5NNNC | C1525 | Basic | was Murata GRM155R71E104KE14D (C77014, Extended) |
| R2, R4, R6 | 3 | 1 kΩ 1 % | 0402 | Uniroyal 0402WGF1001TCE | C11702 | Basic | was Yageo RC0402FR-071KL (C106235) |
| R3 | 1 | 33 Ω 1 % | 0402 | Uniroyal 0402WGF330JTCE | C25105 | Basic | was Yageo RC0402FR-0733RL (C138002) |
| R7, R8 | 2 | 27 Ω 1 % (USB series) | 0402 | Uniroyal 0402WGF270JTCE | C25100 | Extended | no Basic 27 Ω exists in 0402; was Yageo RC0402FR-0727RL (C138021) |
| R11, R12 | 2 | 5.1 kΩ 1 % (USB-C CC pull-downs) | 0402 | Uniroyal 0402WGF5101TCE | C25905 | Basic | new |
| U1 | 1 | RP2354A | QFN-60 | Raspberry Pi RP2354A | C41378174 | Extended | |
| U2 | 1 | 3.3 V LDO 1 A | SOT-223 | AMS1117-3.3 | C6186 | Basic | was onsemi NCP1117ST33T3G (C26537, Extended); pin-compatible, Vin max 15 V, 1.1 V dropout |
| X1 | 1 | 12 MHz crystal, 20 pF, 80 Ω max, ±10 ppm | 3225 | YXC X322512MSB4SI | C9002 | Basic | was Abracon ABM8-272-T3 (C20625731, Extended, out of stock); ESR above the 50 Ω of the qualified part, verify oscillator start-up on the first boards |
| L1 | 1 | 3.3 uH, 2.1 A, 115 mΩ | 2016 (0806) | Abracon AOTA-B201610S3R3-101-T | C42411119 | Extended | kept; see alternatives |
| J1 | 1 | USB-C receptacle, USB 2.0, 16 pin | SMD + TH shell | Korean Hroparts TYPE-C-31-M-12 | C165948 | Extended | replaces Amphenol Micro-USB 10103594-0001LF (C428495); no USB-C is Basic |
| J2, J3 | 2 | Pin header 2x18, 2.54 mm | THT | CONNFLY DS1021-2x18SF11-B | C7430414 | Extended | was Samtec TSW-118-07-L-D (only $6+ Samtec variants at JLCPCB); THT, optional to assemble |
| J4 | 1 | JST GH 3-pin, 1.25 mm, right angle | SMD | JST SM03B-GHS-TB(LF)(SN) | C514175 | Extended | **DNP** (not assembled). SWD is also on J3 (SWD, SWCLK, GND). Was JST SH SM03B-SRSS-TB (C160403, out of stock) |
| SW1, SW2 | 2 | Tactile switch 5.1x5.1x1.5 mm, 160 gf | SMD | XKB TS-1187A-B-A-B | C318884 | Basic | was Wurth 434133025816 (C5504987, Extended, ~$1.05); new footprint, SW1/R6 and SW2/R4 moved on the board |
| MH1-MH4 | 4 | Mounting hole 2.75 mm | - | - | - | - | no part |

Not in the BOM: PCB1, SBREV1, VARIANT1 (Raspberry Pi BOM directive symbols,
excluded from BOM), PWR_FLAG and power symbols.

## Alternatives not applied (owner decision)

- **SW1, SW2**: same-size (4.2x3.2x2.5 mm) Extended options if the original
  footprint is wanted back: SHOU HAN TS3425BA (C2681476, ~$0.04) or ALPS
  SKRPACE010 (C139797, ~$0.06). No Basic or Preferred part exists in that size.
- **X1**: to return to the Raspberry Pi qualified crystal use Abracon
  ABM8-272-T3 (C20625731, Extended, 10 pF load) with C3/C4 back at 15 pF
  (FH 0402CG150J500NT, C1548, Basic).
- **U1**: RP2354A stock is thin (13 at LCSC on 2026-11-01). The RP2354B
  (C39843328) also has 2 MB internal flash but is QFN-80 (different
  footprint). Fallback with stock: RP2350A (C42411118) plus external QSPI
  flash W25Q128JVSIQ (C97521); the flash symbol and footprint are still in
  lib/ and the original circuit is in docs/reference.
- **L1**: APV APH201610T3R3M (C5349598, Extended, 3.3 uH, 1.5 A, 2 A sat,
  180 mΩ, same 2016 size) if the Abracon part is out of stock.
- **J2, J3**: any 2x18 or cut 2x20 2.54 mm header; cheapest option is to leave
  them unpopulated and solder them by hand.
