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
bottom. Dependencies (`duckdb`, `pandas`, `great_tables`) are installed by the
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

Four [great_tables](https://posit-dev.github.io/great-tables/) color-shaded
tables — one row per month, DC / rest-of-country / **Total** (their sum,
so nationwide headcount is never a mental-math exercise) as separate
columns each colored on its own scale (a matplotlib `imshow` heatmap was
tried first and was hard to read at this size): headcount, hires,
separations, and net (hires − separations). The net table is the most
direct read on workforce trajectory — it's what actually surfaces the 2025
attrition spike (deep red at Jan 2025 and Sep 2025 in both areas) rather
than a hiring slowdown.

The headcount table additionally indexes each of DC/rest-of-country/Total
to the first available month (Nov 2024, pre-inauguration) as a 100%
baseline, so loss reads as a percentage — by late 2025 DC, rest-of-country,
and the nationwide total all sit around 86% of their Nov 2024 headcount.
The raw headcount columns there are deliberately left uncolored: coloring
both the raw counts (high = dark = good) and the % columns (far from
baseline = dark = bad) in the same row told two contradictory stories with
the same visual weight. Only the % columns carry color, and all three
share one domain so DC's shade is directly comparable to rest-of-country's
rather than each being scaled independently.

Accessions/separations are filtered to exactly Nov 2024–present: a file
named for one month can contain a late-processed correction with an older
effective date, and `query_monthly` filters on the data's own date column
(not just on which files get fetched) to keep those out.

`START_YM`/`END_YM` in the config cell are plain variables you can widen back
to 2005-01 (accessions/separations) or 2005-05 (employment) for full history.
