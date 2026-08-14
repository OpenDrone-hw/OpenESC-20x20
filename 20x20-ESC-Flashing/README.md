# 20x20-ESC-Flashing: AM32 SWD flashing station

SWD flashing jig for the **OpenESC-20x20** (`../hardware/4in1-mini`). 100 x 100 mm,
4 layer, ten 1.2 mm SMD pogo pins, four ST-LINK breakout headers, 4 mm banana
jacks for target power, M3 corners. Fabbed as `20x20-flashing-V0.1`.

Pogo pins land on the ESC's B.Cu face: `/SWD1_CLK` … `/SWD4_CLK` on each
channel's PA14 test point, `/SWD1_DIO` … `/SWD4_DIO` on PA13, plus `/VBAT` and
`GND` on the battery pads so the target is powered while flashing.

The 30x30 version is `../../OpenESC-30x30/30x30-ESC-Flashing`, produced from this
board by `~/OpenDrone/software/OpenDrone-Scripts/esc/esc_jig_retarget.py`.

## Flashing

SWD once per MCU to clear readout protection and write the AM32 bootloader, then
firmware and settings over the DShot signal pin. Host side is
`../hardware/flash_openesc20.sh`: OpenOCD, AT32F421 FAP unlock, program and
verify bootloader plus firmware, `--loop` to auto-flash the next board on ST-LINK
reconnect.

## Parts

| Ref | Part | LCSC |
|---|---|---|
| TP1-TP10 | YZ118311024R-02, 1.2 mm SMD pogo | C5157376 |
| J1, J3, J5, J6 | 4-pin 2.54 mm header | C124378 |
| J2 | 4 mm banana jack, red | C7437321 |
| J4 | 4 mm banana jack, black | C7437322 |

Libraries are project-local in `4in1-ESC-Flashing.pretty/`.

License: hardware CERN-OHL-S-2.0, same as the parent repo.
