# AUSA attorney tracker

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/abigailhaddad/ausa-attorney-tracker/blob/main/ausa_attorney_tracker.ipynb)

One notebook. Monthly career-attorney headcount for Assistant U.S.
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

The obvious next step — breaking AUSA headcount down by state or district — isn't
possible with this data. Every geographic field (`duty_station_state_abbreviation`,
`duty_station_city`, `core_based_statistical_area`) is privacy-redacted for
**~91% of AUSA records** (checked directly: 5,849 of 6,400 employment records,
Dec 2024) — a small-occupational-subgroup suppression rule applied uniformly
across the whole location hierarchy, not specific to any one field. DC is the
one exception: its ~500-attorney cell is large enough to clear the suppression
threshold and reports a real number. So the only two honest "area" buckets are
**DC** and **rest-of-country (aggregate)** — the notebook does not fabricate
state-level detail the underlying data doesn't actually contain.

## "DC" is not exactly "the DC U.S. Attorney's Office"

DOJ's own page for USAO-DC cites ~330-350 AUSAs — well below this
notebook's DC count (467-538 across the window). `agency_subelement`
combines EOUSA's national headquarters (physically based in DC) with all
93 individual U.S. Attorney's offices as one label; EHRI doesn't track
them separately.

Checked directly: at least 44 of DC's 538 (Nov 2024) sit in a structurally
distinct group —

- `personnel_office_identifier_code = 4261`: mostly DC (44 of 48
  nationwide), the rest scattered 1-3 people at a time in cities that
  aren't typical USAO district seats (Old Saybrook CT, Saratoga Springs
  NY, Anchorage AK) — the pattern you'd expect from a centralized HQ
  personnel office, not a normal field office
- **GS-graded** (13-15), not DOJ's usual attorney AD pay scale (21-40)
  that the other 494 use
- **~40% supervisor/manager**, vs. ~13% in the rest

All consistent with EOUSA HQ staff, not USAO-DC prosecutors. The remaining
~494 can't be split further with available fields — the finer
office-identifier is redacted for that whole group — so the true HQ
contribution, and therefore how much of "DC" is really USAO-DC, is
bounded (at least 44) but not fully known. Read "DC" as DC-based
EOUSA/USAO staff broadly, not literally the DC U.S. Attorney's Office
headcount.

## Scope: career attorneys only, not political appointees

`appointment_type` is used to exclude political appointments from the
query, so headcount figures track the career AUSA workforce rather than
political leadership turnover. Three distinct legal categories of
political appointment show up in this data, and all three are excluded
(`POLITICAL_APPOINTMENT_TYPES` in the config cell):

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

One live query, grouped by month and by area (DC / rest-of-country),
covering **Nov 2024 (pre-inauguration baseline) through present** —
`EMPLOYMENT_MONTH_STRIDE = 1` pulls every available month by default.
Employment snapshots are the full federal workforce each month (26–75 MiB
apiece) filtered down to ~6,000 AUSA rows; within this ~20-month window
that's still fast (~20s) and doesn't trip HuggingFace's rate limit on
repeated large-file reads (that only happened pulling the full 2015-present
history, ~140 files, in an earlier version of this notebook — raise the
stride above 1 if you widen `START_YM`/`END_YM` back that far).

One [great_tables](https://posit-dev.github.io/great-tables/) color-shaded
table (a matplotlib `imshow` heatmap was tried first and was hard to read
at this size) — one row per month (formatted "Mon YYYY", e.g. "Nov 2024" —
the raw `202411` YYYYMM string isn't readable), DC and rest-of-country as
separate columns, each also indexed to the Nov 2024 baseline (=100%) so
loss reads as a percentage. No Total column: rest-of-country outnumbers DC
~10:1, so a sum just mirrors rest-of-country's pattern almost exactly
without adding real signal. Column widths are set explicitly (`cols_width`)
instead of left to stretch to fill the browser/notebook cell, which
otherwise left large empty gutters in the % columns.

Raw headcount columns are deliberately left uncolored: coloring both the
raw counts (high = dark = good) and the % columns (far from baseline =
dark = bad) in the same row would tell two contradictory stories with the
same visual weight. Only the % columns carry color — a diverging
red(loss)/blue(gain) scale, both areas sharing ONE fixed domain (`[75,
125]`, not derived from this window's own min/max — a domain scaled to
just this window's extremes made the worst month always look maximally
saturated no matter how mild the real swing was) — so DC's shade is
directly comparable to rest-of-country's, and the color intensity means
the same thing across different runs of this notebook. A spanner ("DC" /
"Rest of country") sits over each paired Count/% column; the sub-columns
are labeled "Count", not "DC"/"Rest of country" again, since the spanner
already says that.

By late 2025 both DC and rest-of-country sit around 86–89% of their Nov
2024 headcount — DC took a sharper early hit (92% by Feb 2025, vs. 98% for
rest-of-country that same month) that the rest of the country then caught
up to, not the other way around.

`START_YM`/`END_YM` in the config cell are plain variables you can widen
back to 2005-05 for full employment history.

A second, 4-row **key-months** table follows the full one — same styling,
just filtered down to the baseline month, Feb 2025 (DC's sharp early
drop), Sep 2025 (the shared national low both areas converged toward), and
whatever the latest available month is. Meant for a screenshot where the
full monthly table is too tall (e.g. a social post); the last of the four
months updates itself as new data arrives (`df_employment.ym.max()`) so it
doesn't need editing by hand.

## The tables stand alone

Each one's title states what it is, the cadence ("Every available month",
"Every N months" if you raise the stride, or "4 key months" — never a
statistical sample), and the career-attorneys-only scope; a source note
states where the data comes from and that "DC / rest-of-country" is the
finest area breakdown available (not full state-level detail — see the
scope note above). None of this depends on surrounding notebook text, so a
screenshot of just one table is still fully interpretable on its own.
