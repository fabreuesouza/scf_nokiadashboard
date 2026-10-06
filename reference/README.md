# Nokia LTE Reference Data

Reference datasets used to ground parameter/event logic in the dashboard (e.g. the
Neighbor Analytics > Mapping event-dynamics engine) against official Nokia definitions,
instead of guessing parameter names or units.

## Files

- **`nokia_lte_parameters_2026.csv`** — Nokia LTE parameter dictionary. Columns include
  `Abbreviation`, `Full name`, `MOC` (managed object class), `Full path`, `Short description`,
  `Related features`, `Data type`, `Unit`, default values, etc. Use this to confirm which MO a
  parameter actually lives on (e.g. `threshold2a` → LNCEL, `tac`/`phyCellId` → LNADJL for
  external relations) and its real meaning/unit before wiring it into the tool.

- **`nokia_lte_kpis_27R1.csv`** — Nokia LTE KPI dictionary (release 27R1). Columns include
  `Kpi Id`, `Abbreviation`, `Name`, `Description`, `Logical Formula`, `Summarisation Formula`,
  `Unit`, `Kpi Class/Category`, `Network Element`, etc. Use this when a feature needs to
  reference or compute a standard Nokia KPI, to match the official formula/counter names.

Both are `;`-delimited CSVs with a header row; open with any spreadsheet tool or
`csv.DictReader(..., delimiter=';')` in Python.
