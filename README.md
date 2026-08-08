# AUSA attorney tracker

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/abigailhaddad/ausa-attorney-tracker/blob/main/ausa_attorney_tracker.ipynb)

One notebook. Monthly headcount, hiring, and departures for Assistant U.S.
Attorneys (DOJ, Executive Office for U.S. Attorneys and the Offices of the
U.S. Attorneys, occupational series 0905), queried **live** from the public
OPM/EHRI mirror on HuggingFace
([`impactproject/opm-ehri-data`](https://huggingface.co/datasets/impactproject/opm-ehri-data))
with DuckDB over HTTPS. Nothing is downloaded to disk.

**Run it:** click the *Open in Colab* badge, or open
[`ausa_attorney_tracker.ipynb`](ausa_attorney_tracker.ipynb) and run top to
bottom. Dependencies (`duckdb`, `pandas`, `matplotlib`) are installed by the
notebook if missing.

## Scope: DC vs. rest-of-country, not state-by-state

The obvious next step — breaking AUSA hiring down by state or district — isn't
possible with this data. Every geographic field (`duty_station_state_abbreviation`,
`duty_station_city`, `core_based_statistical_area`) is privacy-redacted for
**~91% of AUSA records** (checked directly: 5,849 of 6,400 employment records,
Dec 2024) — a small-occupational-subgroup suppression rule applied uniformly
across the whole location hierarchy, not specific to any one field. DC is the
one exception: its ~500-attorney cell is large enough to clear the suppression
threshold and reports a real number. So the only two honest "area" buckets are
**DC** and **rest-of-country (aggregate)** — the notebook does not fabricate
state-level detail the underlying data doesn't actually contain.

## What's in the notebook

Three live queries, each grouped by month and by area (DC / rest-of-country),
covering **Nov 2024 (pre-inauguration baseline) through present**:

- **Accessions** — monthly hires (`personnel_action_effective_date_yyyymm`)
- **Separations** — monthly departures (same date field)
- **Employment** — headcount snapshots (`snapshot_yyyymm`), sampled every 3rd
  available month (~quarterly) rather than pulled monthly — those files are
  the full federal workforce each month (26–75 MiB apiece) filtered down to
  ~6,000 AUSA rows, and querying all of them monthly is both slow and enough
  to trip HuggingFace's rate limit on repeated large-file reads. `EMPLOYMENT_SAMPLE_STRIDE`
  in the config cell controls this — set it to 1 for full monthly resolution
  if you're willing to wait longer.

Four heatmaps — one row per area (DC, rest-of-country), columns = every
distinct month present, values annotated on each cell: headcount, hires,
separations, and net (hires − separations). The net heatmap is the most
direct read on workforce trajectory — it's what actually surfaces the 2025
attrition spike (deep losses concentrated at Jan 2025 and Sep 2025 in both
areas) rather than a hiring slowdown.

A handful of stray months outside Nov 2024–present can show up in the
accessions/separations heatmaps: a file named for one month can contain a
late-processed correction with an older effective date, and those are read
as real events straight from the data's own date column rather than filtered
out. Accessions' and separations' stray months don't line up with each
other — hires and departures are independent transaction streams with their
own separate correction cycles.

`START_YM`/`END_YM` in the config cell are plain variables you can widen back
to 2005-01 (accessions/separations) or 2005-05 (employment) for full history.
