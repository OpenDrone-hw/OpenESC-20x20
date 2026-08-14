# OpenESC-20x20

Open source 4-in-1 BLDC ESC, 20 x 20 mm mounting pattern. Four fully
independent channels, each with its own MCU, gate driver and six MOSFETs: the
distributed-MCU AM32 topology, not a single shared MCU. Control is DShot over
the standard 8-pin connector. 6S only, and there is no input TVS.

## Repo

| | |
|---|---|
| Maintainer | @Just4Stan |
| Status | See the `status-*` topic on the repo. Never written here. |
| Designed in | KiCad 10 |
| KiCad project | `hardware/4in1-mini.kicad_pro` |
| Root schematic | `hardware/4in1-mini.kicad_sch` (power, current sense, connector) plus `hardware/ESC.kicad_sch`, one channel instantiated 4x |
| Board | `hardware/4in1-mini.kicad_pcb`, 6 layers, 1.69 mm, outline approx 31.3 x 33.1 mm |
| QC fixture | `20x20-ESC-QC/`, a separate project. Press-contact fixture, board is a negative of the ESC contact face |
| Local library | `hardware/components.kicad_sym`, `hardware/4in1ESC.pretty/`. Frozen pre-consolidation libraries: use them, do not add to them |
| Shared library | [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), catalogue only; every library this board uses is local to the repo |
| Design rules | `hardware/4in1-mini.kicad_dru` |
| Fab config | `hardware/fabrication-toolkit-options.json` |
| License | CERN-OHL-S-2.0 |

The project is named `4in1-mini`, not after the repo. Renaming it would break
the fab archive names, the release assets and the website board art.

## Rules

Identical in every OpenDrone repo. Do not edit here; edit the template.

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
- **Metadata yes, connections no.** An agent may write BOM and documentation
  fields. An agent may not change nets, wiring, routing, placement, footprint
  assignment, or any value that changes the circuit.
- **Close KiCad before any write to a KiCad file.**
- **Reuse before you draw.** Check
  [KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library) and its
  `PARTS-USED.md` first, and copy what fits into this repo's own library.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **ERC and DRC clean before every pull request.**

```sh
kicad-cli sch erc --exit-code-violations hardware/4in1-mini.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones --exit-code-violations hardware/4in1-mini.kicad_pcb
```

`--refill-zones` stops stale fills inventing clearance errors. `--schematic-parity`
matters here in particular: this board carries bulk capacitors that exist on the
PCB with no symbol behind them, so parity is noisy and a real error hides easily.

## Architecture

Four independent channels share one power input and one connector. Per channel:
an **AT32F421G8U7** (Cortex-M4, QFN-28) drives an **NSG2065Q** three-phase
half-bridge gate driver, which drives six **DOY180N03T** MOSFETs, two per phase.
One channel is drawn once in `ESC.kicad_sch` and instantiated four times.

Current sensing is **board level, not per channel**: an INA186A3IDCKR at
100 V/V across a single 0.2 mOhm 2512 shunt in the +BATT feed. That gives
20 mV/A and 165 A full scale against a 3.3 V ADC, reported as `/CURR`. The
30x30 sibling uses two shunts in parallel and reads twice the current at half
the sensitivity.

**There is no input protection.** The two SMF24A-T13 TVS earlier revisions
carried are gone: a 24 V standoff sits below the 25.2 V a full 6S pack reaches.
The board is 6S only. The MOSFET and the 165 A sense full scale bound the
practical envelope; characterise before quoting a hard rating.

## Key parts

| Function | Ref | Part | LCSC | Note |
|---|---|---|---|---|
| Motor MCU, x4 | | AT32F421G8U7, QFN-28 | C2765098 | One per channel, independent AM32 target |
| Gate driver, x4 | | NSG2065Q, QFN-24 | C41414478 | FD6288Q compatible |
| Power MOSFET, x24 | | DOY180N03T, PowerDI3333-8 | C49441966 | 30 V, 6 per channel |
| Current sense amp | | INA186A3IDCKR, SC-70-6 | C2058245 | 100 V/V, board level high side |
| Current shunt | | 0.2 mOhm 2512 | | Single, in the +BATT feed |
| Buck | | LMR54406DBVR, SOT-23-6 | C5219316 | FB 115k/10k against 0.8 V, 10.0 V out |
| Buck inductor | | FTC160808S4R7MBCA | C46594347 | 4.7 uH |
| LDO | | TLV76733DRVR, WSON-6 | C2848334 | +10 V to +3V3 |
| Connector | J1 | SM08B-SRSS-TB, JST SH 8-pin | C160407 | |

## Power

```
+BATT (6S) ─┬─► MOSFET drains, motor phases
            ├─► 0.2mOhm shunt ─► INA186A3 ─► /CURR
            └─► LMR54406DBVR buck + 4.7uH ─► +10V ─┬─► 4x gate driver
                                                    └─► TLV76733DRVR ─► +3V3 ─► 4x MCU, INA186
no input TVS
```

## Connectors and I/O

8-pin JST SM08B-SRSS-TB (J1). Connector ground returns on pads P1 and P2.

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

[AM32](https://github.com/am32-firmware/AM32), one independent target per
channel. Boards ship with the AM32 bootloader pre-loaded; firmware is flashed
and configured in-browser at [am32.ca](https://am32.ca). Works with Betaflight
and any other DShot-capable flight controller.

## Layout rules

Bulk decoupling on +BATT and GND exists on the PCB without matching schematic
symbols. That is a deliberate board-only bank, and it is why DRC parity reports
are noisy here. Do not run update-from-schematic without checking what it would
delete.

Mounting is 4x 3.0 mm holes for M2 hardware with grommets, on a 20 x 20 mm
pattern. The board outline is larger than the mounting pattern.

## Revisions

| Rev | Date | Change |
|---|---|---|
| Rev3.1 | 2026-08-14 | Export `20x20_ESC_Rev3.1`, current. Bulk bank: 22 x 10 uF 1206 on +BATT/GND, 21 of them PCB-only (19 CL refs absent from the schematic, CL50/CL51 doubled). Board setup on the line standard. |
| Rev3 | 2026-08-11 | Rev3 tag. |
| Rev2-20x20 | 2026-06-05 | Validated build. |
| V2 | 2026-05-04 | Export `V2`. |
| V1 | 2026-03-18 | Export `V1`. A NextPCB variant export `V1_nextpcb` preceded it on 2026-03-14. |
| v0.3 | 2025-11-13 | Export `v0.3`. |
| v0.2 | 2025-11-13 | Export `v0.2`. |
| v0.1 | 2025-11-10 | First production export. |
