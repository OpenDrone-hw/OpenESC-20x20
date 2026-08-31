# OpenESC-20x20

Open-Source 4-in-1 sensorless BLDC motor Electronic Speed Controller (ESC), 20 x
20 mm mounting pattern (FPV Drone standard). Part of OpenDrone by Incutec
product lineup.

## Repo

| Maintainer | @Just4Stan (Discord juststan_) |
|---|---|
| Status | See the `status-*` topic on the repo. |
| Designed in | KiCad 10 |
| KiCad project | `hardware/4in1-mini.kicad_pro` |
| Root schematic | `hardware/4in1-mini.kicad_sch` (power, current sense, connector) plus `hardware/ESC.kicad_sch`, one channel instantiated 4x |
| Board | `hardware/4in1-mini.kicad_pcb`, 6 layers, 1.6 mm, 2 oz outer copper, 1 oz inner copper. |
| Fixtures | [OpenDrone-Fixtures](https://github.com/OpenDrone-hw/OpenDrone-Fixtures): `OpenESC-20x20-QC/` press-contact QC fixture, `OpenESC-20x20-Flashing/` ST-LINK pogo-pin jig used by the flash script |
| Local library | `hardware/components.kicad_sym`, `hardware/4in1ESC.pretty/`, `hardware/4in1ESC.3dshapes/`. Frozen pre-consolidation libraries: use them, do not add to them |
| Shared library | [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), Contains all components used on produced (Alpha onwards) hardware. Work in local library and migrate to shared library once component selection is fixed. |
| Design rules | `hardware/4in1-mini.kicad_dru` |
| Fab config | `hardware/fabrication-toolkit-options.json` |
| Board setup | 6 layers, 0.09 mm clearance and track, 0.16mm on outer layers (2 oz), via 0.35 on 0.20 drill |
| License | CERN-OHL-S-2.0 |

The project is named `4in1-mini`, not after the repo. Renaming it would break
the fab archive names, the release assets and the website board art.

## Rules

Identical in every OpenDrone board repo. Do not edit here; edit the template.

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
- **Metadata yes, connections no.** An agent may write BOM and documentation
  fields (MPN, Manufacturer, LCSC, Cost, Datasheet, text variables). An agent
  may not change nets, wiring, routing, placement, footprint assignment, or any
  value that changes the circuit.
- **Close KiCad before any write to a KiCad file.** KiCad caches library tables
  at process start and overwrites files on save.
- **Reuse before you draw.** Check
  [KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library) and its
  `PARTS-USED.md` first. If the part is there we have already sourced,
  footprinted and shipped it: copy the symbol and footprint into this repo's
  `lib` library and use it. Draw a new part only when the library has nothing
  that fits, and import it with `easyeda2kicad` from its LCSC number.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **ERC and DRC clean before every pull request.** Commands below.

## Environment

```sh
# schematic and board checks
kicad-cli sch erc --exit-code-violations hardware/4in1-mini.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones --exit-code-violations hardware/4in1-mini.kicad_pcb

# netlist, for scripted analysis
kicad-cli sch export netlist --format kicadsexpr -o /tmp/4in1-mini.net hardware/4in1-mini.kicad_sch
```

Shared scripts (renders, STEP export, packaging art) live in
`OpenDrone-Scripts`; board-specific scripts live in `hardware/tools/`.

## Architecture

Four independent channels share one power input and one connector. Per channel:
an **AT32F421G8U7** (Cortex-M4, QFN-28) drives an **NSG2065Q** three-phase
half-bridge gate driver, which drives six **DOY180N03T** MOSFETs, two per phase.
One channel is drawn once in `ESC.kicad_sch` and instantiated four times.

Current sensing is **board level, not per motor**: an INA186A3IDCKR at 100 V/V
across a single 0.2 mOhm 2512 shunt in the +BATT feed. That gives 20 mV/A and
165 A full scale against a 3.3 V ADC, reported as `/CURR`. The 30x30 sibling
uses two shunts in parallel and reads twice the current at half the sensitivity.

## Key parts

| Function | Ref | Part | LCSC | Note |
|---|---|---|---|---|
| Motor MCU, x4 | U2, U6, U8, U10 | AT32F421G8U7, QFN-28 | C2765098 | One per channel. |
| Gate driver, x4 | U3, U7, U9, U11 | NSG2065Q, QFN-24 | C41414478 | Standard footprint, many alternatives exist. |
| Power MOSFET, x24 | Q1-Q24 | DOY180N03T, PowerDI3333-8 | C49441966 | 30 V, 6 per channel. Standard 3x3 DFN footprint, many alternatives exist. |
| Current sense amp | U12 | INA186A3IDCKR, SC-70-6 | C2058245 | 100 V/V, board level high side |
| Current shunt | Rsense1 | 0.2 mOhm 2512 | C695806 | Single, in the +BATT feed |
| Buck | U13 | LMR54406DBVR, SOT-23-6 | C5219316 | 36 V in, 0.6 A; FB 115k/10k against 0.8 V, 10.0 V out |
| Buck inductor | U5 | FTC160808S4R7MBCA | C46594347 | 4.7 uH |
| LDO | U1 | TLV76733DRVR, WSON-6 | C2848334 | +10 V to +3V3 |
| Connector | J1 | SM08B-SRSS-TB, JST SH 8-pin | C160407 | Also broken out as solder pads. |
| Bulk electrolytic. | / | 470 uF 35V | / | To be installed on the battery connector by the user. |
| Bulk ceramic | fix | fix | fix | 22 x 10 uF 1206 |

## Power

```
Battery + (2S-6S) ─► 0.2mOhm shunt ─► +BATT
+BATT ─┬─► MOSFET drains, motor phases
       └─► LMR54406DBVR buck ─► +10V ─┬─► 4x gate driver
                                      └─► TLV76733DRVR ─► +3V3 ─► 4x MCU, INA186
```

## Connectors and I/O

Betaflight Standard:

| Pin | Net | Function |
|---|---|---|
| 1 | +BATT | Battery positive |
| 2 | GND | Ground |
| 3 | /CURR | Current sense telemetry, INA186 output |
| 4 | unconnected | See below |
| 5 | /M1 | DShot, channel 1 |
| 6 | /M2 | DShot, channel 2 |
| 7 | /M3 | DShot, channel 3 |
| 8 | /M4 | DShot, channel 4 |

Pin 4 is the dedicated telemetry pin in the Betaflight 8-pin standard and is
intentionally left unconnected: telemetry rides the motor signal lines over
bidirectional extended DShot instead.

## Firmware

[AM32 first needs a boatloader loaded using an ST-LINK
(AM32_F421_BOOTLOADER_PB4_V19.hex)
](https://github.com/am32-firmware/AM32) firmware is flashed and configured
in-browser at [am32.ca](https://am32.ca). Works with Betaflight and any other
DShot-capable flight controller.

## Layout rules

Bulk decoupling on +BATT and GND exists on the PCB without matching schematic
symbols. That is a deliberate board-only bank. Do not run
update-from-schematic without checking what it would delete.

## Revisions

| Rev | Date | Change |
|---|---|---|
| Rev3.3 | 2026-08-25 | Export OpenESC-20x20-rev3.3. Silkscreen rebranded OpenDrone -> incutec for export restriction reasons on flagging anything containing 'Drone'. First Incutec production run. |
| Rev3.2 | 2026-08-22 | Export OpenESC-20x20-rev3.2. Matched input network at the current-sense amplifier (R89/R90 1k, C40/C41 100n 50V, C42 1u) against the high-side common-mode feedthrough. |
| Rev3.1 | 2026-08-14 | Export `20x20_ESC_Rev3.1`. Bulk bank: 22 x 10 uF 1206 on +BATT/GND, 21 of them PCB-only (19 CL refs absent from the schematic, CL50/CL51 doubled). Board setup moved to the line standard. |
| Rev3 | 2026-08-11 | Input clamp diodes D1-D3 removed, TVS diodes offer no protection when rail voltage is this close to the MOSFET Vds. |
| Rev2-20x20 | 2026-06-05 | Validated build. 15 pieces ordered by Incutec. |
| V2 | 2026-05-04 | Export `V2`. |
| V1 | 2026-03-18 | Export `V1`. A NextPCB variant export `V1_nextpcb` preceded it on 2026-03-14. |
| v0.3 | 2025-11-13 | Export `v0.3`. |
| v0.2 | 2025-11-13 | Export `v0.2`. |
| v0.1 | 2025-11-10 | First production export. |
