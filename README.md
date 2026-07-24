# Brisbane Bus Service Equity Analysis

This project analyses **bus service equity across Brisbane suburbs** by combining population data with public transport supply.  
The goal is to identify **SA2 areas with high population but weak bus service access** to inform fairer bus investment decisions.

## Project Overview

- Study area: **Greater Brisbane (SA2 level)**
- Objective: Find suburbs where **demand (population density)** is high but **bus service supply** (stops and frequency) is low.
- Tools: **QGIS**, GroupStats, Data Plotly
- Data sources:
  - ABS Regional Population, SA1/SA2 (2025 projections)
  - GTFS timetable data (routes, trips, shapes, stop times)
  - Brisbane City Council bus stops dataset
  - OpenStreetMap basemap

## Methods

1. **Preprocessing and CRS**
   - Set project CRS to `GDA2020 / MGA Zone 56 (EPSG:7856)`.
   - Loaded Brisbane SA1/SA2 boundary layers and ABS population tables.
   - Joined population data to SA1 and SA2 polygons.

2. **Bus stops and routes**
   - Imported council **bus stops** as point data and reprojected to match the project CRS.
   - Imported GTFS `shapes.txt`, `trips.txt`, `routes.txt`, and `stop_times.txt`.
   - Reconstructed **bus route paths** from GTFS shapes and joined route + trip information.

3. **Filtering to Brisbane**
   - Intersected bus stops and routes with the Brisbane boundary to keep only **stops and services inside Brisbane**.
   - Created a cleaned “Brisbane bus stops” layer and “Brisbane routes” layer.

4. **Service frequency and stop density**
   - Aggregated `stop_times` by `stop_id` to compute **service frequency** per stop.
   - Joined stop frequencies back to the bus stops layer.
   - Summarised to SA1 and SA2 using spatial joins:
     - Total **stop count** per area.
     - Total **frequency** (departures) per area.
   - Created per‑capita and per‑area metrics:
     - `busstops_per_1000`
     - `busstops_per_km2`
     - `frequencies_per_1000`
     - `frequencies_per_km2`

5. **Demand-supply and service deficit**
   - Defined:
     - **Demand** = `population_density_2025 / frequencies_per_1000`
     - **Supply** = `frequencies_per_1000 / 1000`
     - **Service deficit** = `Demand - Supply`
   - Interpreted service deficit:
     - Positive → **under‑served** (demand > supply proxy)
     - Near zero → **balanced**
     - Negative → **well‑served**
   - Categorised each SA2 with:
     ```text
     deficit_category =
       CASE
         WHEN "service_deficit" IS NULL THEN 'No data'
         WHEN "service_deficit" < 0 THEN 'Well-served'
         WHEN "service_deficit" = 0 THEN 'Balanced'
         WHEN "service_deficit" > 0 THEN 'Under-served'
       END
     ```

6. **Quality checks**
   - Verified that total service frequencies matched across layers:
     - Sum of frequencies in SA1 = sum in SA2 = sum in Brisbane stops.
   - Checked distribution of `frequencies_per_1000` and treated extreme outliers by **not** standardising (avoided misleading z‑scores/min–max on a heavily skewed distribution).

## Results

- Produced a **Brisbane SA2 map** showing:
  - Population density classes.
  - Service deficit category (well‑served / balanced / under‑served).
- Generated a **priority list of under‑served SA2 suburbs** with high population and low bus service:
  - e.g. Kangaroo Point, Chambers Flat–Logan Reserve, Samford Valley, Morayfield–East, Doolandella, Yarrabilba, Wakerley, Ipswich–North, Bellbird Park, Spring Hill, Hawthorne, Hamilton (Qld), Birkdale.
- Final QGIS layout includes:
  - Main choropleth map
  - Service‑deficit legend
  - Side table listing top‑priority under‑served suburbs.

This project was developed as part of a data‑driven public transport equity analysis for Greater Brisbane.
