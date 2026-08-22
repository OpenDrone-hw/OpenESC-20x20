# OpenESC-20x20

Open source 4-in-1 BLDC ESC, 20 x 20 mm mounting pattern. Four fully
independent channels, each with its own MCU, gate driver and six MOSFETs: the
distributed-MCU AM32 topology, not a single shared MCU. It is not an 8S board;
an 8S board is a separate SKU.

## Repo

| | |
|---|---|
| Maintainer | @Just4Stan |
| Status | See the `status-*` topic on the repo. Never written here. |
| Designed in | KiCad 10 |
| KiCad project | `hardware/4in1-mini.kicad_pro` |
| Root schematic | `hardware/4in1-mini.kicad_sch` (power, current sense, connector) plus `hardware/ESC.kicad_sch`, one channel instantiated 4x |
| Board | `hardware/4in1-mini.kicad_pcb`, 6 layers, 1.6 mm (the stackup field reads 1.69 mm; JLC ships 1.6) |
| Fixtures | [OpenDrone-Fixtures](https://github.com/OpenDrone-hw/OpenDrone-Fixtures): `OpenESC-20x20-QC/` press-contact QC fixture, `OpenESC-20x20-Flashing/` ST-LINK pogo-pin jig used by the flash script |
| Local library | `hardware/components.kicad_sym`, `hardware/4in1ESC.pretty/`, `hardware/4in1ESC.3dshapes/`. Frozen pre-consolidation libraries: use them, do not add to them |
| Shared library | [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), catalogue only; every library this board uses is local to the repo |
| Design rules | `hardware/4in1-mini.kicad_dru` |
| Fab config | `hardware/fabrication-toolkit-options.json` |
| Board setup | Line standard: 6 layers, 0.09 mm clearance and track, via 0.35 on 0.20 drill |
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

On macOS `kicad-cli` is at
`/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`, and `pcbnew` imports
only under KiCad's bundled Python. Shared scripts (renders, STEP export,
packaging art) live in `OpenDrone-Scripts`; board-specific scripts live in
`hardware/tools/`.

`--refill-zones` stops stale fills inventing clearance errors.
`--schematic-parity` is noisy on this board because of the PCB-only bulk bank
(see Layout rules), so a real parity error hides easily.

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

**Rev3.2 adds a matched input network at the amplifier**: 1 k in each sense
leg (R91/R92), 100 nF 50 V from each input to ground (C52/C54) and 1 uF
across the inputs (C51). Through rev3.1 the bare high-side connection made
`/CURR` unusable below full throttle, measured 2026-08-21: both amplifier
inputs ride the switching bus and the common-mode feedthrough rectifies to a
duty-dependent offset. Against a current clamp the chain read +164% at 15%
throttle, hard-failed at 80% (the raw ADC count itself collapsed from ~680
to ~290 while the clamp held a steady 32 A), and was honest only at 100%
where AM32 stops chopping. The network attenuates the common-mode at the
pins ~300x and the mismatch-converted differential to 2.8 mVpp worst case
(SPICE, 1% R / 10% C opposed); scale is unchanged at 20 mV/A. Not yet
verified on hardware: the acceptance test is the ESC-04 mid-throttle
i_a-versus-clamp sweep on a rev3.2 board. On rev3.1 and earlier, anything
consuming `/CURR` (Betaflight OSD current, mAh-drawn battery math) gets
garbage at flight throttles. Evidence:
`OpenDrone-Testing/Logs/esc-04-20x20-20260821T150430Z/`.

**Thermal envelope, measured 2026-08-21 on one board.** Continuous 30-34 A
per channel fan-cooled at a 115 C die; die plateau demonstrated at 26 A, the
rest of the bracket extrapolated from ramp data. Burst 64 A for 2 s on one
channel, 112 A board-level for 2.4 s, die still rising 4-6 C/s at step end.
Destruction at ~40 A sustained: the board fails by SOLDER REFLOW, two FETs
desoldered (joints past 217 C) while the die read 161 C, so the die
under-reads the FET joint by roughly 50 C and heat extraction, not silicon,
is the ceiling. Thermal time constant 4.2-5.3 s. Evidence:
`OpenDrone-Testing/Logs/esc-18-20x20-20260821T181141Z/` and
`esc-14-20x20-20260821T154902Z/`.

**As-flashed AM32 config, audited 2026-08-21.** Boards flash with
`limits.temperature = 255`, so AM32's own thermal foldback never engages and
nothing on the board protects the FETs; `motor_kv` defaults to the 2808
preset (decodes 2220 Kv), which caps duty near 0.26 on lower-Kv motors via
`low_rpm_level`/`high_rpm_level`. Both must be set per product in the
flashing jig before boards ship.

**There is no input protection.** A clamp diode's 24 V standoff sits below
the 25.2 V a full 6S pack reaches, so none is fitted on the 2S-6S input.

## Key parts

| Function | Ref | Part | LCSC | Note |
|---|---|---|---|---|
| Motor MCU, x4 | U2, U6, U8, U10 | AT32F421G8U7, QFN-28 | C2765098 | One per channel, independent AM32 target |
| Gate driver, x4 | U3, U7, U9, U11 | NSG2065Q, QFN-24 | C41414478 | FD6288Q compatible, integrated bootstrap diodes |
| Power MOSFET, x24 | Q1-Q24 | DOY180N03T, PowerDI3333-8 | C49441966 | 30 V, 6 per channel |
| Current sense amp | U12 | INA186A3IDCKR, SC-70-6 | C2058245 | 100 V/V, board level high side |
| Current shunt | Rsense1 | 0.2 mOhm 2512 | C695806 | Single, in the +BATT feed |
| Buck | U13 | LMR54406DBVR, SOT-23-6 | C5219316 | 36 V in, 0.6 A; FB 115k/10k against 0.8 V, 10.0 V out |
| Buck inductor | U5 | FTC160808S4R7MBCA | C46594347 | 4.7 uH |
| LDO | U1 | TLV76733DRVR, WSON-6 | C2848334 | +10 V to +3V3 |
| Connector | J1 | SM08B-SRSS-TB, JST SH 8-pin | C160407 | |
| Bulk electrolytic, supplied | n/a | 470 uF | | Shipped with the board, not fitted to it. The user solders it across the battery terminals. Standard pairing of on-board ceramics with a pack-side elco; it dominates the bus capacitance once installed |

## Power

```
Battery + (2S-6S) ─► 0.2mOhm shunt (INA186A3 across it ─► /CURR) ─► +BATT
+BATT ─┬─► MOSFET drains, motor phases
       └─► LMR54406DBVR buck + 4.7uH ─► +10V ─┬─► 4x gate driver
                                              └─► TLV76733DRVR ─► +3V3 ─► 4x MCU, INA186
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

Production flashing: `hardware/flash_openesc20.sh` writes the AM32 F421 PB4
bootloader and the `AM32_OPENESC_20_F421` firmware build over an ST-LINK V2
and the `OpenESC-20x20-Flashing` pogo-pin jig from OpenDrone-Fixtures. It requires `AM32_UNLOCKER_DIR`
(AM32-unlocker checkout: bundled openocd, probe config, bootloaders) and
`AM32_DIR` (AM32 checkout with the OpenESC_20 target built).

## Layout rules

Bulk decoupling on +BATT and GND exists on the PCB without matching schematic
symbols. That is a deliberate board-only bank. Do not run
update-from-schematic without checking what it would delete.

## Revisions

| Rev | Date | Change |
|---|---|---|
| Rev3.2 | 2026-08-22 | Export `20x20_ESC_Rev3.2`. Matched input network at the current-sense amplifier (R91/R92 1k, C52/C54 100n 50V, C51 1u) against the high-side common-mode feedthrough; shunt redrawn as Rsense2. Scale unchanged, 20 mV/A. |
| Rev3.1 | 2026-08-14 | Export `20x20_ESC_Rev3.1`, current. Bulk bank: 22 x 10 uF 1206 on +BATT/GND, 21 of them PCB-only (19 CL refs absent from the schematic, CL50/CL51 doubled). Board setup moved to the line standard. |
| Rev3 | 2026-08-11 | Rev3 tag. Input clamp diodes (D1, D2) removed. |
| Rev2-20x20 | 2026-06-05 | Validated build. |
| V2 | 2026-05-04 | Export `V2`. |
| V1 | 2026-03-18 | Export `V1`. A NextPCB variant export `V1_nextpcb` preceded it on 2026-03-14. |
| v0.3 | 2025-11-13 | Export `v0.3`. |
| v0.2 | 2025-11-13 | Export `v0.2`. |
| v0.1 | 2025-11-10 | First production export. |
