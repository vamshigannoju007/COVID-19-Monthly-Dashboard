# COVID-19 Monthly Dashboard (PostgreSQL → Power BI)

A monthly COVID-19 dashboard built end-to-end using PostgreSQL for data preparation and Power BI for reporting.  
It supports entity-level analysis (countries + World), time filtering (Year/Quarter/Month), and comparisons like Top/Bottom countries.

---

## What this project shows

### Main questions answered
- How did monthly **new cases** and **new deaths** change over time?
- What are the **month-end total cases** and **total deaths** for an entity?
- How did **vaccination coverage** (one dose / complete dose) progress over time?
- How did the vaccination effect the covid cases over time?

### Key outputs
- Interactive dashboard pages:
  - Cover
  - Trends (cases/deaths)
  - Vaccination
  - Information / Notes

---

## Data

### Tables used (PostgreSQL)
- `covid_cumulative_abs`  
  Daily cumulative confirmed cases and deaths by entity.
- `covid_vaccines`  
  Daily vaccination metrics by entity (one dose, complete dose, etc.).

### Grain used in the dashboard
Monthly (one row per **entity + month**).

---

## Core logic (important)

### Month-end snapshot (vaccination)
Vaccination fields are cumulative. They must be taken as a **month-end value**, not summed.

I take the last available record in each month:
- `DISTINCT ON (entity, month_)`
- `ORDER BY entity, month_, date_ DESC`

### Monthly new values (cases/deaths)
Cases/deaths are cumulative in the source. Monthly “new” values are derived as:

- `new_cases = total_cases - LAG(total_cases)`
- `new_deaths = total_deaths - LAG(total_deaths)`

### Death percentage used
I use cumulative CFR to avoid unstable monthly spikes:

- `death_pct = total_deaths / total_cases * 100`

---

## SQL views (what Power BI imports)

### 1) `covid_abs`
Monthly cases/deaths view derived from cumulative data:
- `entity, month_`
- `total_cases, total_deaths`
- `new_cases, new_deaths`

### 2) `covid_abs_all` (final model table)
Joined monthly cases/deaths with month-end vaccination snapshot:
- `entity, month_`
- `total_cases, total_deaths`
- `new_cases, new_deaths`
- `death_pct`
- `one_dose, complete_dose`

This is the main table used in Power BI visuals and slicers.

---

## Power BI model setup

### Relationships
Create a Date table (or Month table) and relate it to `covid_abs_all`:

- `DateTable[MonthStart]` → `covid_abs_all[month_]`

Cross filter direction: **Single**

### Slicers (recommended)
- Entity (dropdown): `covid_abs_all[entity]`
- Year: `DateTable[Year]`
- Quarter: `DateTable[Quarter]`
- Month: `DateTable[MonthYear]`

### Visual aggregation rules (don’t break these)
- Cumulative columns (`total_cases`, `total_deaths`, `one_dose`, `complete_dose`) → use **MAX**
- Monthly increment columns (`new_cases`, `new_deaths`) → use **SUM**

---

## Dashboard pages (suggested)

### Cover
- Title + subtitle (date range, grain)
- 3–5 KPI cards (as-of last month in filter)
- Navigation buttons to other pages

### Trends
- New cases (monthly) line/column
- New deaths (monthly) line/column
- Optional: total cases/total deaths (month-end) line

### Vaccination
- One dose vs complete dose (month-end) line chart
- Optional: compare with new cases on a combo chart



### Information
- Definitions, refresh instructions

---

## Refresh workflow

### When SQL changes
1. Update the view in PostgreSQL (use `CREATE OR REPLACE VIEW ...`).
2. In Power BI Desktop: **Refresh**.
3. If published: refresh dataset in the Power BI Service.

---

## Common pitfalls (what this project fixed)
- Summing cumulative values (wrong totals)
- Random monthly snapshot dates due to incorrect `DISTINCT ON` ordering
- Flat lines in charts caused by missing relationships / wrong axis fields
- Aggregates (continents/income groups) contaminating “Top countries” lists


---

## Author
Vamshi Gannoju

