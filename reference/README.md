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

- **`nokia_5g_kpis_26R2.csv`** — Nokia 5G NR (gNB) KPI dictionary, MIND export, release 26R2
  (1,427 KPIs). Same column structure as the LTE/GSM KPI files (`Kpi Id`, `Abbreviation`, `Name`,
  `Description`, `Logical Formula`, `Summarisation Formula`, `Summarisation Formula With PI Abbr`,
  `Unit`, `Kpi Class`/`Category`/`Subcategory`, `Network Element`, `Features`, etc.). The
  `Summarisation Formula` column references raw 5G counters in Nokia's `M<measurement_id>C<counter_number>`
  format (e.g. `[M55147C00012]`) — this is the correct Nokia 5G counter naming convention (NOT an
  Ericsson/vendor-specific format), with the matching human-readable counter name given right next
  to it in `Summarisation Formula With PI Abbr` (e.g. `[SUCC_INTER_DU_HOS]`). Use this file to map a
  raw 5G counter ID (`M...C...`) or KPI abbreviation back to its full KPI definition and formula.

All are `;`-delimited CSVs with a header row; open with any spreadsheet tool or
`csv.DictReader(..., delimiter=';')` in Python.
