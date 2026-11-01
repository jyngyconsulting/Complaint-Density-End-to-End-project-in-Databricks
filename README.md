# NYC 311 Complaint Density Index

A Databricks dashboard and gold data model that compare NYC 311 complaint volume with population, and show how weather-sensitive complaint types move alongside daily temperature.


## Question answered

How do complaint categories move with daily maximum temperature?
This project divides by population (complaints per 1,000 residents) so areas can be compared on rate, not volume.

## Data sources

| Source | Used for | Grain |
|---|---|---|
| NYC 311 Service Requests (NYC Open Data) | Complaint volume, type, location, date, response times | One row per request |
| NOAA weather | Daily max/min temperature, precipitation | One station, applied citywide, one row per day |
| Census.gov | Population by zip, by census vintage | Zip × vintage, with `valid_from` / `valid_to` |

## What was built

### Pipeline orchestration

Databricks job `00_seq_complaint_density_index` (Jobs & Pipelines). Five tasks:

| Task | Type | Feeds gold |
|---|---|---|
| `00_job_nyc_311` | Job task | Yes |
| `01_job_census` | Job task | Yes |
| `02_job_noaa_weather` | Job task | Yes |
| `00_gold_facts_dimensions` | Task at path `.../03_gold/gold_facts_dimensions`, serverless compute | Depends on the three tasks above |
| `03_job_housing_violation` | Job task | No dependency link. The task icon indicates it is disabled |

The three source tasks have no dependencies on each other. The gold task starts after all three finish.

<img width="1538" height="888" alt="population_orchestration" src="https://github.com/user-attachments/assets/191a3b09-7386-41f2-ab3f-a494c3f7763c" />


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

### 
### 
### 
### Dashboard: "2024 Complaint Density Index Report"

Two tabs.

**Citywide**

| Visual | Content |
|---|---|
| Total complaints | 1.64M |
| Total complaints by month | Line, 2024-02 to 2024-07 |
| Month-over-month complaints | Bar: 2024-03 14.9, 2024-04 0.63, 2024-05 6.21, 2024-06 8.02, 2024-07 -2.37 |
| Highest complaint rate per capita | Bar by borough, complaints per 1,000 residents |

###
###

<img width="1534" height="885" alt="citywide" src="https://github.com/user-attachments/assets/ef221a09-0ab1-43e1-94bf-aabeb6534f43" />

###
### 
### 

**Complaint Patterns by Weather Conditions**

| Visual | Content |
|---|---|
| Filters | Month, Borough, Complaint category |
| Complaint categories to temperature spread | Scatter: daily max temperature (°C) vs number of complaints; categories Heat/Hot Water, Noise, Other, Water/Flooding |
| Complaint seasonality | Heatmap: correlation (r) of max temperature vs daily complaints, by category and season (Spring, Summer, Winter) |

###
###
###

<img width="1469" height="874" alt="complaint_patterns_by_weather_conditions" src="https://github.com/user-attachments/assets/40e99735-779d-408f-b9c4-6939c6249c21" />

###
###
###


