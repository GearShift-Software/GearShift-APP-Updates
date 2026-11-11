# GearShift hardware databases

GearShift detects the user's parts and looks them up in these files:

| File | What it holds | Matched against |
| --- | --- | --- |
| `gpus.json` | OC / undervolt profiles, safe values, sources, notes | Windows GPU name (e.g. `AMD Radeon RX 6800 XT`) |
| `cpus.json` | Best tuning, RAM sweet spot, BIOS tips, known issues | Windows CPU name (e.g. `13th Gen Intel(R) Core(TM) i5-13600KF`) |
| `memory.json` | XMP/EXPO and stick advice | RAM type from SMBIOS (`DDR4`, `DDR5`) |
| `boards.json` | Chipset features, BIOS menu names per vendor | Board model (e.g. `B760M Pro RS`) and maker |

## How matching works

1. **models** are checked first. The longest match wins, so `RX 6800 XT` beats `RX 6800`.
2. If no model matches, the first **family** in file order that matches is used (put generic families last).
3. A model only lists what is different: everything else comes from its `family`.

`match` entries are regular expressions, case-insensitive. `(R)`, `(TM)`, ® and ™ are removed from names before matching.

## Add a new card (example)

```json
{ "id": "rx9070gre", "name": "Radeon RX 9070 GRE", "family": "rdna4",
  "match": ["RX\\s*9070\\s*GRE"],
  "stock": "Board power from the official spec page.",
  "notes": ["One line per fact, with the source in src."],
  "src": [ { "T": "What the test found.", "S": "Site name", "U": "https://..." } ] }
```

A model can also carry its own `profiles` (always 4: Silent, Balanced, Performance, Max OC), `safe` rows (4 columns)
and `tips`. Each profile needs `N`, `D`, `L`, `Pw`, `Pf`, `Nz` (1-3) and `Fan` (`Silent`, `Balanced` or `Performance`).

## Rules

- Every number comes from a public test or an official spec, linked in `src`.
- Raise `version` (yyyymmdd) when you change a file. Users get the update automatically when `LibraryUrl` points at
  your GitHub copy of `library/games.json`; the app fetches `library/hw/*.json` next to it.
- Run `python tests/hwdb_check.py` before publishing: it checks the format and that real device names land on the
  right entry. Add a test line for every new model.
