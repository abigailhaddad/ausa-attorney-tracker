# AUSA attorney tracker

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/abigailhaddad/ausa-attorney-tracker/blob/main/ausa_attorney_tracker.ipynb)

**Did the D.C. U.S. Attorney's Office lose a bigger share of its career
attorneys than the rest of the country did?**

Kinda. The public data has a "DC" bucket, and it's down about as much as
everywhere else — ~88% of its Nov 2024 headcount versus ~86% nationally, with
DC getting there first (92% by Feb 2025, when the rest of the country was
still at 98%). But "DC" is not the D.C. U.S. Attorney's Office, and the field
that would separate them is redacted. This notebook pushes the question as far
as the data goes and quantifies the gap left over.

Monthly career-attorney headcount (occupational series 0905, DOJ's Executive
Office for U.S. Attorneys and the Offices of the U.S. Attorneys), queried
**live** from the OPM/EHRI mirror on HuggingFace
([`impactproject/opm-ehri-data`](https://huggingface.co/datasets/impactproject/opm-ehri-data))
with DuckDB over HTTPS. Nothing downloaded to disk.

**Run it:** click the Colab badge, or open the notebook and run top to bottom.
Dependencies install themselves if missing.

## Why "DC" isn't the D.C. office

The DC bucket is everyone in this corner of DOJ whose duty station is
Washington, and two organizations fit that description: the U.S. Attorney's
Office for D.C., one of the 93 district offices, and EOUSA, the national
headquarters that oversees all 93 — also in Washington. They share one
`agency_subelement` label.

Part of headquarters is identifiable, and the notebook excludes it from every
table. Personnel office `4261` looks like HQ on three signals of unequal
strength:

- **pay plan (the strong one)** — 100% GS/ES/SL, where every other DC career
  attorney is 100% AD, DOJ's attorney schedule. Exceptionless across all 20
  months: 803 person-months here, none on AD; 8,833 for the rest of DC, none
  on anything else. Two pay systems that never mix are two organizations.
- **supervisory share** — 40.9% supervisor or manager vs. 12.4% for the rest
  of DC (Nov 2024).
- **geography (the weak one)** — 44 of the 61 people nationwide with this code
  are in DC; the other 17 are scattered 1–3 at a time across 13 states. A few
  sit in towns that plainly aren't district seats (Old Saybrook CT, Saratoga
  Springs NY, Collegeville PA), but most are in cities that *are* (Columbia
  SC, Houston, Chicago, Minneapolis). Mildly corroborating, not proof.

Excluding it moves DC's latest figure from 87.3% to 87.6% of baseline — which
is the point. The fixable part of the problem wasn't the problem:

| | Nov 2024 | Jun 2026 |
|---|---|---|
| DC, all career attorneys | 535 | 467 |
| − identified EOUSA HQ (personnel office 4261) | 44 | 37 |
| = DC, excluding identified HQ | 491 | 430 |
| − DOJ's published USAO-DC AUSA count | 330 | 330 |
| **= unexplained excess** | **161** | **100** |

DOJ's About page, verbatim: *"It is the largest United States Attorney's
Office with over 330 Assistant United States Attorneys and over 330 support
personnel"*
([justice.gov/usao-dc/about-us](https://www.justice.gov/usao-dc/about-us),
checked 2026-08-09).

The likeliest explanation for the leftover is that EOUSA headquarters employs
a substantial number of attorneys on the same AD schedule, in the same city,
under the same subelement, whose `personnel_office_identifier_code` is
redacted like everyone else's. They can't be pulled out because the leftover
group is *homogeneous* on everything the data exposes — 100% AD, 100% duty
city Washington, ~99% excepted service, split across the two ordinary AUSA
appointment codes. Personnel office `4261` announced itself by running on a
different pay system; these 491 leave no seam to cut along.

## What that costs you

The DC column is office + hidden headquarters and only the sum is published,
so the office's own trajectory isn't identified. Allocating the observed
decline three ways, all consistent with the published data:

| | Feb 2025 | Jun 2026 |
|---|---|---|
| Rest of country (observed) | 98% | 86% |
| DC excl. identified HQ (observed) | 92% | 88% |
| USAO-DC if the whole DC decline was the office | 88% | 82% |
| USAO-DC if hidden HQ shrank like the HQ we can see | 90% | 89% |
| USAO-DC if the whole DC decline was headquarters | 100% | 100% |

At **both** dates the rest-of-country number falls inside the range. The data
cannot establish that the D.C. office was hit harder than the rest of the
country, or that it wasn't.

"Over 330" is a floor, not a measurement — strictly, 491 is *consistent* with
it. So the ~160 is best read as the **largest** the hidden group can be while
DOJ's number holds, and the scenarios are deliberately built on 330 exactly:
the biggest hidden group gives the widest range, making this the most
conservative version of "you can't tell." If the office is really larger than
330, the range narrows and every scenario moves toward the office having
absorbed the decline itself.

The middle row is the most defensible single read — the one headquarters group
we *can* watch barely moved early (down ~4% by Feb 2025 vs. ~8% for the DC
bucket), so if the hidden attorneys behaved like the visible ones, most of the
early drop was the office. That's a lean resting on an assumption about a
group the data hides, not a measurement.

## Data notes

**Geography is DC vs. rest-of-country, and that's the ceiling.** Every
geographic field (`duty_station_state_abbreviation`, `duty_station_city`,
`core_based_statistical_area`) is redacted for ~91% of AUSA records — 5,849 of
6,400 in Dec 2024 — uniformly across the whole location hierarchy. DC is the
one exception. State- or district-level detail isn't in here to be had.

**Career attorneys only.** Three categories of political appointment appear in
this population and all three are excluded (`POLITICAL_APPOINTMENT_TYPES` in
the config cell): Schedule C, noncareer SES, and `EXECUTIVE (EXCEPTED SERVICE
NONPERMANENT)` — the last being consistently pay plan `AD`, grade `40`,
supervisor, i.e. the Presidentially-appointed U.S. Attorney slot. Schedule C
alone would have missed the other two, including the U.S. Attorney position
itself.

**`OTHER (EXCEPTED SERVICE NONPERMANENT)` stays in** despite sounding
temporary — it's 90–99.7% of AUSA hires every year 2015–2024 and 79.7% in
2025, i.e. the ordinary hiring code. It does collapse to 7.1% in 2026, so the
hiring authority for new AUSAs changed this year. That's a separate story this
notebook doesn't chase; it's flagged because it's the kind of break that
invalidates a rule of thumb quietly. It doesn't affect the headcount tables.

**Everything is queried live**, so figures may shift as OPM/EHRI publishes
revisions (files are versioned; the notebook takes the latest per month).
`count` is a string in the source parquet and is cast with `TRY_CAST`, so a
malformed value would under-count rather than crash.

## What's in the notebook

One live query covering Nov 2024 (pre-inauguration baseline) through present,
grouped by month and segment, feeding four
[great_tables](https://posit-dev.github.io/great-tables/) tables: the full
monthly headcount, a 4-row key-months version for screenshots, the
reconciliation above, and the scenarios above. Each table's title and source
note state its scope, so a screenshot of one is interpretable on its own.

`START_YM`/`END_YM` are plain variables you can widen back to `200503`, the
earliest employment snapshot in the mirror; raise `EMPLOYMENT_MONTH_STRIDE`
above 1 if you do, to keep the query fast. Design decisions in the tables
(fixed color domain, uncolored raw counts, explicit column widths) are
explained where they're made, in the notebook's own comments.
