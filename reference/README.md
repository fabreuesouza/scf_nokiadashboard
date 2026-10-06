# Nokia Reference Data

Reference datasets used to ground parameter/event logic in the dashboard (e.g. the
Neighbor Analytics > Mapping event-dynamics engine, Site Health Check) against official
Nokia definitions, instead of guessing parameter names, units or conversion formulas.

## Files

- **`nokia_lte_parameters_2026.csv`** — Nokia LTE parameter dictionary. Columns include
  `Abbreviation`, `Full name`, `MOC` (managed object class), `Full path`, `Short description`,
  `Related features`, `Data type`, `Unit`, `Internal value formula` (raw-value <-> real-unit
  conversion), default values, etc. Use this to confirm which MO a parameter actually lives on
  (e.g. `threshold2a` → LNCEL, `tac`/`phyCellId` → LNADJL for external relations), its real
  meaning/unit, and — critically — its exact conversion formula (these vary per parameter: e.g.
  `dBm = raw - 140` for most A2 RSRP thresholds, but `dBm = raw - 156` for B1-NR RSRP, `dB = raw / 2`
  for A3 offset — never assume one formula fits all).

- **`nokia_lte_kpis_27R1.csv`** — Nokia LTE KPI dictionary (release 27R1). Columns include
  `Kpi Id`, `Abbreviation`, `Name`, `Description`, `Logical Formula`, `Summarisation Formula`,
  `Unit`, `Kpi Class/Category`, `Network Element`, etc. Use this when a feature needs to
  reference or compute a standard Nokia LTE KPI, to match the official formula/counter names.

- **`nokia_gsm_kpis_2026.csv`** — Nokia GSM KPI dictionary. Same column structure as the LTE KPI
  file, for GSM/2G KPIs. Use this for any GSM-related KPI work.

- **`nokia_gsm_counters_2026.csv`** — Nokia GSM performance counter (PI) dictionary (12,290
  counters). Columns include `PI Id`, `Network Element Name`/`NetAct Name` (the actual counter
  name as it appears in NetAct/raw PM data), `Network Element Abbreviation`, `Description`,
  `Unit`, `Measurement`, `Features`, `Aggregation Dimension`, `Trigger Type`, `Logical Type`,
  `Updating Process Name`, etc. Use this to resolve a raw GSM counter referenced in a KPI formula
  (e.g. the `sum([...])`/`max([...])` logical formulas in the GSM KPI dictionary) back to its full
  definition, or to look up what a specific GSM counter actually measures.

All are `;`-delimited CSVs with a header row; open with any spreadsheet tool or
`csv.DictReader(..., delimiter=';')` in Python.
