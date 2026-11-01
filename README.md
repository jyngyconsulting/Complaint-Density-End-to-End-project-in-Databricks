# NYC 311 Complaint Density Index

A Databricks dashboard and gold data model that compare NYC 311 complaint volume with population, and show how weather-sensitive complaint types move alongside daily temperature.

<img src="docs/images/citywide-overview.png" alt="Citywide overview" width="800">

## Question answered

How do complaint categories move with daily maximum temperature?

Raw complaint counts reward large populations. This project divides by population (complaints per 1,000 residents) so areas can be compared on rate, not volume.

## Data sources

| Source | Used for | Grain |
|---|---|---|
| NYC 311 Service Requests (NYC Open Data) | Complaint volume, type, location, date, response times | One row per request |
| NOAA weather | Daily max/min temperature, precipitation | One station, applied citywide, one row per day |
| Census.gov | Population by zip, by census vintage | Zip × vintage, with `valid_from` / `valid_to` |

## What was built

### Gold data model

Catalog `nyc_complaint_density_index`, schema `03_gold`.

| Table | Grain | Key |
|---|---|---|
| `fact_complaints_daily` | date × zip × complaint type | `date_id`, `zip_id`, `complaint_type_id` |
| `dim_date` | one row per calendar day | `date_id` |
| `dim_zip` | zip_code × district | `zip_id` |
| `dim_complaint_type` | complaint type | `complaint_type_id` |
| `dim_population` | zip × census vintage | `population_id` |

Fact measures: `complaints`, `closures_valid`, `closed_count_hours`, `sum_response_hours`, `closed_count_days`, `sum_response_days`.

Weather columns on `dim_date`: `tmax_c`, `tmin_c`, `prcp_mm`, `hdd` (°F degree-days, base 65°F), `precip_day_flag` (prcp_mm ≥ 0.254), `freeze_flag` (tmin ≤ 32°F).

### Dashboard: "2024 Complaint Density Index Report"

Two tabs.

**Citywide**

| Visual | Content |
|---|---|
| Total complaints | 1.64M |
| Total complaints by month | Line, 2024-02 to 2024-07 |
| Month-over-month complaints | Bar: 2024-03 14.9, 2024-04 0.63, 2024-05 6.21, 2024-06 8.02, 2024-07 -2.37 |
| Highest complaint rate per capita | Bar by borough, complaints per 1,000 residents |

Complaints per 1,000 residents by borough (read from the chart, approximate):

| Borough | Per 1,000 |
|---|---|
| Bronx | 227 |
| Manhattan | 212 |
| Brooklyn | 191 |
| Queens | 169 |
| Staten Island | 122 |

<img src="docs/images/citywide-per-capita.png" alt="Complaints per 1,000 residents by borough" width="800">

**Complaint Patterns by Weather Conditions**

| Visual | Content |
|---|---|
| Filters | Month, Borough, Complaint category |
| Complaint categories to temperature spread | Scatter: daily max temperature (°C) vs number of complaints; categories Heat/Hot Water, Noise, Other, Water/Flooding |
| Complaint seasonality | Heatmap: correlation (r) of max temperature vs daily complaints, by category and season (Spring, Summer, Winter) |

<img src="docs/images/weather-context.png" alt="Weather context tab" width="800">

## Reading the results

- **Per-capita ranking.** The Bronx is highest, followed by Manhattan and Brooklyn. Staten Island is lowest.
- **Volume trend.** Monthly complaints rise from February to a peak in June, then fall in July (-2.37 month-over-month).
- **Weather views show association only.** They do not show causation. Weather is one station applied to the whole city, so it cannot explain differences between areas on the same day.

## Known limitations

- Dashboard data covers 2024-02 to 2024-07, six months, although the title reads "2024".
- The per-capita view is at borough level. Zip-level rates are not shown on the dashboard.
- The heatmap shows three seasons. If seasons are meteorological, Winter is February 2024 only.
- The heatmap does not respond to the Month filter. It shows Spring and Summer while Month is set to 2024-02.
- The heatmap and scatter do not display the correlation coefficient or the number of days behind it.
- The month-over-month chart has no unit label, and its axis label reads "Compaints rate".
- Population is never summed across fact rows. It is a period estimate from annual census vintages.

## Open items

- Confirm whether the gold-layer fixes in the BI specification (population key on the fact, key and total reconciliation, complaint grouping, population coverage) were applied. The specification lists them as required before the build.
- Zip-level density, service performance (response times) and a methodology page are specified but are not on the dashboard.
- Pipeline orchestration, scheduling and test results: add from the repository.

## Repository structure

TODO: complete from repository contents.

## How to run

TODO: complete from repository contents.

## Documents

- `docs/nyc-pipeline-brd.md`: Business Requirements Document v1.3 (2026-10-03)
- `docs/NYC311_Complaint_Density_Index___BI_Report_Specification.docx`: BI Report Specification v1.0 (2026-10-21)
- `docs/images/`: dashboard screenshots taken 2026-11-01
