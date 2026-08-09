# AUSA attorney tracker

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/abigailhaddad/ausa-attorney-tracker/blob/main/ausa_attorney_tracker.ipynb)

**The question:** did the D.C. U.S. Attorney's Office lose a bigger share of
its career attorneys than the rest of the country did?

**The answer: kinda.** The public data has a "DC" bucket, and that bucket is
down about as much as everywhere else — ~88% of its Nov 2024 headcount versus
~86% for the rest of the country. But "DC" is not the D.C. U.S. Attorney's
Office. It's at least two different groups of attorneys added together, and
the field that would separate them is redacted. This notebook pushes the
question as far as the data goes, then quantifies the gap that's left.

One notebook. Monthly career-attorney headcount for Assistant U.S. Attorneys
(DOJ, Executive Office for U.S. Attorneys and the Offices of the U.S.
Attorneys, occupational series 0905), queried **live** from the public
OPM/EHRI mirror on HuggingFace
([`impactproject/opm-ehri-data`](https://huggingface.co/datasets/impactproject/opm-ehri-data))
with DuckDB over HTTPS. Nothing is downloaded to disk.

**Run it:** click the *Open in Colab* badge, or open
[`ausa_attorney_tracker.ipynb`](ausa_attorney_tracker.ipynb) and run top to
bottom. Dependencies (`duckdb`, `pandas`, `great_tables`) are installed by the
notebook if missing.

## The finding, and its limit

By mid-2026 both DC and the rest of the country sit around 86–88% of their
Nov 2024 career-attorney headcount. DC got there first: it was at 92% by Feb
2025 while the rest of the country was still at 98%, and the rest of the
country caught down to it over the following year rather than the reverse.

That much is solid. What isn't solid is attributing any of it to the D.C.
U.S. Attorney's Office specifically — because of how the data is structured,
not because of anything about the analysis.

## Why "DC" is two groups

The DC bucket is everyone in this corner of DOJ whose duty station is
Washington — and two different organizations fit that description. One is the
U.S. Attorney's Office for D.C., an ordinary district office that happens to
be in the capital. The other is EOUSA, the national headquarters that oversees
all 93 district offices, also in Washington. In this data they share a single
`agency_subelement` label, and EHRI doesn't track them separately.

Part of headquarters *is* identifiable, and the notebook excludes it from
every table. Personnel office `4261` looks like HQ rather than a field
office, on three signals of unequal strength:

- **pay plan (the strong one)** — 100% GS/ES/SL, where every other DC career
  attorney is 100% AD, DOJ's attorney schedule. Exceptionless across all 20
  months: 803 person-months in that personnel office, none of them AD; 8,833
  for the rest of DC, none of them anything but AD. Two pay systems that
  never mix are two different organizations.
- **supervisory share** — 40.9% supervisor or manager, versus 12.4% for the
  rest of DC (Nov 2024). Top-heavy the way a headquarters is.
- **geography (the weak one)** — 44 of the 61 people nationwide with this
  code are in DC; the other 17 are scattered 1–3 at a time across 13 states.
  A few of those are in towns that plainly aren't district seats (Old Saybrook
  CT, Saratoga Springs NY, Collegeville PA), but most are in cities that *are*
  district seats or major USAO locations (Columbia SC, Houston, Chicago,
  Minneapolis). Read this as mildly corroborating, not as proof.

Excluding it doesn't fix the problem. It moves DC's latest figure from 87.3%
to 87.6% of baseline, and leaves this:

| | Nov 2024 | Jun 2026 |
|---|---|---|
| DC, all career attorneys (series 0905) | 535 | 467 |
| − identified EOUSA HQ (personnel office 4261) | 44 | 37 |
| = DC, excluding identified HQ | 491 | 430 |
| − DOJ's published USAO-DC AUSA count | 330 | 330 |
| **= unexplained excess** | **161** | **100** |

DOJ's own About page for the office, verbatim: *"It is the largest United
States Attorney's Office with over 330 Assistant United States Attorneys and
over 330 support personnel"*
([justice.gov/usao-dc/about-us](https://www.justice.gov/usao-dc/about-us),
checked 2026-08-09 — undated boilerplate, treated as a rough floor).

So on DOJ's own published floor, ~160 career attorneys in the DC bucket are
unaccounted for. (That floor is doing real work here, and it's worth being
precise about how much — see the caveat under "What that costs you.") The
likeliest explanation is that EOUSA headquarters employs a substantial number
of attorneys on the same AD attorney schedule, in the same city, under the
same `agency_subelement`, whose `personnel_office_identifier_code` is redacted
along with everyone else's.

They can't be pulled out because the leftover group is *homogeneous* on
everything this data exposes. Personnel office `4261` announced itself by
running on a different pay system; these 491 people are 100% AD pay plan, 100%
duty city Washington, ~99% excepted service, and split across the two ordinary
AUSA appointment codes. There's no seam to cut along.

## What that costs you

The DC column is office + hidden headquarters, and only the sum is published,
so the office's own trajectory isn't identified. The notebook's last table
allocates the observed decline three ways, all arithmetically consistent with
the published data:

| | Feb 2025 | Jun 2026 |
|---|---|---|
| Rest of country (observed) | 98% | 86% |
| DC excl. identified HQ (observed) | 92% | 88% |
| USAO-DC if the whole DC decline was the office | 88% | 82% |
| USAO-DC if hidden HQ shrank like the HQ we can see | 90% | 89% |
| USAO-DC if the whole DC decline was headquarters | 100% | 100% |

At **both** dates the rest-of-country number falls inside the range of
possible USAO-DC values. The data cannot establish that the D.C. office was
hit harder than the rest of the country, or that it wasn't.

Note that "over 330" is a floor, not a measurement — strictly, 491 is
*consistent* with "over 330." So the ~160 above is best read as the
**largest** the hidden headquarters group can be while DOJ's own number holds,
and the scenario table is deliberately built on 330 exactly: the biggest
possible hidden group gives the widest possible range, so this is the most
conservative version of "you can't tell." If the D.C. office is really larger
than 330, the hidden group shrinks, the range narrows, and every scenario
moves toward the office having absorbed the decline itself.

The middle row is the most defensible single read: the one headquarters group
we *can* watch barely moved early (down ~4% by Feb 2025, versus ~8% for the DC
bucket overall), so if the hidden headquarters attorneys behaved like the
visible ones, most of that early drop was the office itself. That's a lean
resting on an assumption about a group the data hides — not a measurement.

## Scope: DC vs. rest-of-country, not state-by-state

Breaking AUSA headcount down by state or district isn't possible here. Every
geographic field (`duty_station_state_abbreviation`, `duty_station_city`,
`core_based_statistical_area`) is privacy-redacted for **~91% of AUSA
records** (checked directly: 5,849 of 6,400 employment records, Dec 2024) — a
suppression rule applied uniformly across the whole location hierarchy, not
specific to any one field. DC is the one exception. So the only two honest
"area" buckets are **DC** and **rest-of-country (aggregate)** — the notebook
does not fabricate state-level detail the underlying data doesn't contain.

## Scope: career attorneys only, not political appointees

`appointment_type` is used to exclude political appointments from the query,
so headcount figures track the career AUSA workforce rather than political
leadership turnover. Three distinct legal categories of political appointment
show up in this data, and all three are excluded
(`POLITICAL_APPOINTMENT_TYPES` in the config cell):

- **Schedule C** (5 CFR 213.3301) — confidential/policy staff
- **Noncareer SES** — senior-executive-level political appointees
- **`EXECUTIVE (EXCEPTED SERVICE NONPERMANENT)`** — always pay plan `AD`,
  grade `40`, "supervisor or manager," one single consistent combination —
  consistent with the U.S. Attorney / top leadership slot per district
  (Presidentially-appointed, Senate-confirmed), not a career classification

Schedule C alone would miss the other two — it's only the "junior" tier of
political appointment, not an umbrella term for all of it, and it would
specifically miss the U.S. Attorney position itself, arguably the single most
obviously political role in the office.

One value that stays IN despite sounding temporary:
`OTHER (EXCEPTED SERVICE NONPERMANENT)` is the ordinary way AUSAs get hired,
not a marker of temporary or surge staffing. Checked against the accessions
data back to 2015: it accounts for 90–99.7% of AUSA hires in every year from
2015 through 2024, and 79.7% in 2025. Excluding it would have thrown out
nearly all AUSA hiring on record for no real signal.

It does collapse in 2026 — just 7.1% of hires so far — which means the hiring
authority used for new AUSAs changed this year. That's a separate story and
this notebook doesn't chase it; it's flagged here because it's the kind of
break that invalidates a rule of thumb quietly. It doesn't affect the
headcount tables, which count everyone regardless of appointment code except
the three political categories above.

## What's in the notebook

One live query, grouped by month and by segment (DC excluding identified HQ /
DC identified HQ / rest-of-country), covering **Nov 2024 (pre-inauguration
baseline) through present** — `EMPLOYMENT_MONTH_STRIDE = 1` pulls every
available month by default. Employment snapshots are the full federal
workforce each month (26–75 MiB apiece) filtered down to ~6,000 AUSA rows;
within this ~20-month window that's still fast (~20s) and doesn't trip
HuggingFace's rate limit on repeated large-file reads (that only happened
pulling the full 2015-present history, ~140 files, in an earlier version —
raise the stride above 1 if you widen `START_YM`/`END_YM` back that far).

Four [great_tables](https://posit-dev.github.io/great-tables/) tables (a
matplotlib `imshow` heatmap was tried first and was hard to read at this
size):

1. **The monthly headcount table** — one row per month (formatted "Mon YYYY";
   the raw `202411` YYYYMM string isn't readable), DC and rest-of-country as
   separate columns, each also indexed to the Nov 2024 baseline (=100%) so
   loss reads as a percentage. No Total column: rest-of-country outnumbers DC
   ~10:1, so a sum just mirrors rest-of-country's pattern without adding real
   signal.
2. **A 4-row key-months version** of the same table — baseline, Feb 2025 (DC's
   sharp early drop), Sep 2025 (the shared national low both areas converged
   toward), and whatever the latest available month is. Meant for a screenshot
   where the full table is too tall; the last of the four updates itself as
   new data arrives (`df_employment.ym.max()`), no hand-editing.
3. **The reconciliation table** — the walk-down from the raw DC bucket to the
   unexplained excess, one subtraction per row so it's visible which step
   closes the gap and which doesn't.
4. **The scenario table** — three allocations of the observed DC decline
   between the office and the hidden headquarters group.

Raw headcount columns are deliberately left uncolored: coloring both the raw
counts (high = dark = good) and the % columns (far from baseline = dark = bad)
in the same row would tell two contradictory stories with the same visual
weight. Only the % columns carry color — a diverging red(loss)/blue(gain)
scale, both areas sharing ONE fixed domain (`[75, 125]`, not derived from this
window's own min/max — a domain scaled to just this window's extremes made the
worst month always look maximally saturated no matter how mild the real swing
was) — so DC's shade is directly comparable to rest-of-country's, and color
intensity means the same thing across runs. A spanner sits over each paired
Count/% column; the sub-columns are labeled "Count", not repeating the area
name the spanner already gives.

Column widths are set explicitly (`cols_width`) instead of left to stretch and
fill the browser/notebook cell, which otherwise left large empty gutters.

`START_YM`/`END_YM` in the config cell are plain variables you can widen back
to `200503`, the earliest employment snapshot in the mirror.

## The tables stand alone

Each one's title states what it is, the cadence ("Every available month",
"Every N months" if you raise the stride, or "4 key months" — never a
statistical sample), and the career-attorneys-only scope; a source note states
where the data comes from, that "DC / rest-of-country" is the finest area
breakdown available, and that "DC" still mixes headquarters with the D.C.
office beyond the personnel office excluded. None of this depends on
surrounding notebook text, so a screenshot of one table is fully
interpretable on its own.
