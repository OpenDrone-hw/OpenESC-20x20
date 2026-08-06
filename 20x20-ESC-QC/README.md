# 20x20-ESC-QC: press-contact QC fixture

Bench test/QC jig for the **OpenESC-20x20** (`../hardware/4in1-mini`). The
assembled ESC is pressed face-down onto this board so spring/pogo contacts land
on the ESC's exposed pads (power in, three phases per channel, signal lines) to
run functional and current-transfer tests without soldering leads.

Concept: this board is a **negative** of the ESC contact face. The ESC nests
into a milled/offset pocket for alignment, and contact features sit under each
of its exposed pads.

## Layout / conventions (mirrors the parent repo)

- KiCad 10 project: `20x20-ESC-QC.kicad_{pro,sch,pcb}`. The schematic is an empty
  stub: the fixture is laid out directly on the board, with no netlist.
- Project-local libraries, declared in `sym-lib-table` and `fp-lib-table`:
  - Symbols: `ESC-QC.kicad_sym` (one symbol, `4in1ESC-negative`, currently unused
    because the schematic is a stub)
  - Footprints: `ESC-QC.pretty/` (one footprint, `4in1ESC-negative`: 34 pads plus
    the Edge.Cuts geometry of the ESC contact face)
  - 3D models: `ESC-QC.3dshapes/` (empty)
- The 44 contact pads come from KiCad's stock global `TestPoint` library
  (`TestPoint_Pad_2.0x2.0mm`, `3.0x3.0mm`, `4.0x4.0mm`), not from a project-local
  library. Nothing else resolves globally.
- License: hardware CERN-OHL-S-2.0 (same as parent).

## Reference geometry

The contact locations must match the ESC's exposed pads exactly. Pad
coordinates and nets are extracted read-only from the source board,
`../hardware/4in1-mini.kicad_pcb` (kicad-cli or the pcbnew API); nothing in the
ESC design is modified.

## Contacts to hit (from the ESC design)

- **+BATT / GND**: high-current, the press-contact points that matter most.
- **Motor phases**: 3 per channel x 4 channels (A/B/C) motor output pads.
- **Signal**: 4 DShot lines (/M1-/M4) plus +BATT, GND, and /CURR on J1
  (SM08B-SRSS-TB), or its pad footprint if contacting the pads directly.
- Optional: SWD/boot test pads for the AT32F421 per channel.
