# Agent notes

Facts for AI agents working in this repo.

- KiCad 10 project. Root schematic `hardware/OpenFC.kicad_sch` with six sub-sheets (`rp2350a`, `power`, `imu`, `osd`, `blackbox`, `pads`), board `hardware/OpenFC.kicad_pcb` (6 copper layers), panel `hardware/OpenFC_Panel.kicad_pcb`. `hardware/_filltest.kicad_pro` is a fill-test scratch project.
- Clone with `git clone --recursive`; the `libs/KiCad-Library` submodule is referenced by the project lib tables for shared parts. Project-local libraries: `hardware/lib.kicad_sym`, `hardware/lib.pretty/`, `hardware/lib.3dshapes/`.
- Never edit `.kicad_*` files as text. Use kicad-skip or the pcbnew API, and only for metadata (text variables, symbol BOM/doc fields). Never change nets, placement, or component values.
- Checks and exports:

```
kicad-cli sch erc --exit-code-violations hardware/OpenFC.kicad_sch
kicad-cli pcb drc --exit-code-violations hardware/OpenFC.kicad_pcb
kicad-cli sch export netlist --format kicadsexpr -o /tmp/OpenFC.net hardware/OpenFC.kicad_sch
```

- Analysis and metadata scripts live in `hardware/tools/` (kicad-skip / pcbnew). Most are read-only; some (`add_mpn_fields.py`, `add_emc_note.py`, `eco_strip_sheets.py`, `rebuild_blackbox.py`, `set_edgecuts_width.py`) modify design files when run with their write options.
- Fabrication Toolkit config: `hardware/fabrication-toolkit-options.json` (tracked). Exports land in `hardware/production/` (gitignored).
- Docs are deterministic: current fact only, no TODOs or plans. Design detail belongs in `hardware/docs/DESIGN.md`; staged next-revision changes in `hardware/docs/REV3_CHANGELIST.md`.
- `main` is protected; push feature branches and open PRs.
