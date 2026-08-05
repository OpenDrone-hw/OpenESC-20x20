# 20x20-ESC-QC — Press-contact QC fixture

Bench test/QC jig for the **OpenESC-20x20** (`../hardware/4in1-mini`). The
assembled ESC is pressed face-down onto this board so spring/pogo contacts land
on the ESC's exposed pads — power in, three phases per channel, signal lines —
to run functional and current-transfer tests without soldering leads.

Concept: this board is a **negative** of the ESC contact face. The ESC nests
into a milled/offset pocket (alignment), and contact features sit under each of
its exposed pads. Design the actual contacts, pocket, and offsets in KiCad —
this folder is scaffolded and ready.

## Layout / conventions (mirrors the parent repo)
- KiCad 10 project: `20x20-ESC-QC.kicad_{pro,sch,pcb}`.
- **Project-local libraries only** (never global):
  - Symbols: `ESC-QC.kicad_sym` (via `sym-lib-table`)
  - Footprints: `ESC-QC.pretty/` (via `fp-lib-table`)
  - 3D models: `ESC-QC.3dshapes/`
- License: hardware CERN-OHL-S-2.0 (same as parent).

## Building the negative — reference geometry
The contact locations must match the ESC's exposed pads exactly. Pull the ESC
board outline + pad coordinates/nets from the source PCB as a reference:

```
kicad-cli pcb export ...   # or pcbnew API against ../hardware/4in1-mini.kicad_pcb
```

Ask Claude to extract the ESC contact-pad table (net, X/Y, size, layer) if you
want a coordinate reference to lay the pogo pins against — read-only against the
source, nothing in the ESC design is modified.

## Contacts to hit (from the ESC design)
- **+BATT / GND** — high-current, the press-contact points that matter most.
- **Motor phases** — 3 per channel × 4 channels (A/B/C) motor output pads.
- **Signal** — 4 DShot lines + 5V/GND on J1 (SM08B-SRSS-TB), or its pad
  footprint if contacting the pads directly.
- Optional: SWD/boot test pads for the AT32F421 per channel.
