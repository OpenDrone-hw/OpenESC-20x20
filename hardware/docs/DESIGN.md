# OpenESC-20x20 Design Notes

Detailed design description of the OpenESC-20x20. Values are extracted from the KiCad design files: `hardware/4in1-mini.kicad_sch`, `hardware/ESC.kicad_sch`, `hardware/4in1-mini.kicad_pcb`.

## Architecture

Four fully independent ESC channels share a common power input and telemetry connector. Each channel has its own MCU and gate driver; the high-current stage is six MOSFETs per channel (three half-bridges). This is the distributed-MCU AM32 topology rather than a single-MCU design.

| Block | Part | LCSC | Per board |
|---|---|---|---|
| Motor MCU | AT32F421G8U7 (QFN-28) | C2765098 | 4 |
| Gate driver | NSG2065Q (QFN-24) | C41414478 | 4 |
| Power MOSFETs | DOY180N03T PowerDI3333-8 | C49441966 | 24 (6 per channel) |

## Full specifications

| Parameter | Value |
|---|---|
| Channels | 4 independent BLDC channels |
| MCU | AT32F421G8U7 (ARM Cortex-M4, QFN-28), one per channel |
| Gate driver | NSG2065Q (QFN-24, FD6288Q-compatible), one per channel |
| Power MOSFETs | DOY180N03T, N-channel, 30 V, PowerDI3333-8, 24 total |
| Current sense | Board-level high-side: INA186A3IDCKR (100 V/V, SC-70-6) across 1x 0.2 mOhm 2512 shunt in the +BATT feed, 20 mV/A, 165 A full-scale at 3.3 V ADC |
| Input | +BATT direct from connector/pads, 6S |
| Input protection | 2x SMF24A-T13 TVS (24 V standoff) |
| Buck regulator | LMR54406DBVR (SOT-23-6) + FTC160808S4R7MBCA 4.7 uH inductor, produces the +10 V gate-drive rail (FB 115k/10k, Vref 0.8 V, 10.0 V out) |
| LDO | TLV76733DRVR (WSON-6), +10 V in, +3V3 out (MCUs, sensing) |
| Signal protocol | DShot (4 independent signal lines, one per channel) |
| Firmware | AM32 (per-channel AT32F421 target, flashed individually) |
| PCB | 6-layer, 1.69 mm, outline approx. 31.3 x 33.1 mm |
| Mounting pattern | 20 x 20 mm, 4x 3.0 mm holes (M2 hardware with grommets) |

The board is 6S only: the input clamp is set by the SMF24A-T13 TVS (24 V standoff). Current and voltage ratings are not printed in the design files; the MOSFET (DOY180N03T) and current-sense full-scale (165 A) bound the practical envelope. Characterize before quoting a hard rating.

## Power tree

+BATT (6S) feeds the MOSFET drains directly, the current shunt, and the LMR54406DBVR buck. The buck produces +10 V for the four gate drivers; the TLV76733DRVR LDO drops +10 V to +3V3 for the four MCUs and the current-sense amplifier. Input clamp: 2x SMF24A-T13 TVS.

## Connector

8-pin JST **SM08B-SRSS-TB** (J1). Pin-to-net mapping extracted from the schematic (net labels at the connector pins):

| Pin | Net | Function |
|---|---|---|
| 1 | +BATT | Battery positive |
| 2 | GND | Ground |
| 3 | /CURR | Current-sense telemetry (INA186 output) |
| 4 | *(unconnected)* | No dedicated telemetry pin, telemetry handled by extended DShot |
| 5 | /M1 | DShot signal, channel 1 |
| 6 | /M2 | DShot signal, channel 2 |
| 7 | /M3 | DShot signal, channel 3 |
| 8 | /M4 | DShot signal, channel 4 |

Connector ground returns on the shield/mounting pads P1/P2 (both GND). Pin 4, the dedicated telemetry pin on the Betaflight 8-pin standard, is intentionally unconnected: ESC to FC telemetry is carried over the motor signal lines via the bidirectional **extended DShot** protocol.

## Variants and revisions

This repo is the 20x20 member of the OpenESC family. A larger sibling, [OpenESC-30x30](https://github.com/incutec-hw/OpenESC-30x30) (30.5 x 30.5 mm), shares this design and mirrors this repo; the two differ only in board/mounting size and a few power-stage parts. Fabrication sets are generated per revision into `hardware/production/` (gitignored) with the Fabrication Toolkit.

## QC fixture

`20x20-ESC-QC/` is a separate KiCad 10 project: a press-contact fixture whose board is a negative of the ESC contact face, used for functional and current-transfer tests without soldering leads. See [20x20-ESC-QC/README.md](../../20x20-ESC-QC/README.md).

## Firmware

[AM32](https://github.com/am32-firmware/AM32) is incutec's default ESC firmware. Boards ship with the AM32 bootloader pre-loaded; firmware is flashed and configured in-browser at [am32.ca](https://am32.ca). Production flashing uses `hardware/flash_openesc20.sh` (AM32 bootloader via ST-LINK). Each channel's AT32F421G8U7 is an independent AM32 target. The AT32F421 + NSG2065Q per-channel topology and the DShot signal nets are the standard AM32 hardware target for this board class. Works with Betaflight and other DShot-capable flight controllers.

## Revisions

- **Rev2-20x20** (2026-06-05): validated build, current production export set.
- **V2** (2026-05-04): export `V2`.
- **V1** (2026-03-18): export `V1`; a NextPCB variant export (`V1_nextpcb`) preceded it on 2026-03-14.
- **v0.3** (2025-11-13): export `v0.3`.
- **v0.2** (2025-11-13): export `v0.2`.
- **v0.1** (2025-11-10): first production export.
