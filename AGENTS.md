# Agent notes

Facts for AI agents working in this repo.

- KiCad 10 project. Top schematic `hardware/4in1-mini.kicad_sch`, channel sheet `hardware/ESC.kicad_sch`, board `hardware/4in1-mini.kicad_pcb` (6 copper layers). Separate QC fixture project in `20x20-ESC-QC/`.
- Clone with `git clone --recursive`; the `libs/KiCad-Library` submodule is referenced by the project lib tables for shared parts. Symbols and footprints are embedded in the design files.
- Never edit `.kicad_*` files as text. Use kicad-skip or the pcbnew API, and only for metadata (text variables, symbol BOM/doc fields). Never change nets, placement, or component values.
- Checks and exports:

```
kicad-cli sch erc --exit-code-violations hardware/4in1-mini.kicad_sch
kicad-cli pcb drc --exit-code-violations hardware/4in1-mini.kicad_pcb
kicad-cli sch export netlist --format kicadsexpr -o /tmp/4in1-mini.net hardware/4in1-mini.kicad_sch
```

- Fabrication Toolkit config: `hardware/fabrication-toolkit-options.json` (tracked). Exports land in `hardware/production/` (gitignored).
- Vendor datasheet PDFs in `hardware/datasheets/` are local only (gitignored); `COMPONENT_REVIEW.md` there is tracked.
- Docs are deterministic: current fact only, no TODOs or plans.
- `main` is protected; push feature branches and open PRs.
