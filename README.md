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

## Scope: career attorneys only, not political appointees

`appointment_type` is used to exclude political appointments from every
query, so hiring/departure/headcount figures track the career AUSA
workforce rather than political leadership turnover. Three distinct legal
categories of political appointment show up in this data, and all three
are excluded (`POLITICAL_APPOINTMENT_TYPES` in the config cell):

- **Schedule C** (5 CFR 213.3301) — confidential/policy staff
- **Noncareer SES** — senior-executive-level political appointees
- **`EXECUTIVE (EXCEPTED SERVICE NONPERMANENT)`** — always pay plan `AD`,
  grade `40`, "supervisor or manager," one single consistent combination —
  consistent with the U.S. Attorney / top leadership slot per district
  (Presidentially-appointed, Senate-confirmed), not a career classification

Schedule C alone would miss the other two — it's only the "junior" tier of
political appointment, not an umbrella term for all of it, and it would
specifically miss the U.S. Attorney position itself, arguably the single
most obviously political role in the office.

One value that stays IN despite sounding temporary:
`OTHER (EXCEPTED SERVICE NONPERMANENT)` is the single largest bucket of
ordinary AUSA hires. Checked back to 2015 — it's the dominant appointment
code for AUSA hiring in every year on record, not a marker of temporary or
surge staffing specific to any period. Excluding it would have gutted
~98% of all AUSA hiring data going back a decade, for no real signal.

## What's in the notebook

Three live queries, each grouped by month and by area (DC / rest-of-country),
covering **Nov 2024 (pre-inauguration baseline) through present**:

- **Accessions** — monthly hires (`personnel_action_effective_date_yyyymm`)
- **Separations** — monthly departures (same date field)
- **Employment** — headcount snapshots (`snapshot_yyyymm`), pulled every 3rd
  available month (a fixed quarterly interval, not a statistical sample)
  rather than every month — those files are the full federal workforce each
  month (26–75 MiB apiece) filtered down to ~6,000 AUSA rows, and querying
  all of them monthly is both slow and enough to trip HuggingFace's rate
  limit on repeated large-file reads. `EMPLOYMENT_MONTH_STRIDE` in the
  config cell controls this — set it to 1 for full monthly resolution if
  you're willing to wait longer.

Four [great_tables](https://posit-dev.github.io/great-tables/) color-shaded
tables — one row per month (formatted "Mon YYYY", e.g. "Nov 2024" — the
raw `202411` YYYYMM string isn't readable), DC and rest-of-country as
separate columns (a matplotlib `imshow` heatmap was tried first and was
hard to read at this size): headcount, hires, separations, and net
(hires − separations). No Total column: rest-of-country outnumbers DC
~10:1, so a sum just mirrors rest-of-country's pattern almost exactly
without adding real signal. Column widths are set explicitly
(`cols_width`) instead of left to stretch to fill the browser/notebook
cell, which otherwise left large empty gutters in the % columns.

Color means the same thing everywhere in the notebook, not just within one
table: **red is always bad, blue is always good.** Hires are colored blue
(more = better); separations are colored red (more = worse) — using the
same "darker = more" scale for both, with no good/bad signal, would have
made identical colors mean opposite things in adjacent tables. Net hires
and the headcount % columns use a diverging red(loss)/blue(gain) scale
centered on zero or on the baseline. The net table is the most direct read
on workforce trajectory — it's what actually surfaces the 2025 attrition
spike (deep red at Jan 2025 and Sep 2025 in both areas) rather than a
hiring slowdown.

The headcount table additionally indexes each of DC/rest-of-country to the
first available month (Nov 2024, pre-inauguration) as a 100% baseline, so
loss reads as a percentage — by late 2025 DC and rest-of-country both sit
around 86–87% of their Nov 2024 headcount. The raw headcount columns there
are deliberately left uncolored: coloring both the raw counts (high = dark
= good) and the % columns (far from baseline = dark = bad) in the same row
told two contradictory stories with the same visual weight. Only the %
columns carry color, and both share one domain so DC's shade is directly
comparable to rest-of-country's rather than each being scaled
independently. The headcount and net tables use a spanner ("DC" / "Rest of
country") over paired Count/% columns — the sub-columns are labeled
"Count", not "DC"/"Rest of country" again, since the spanner already says
that and repeating it was redundant.

Accessions/separations are filtered to exactly Nov 2024–present: a file
named for one month can contain a late-processed correction with an older
effective date, and `query_monthly` filters on the data's own date column
(not just on which files get fetched) to keep those out.

`START_YM`/`END_YM` in the config cell are plain variables you can widen back
to 2005-01 (accessions/separations) or 2005-05 (employment) for full history.

## Every table stands alone

Each table's title states what it is, the cadence (monthly, or "every 3rd
available month (quarterly interval)" for headcount — a fixed interval,
not a statistical sample), and the career-attorneys-only scope; a source
note on every table states where the data comes from and that "DC /
rest-of-country" is the finest area breakdown available (not full
state-level detail — see the scope note above). None of this depends on
surrounding notebook text, so a screenshot of just one table is still
fully interpretable on its own.
