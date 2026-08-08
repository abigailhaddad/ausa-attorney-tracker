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

Three live queries, each grouped by month and by area (DC / rest-of-country):

- **Employment** — monthly headcount snapshots (`snapshot_yyyymm`)
- **Accessions** — monthly hires (`personnel_action_effective_date_yyyymm`)
- **Separations** — monthly departures (same date field)

Four heatmaps (year x month, DC and rest-of-country side by side):
headcount, hires, separations, and net (hires − separations). The net heatmap
is the most direct read on workforce trajectory — it's what actually surfaces
the 2025 attrition spike (separations roughly tripled that year against a
decade-long ~350–590/year baseline) rather than a hiring slowdown.

`START_YM`/`END_YM` in the config cell default to 2015-01–present, comfortably
inside the well-populated EHRI schema era; both are plain variables you can
widen back to 2005-01 (accessions/separations) or 2005-05 (employment) if you
want the full history — expect the employment query to take a few minutes
longer since those files run 26–75 MiB each vs. accessions'/separations'
KiB-to-low-MiB files.
