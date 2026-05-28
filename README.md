# 🏎️ Formula 1 Data Engineering Project
## The Art of Racing – Verstappen vs Norris (2025 Season)

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![SQL Server](https://img.shields.io/badge/SQL%20Server-2019-red.svg)](https://www.microsoft.com/en-us/sql-server)
[![Power BI](https://img.shields.io/badge/Power%20BI-F2C811.svg)](https://powerbi.microsoft.com/)
[![FastF1](https://img.shields.io/badge/FastF1-2.4-green.svg)](https://docs.fastf1.dev/)

---

## 📌 Project Overview

Formula 1 is one of the most data‑intensive sports. Each car generates **thousands of data points** per race – speed, tyre performance, throttle, brake, RPM, and more.

This project applies **Data Engineering principles** to build a complete ETL pipeline:

- **Extract** – Lap and telemetry data using the FastF1 API
- **Transform** – Clean, convert types, and build a star schema in SQL Server
- **Load** – Create an interactive Power BI dashboard

**Drivers analyzed:** Max Verstappen (Red Bull Racing) vs Lando Norris (McLaren) – 2025 season.

---

## 🎯 Project Objectives

- Collect and process F1 data from two races of the 2025 season
- Clean and transform raw CSV data into a structured star schema
- Link high‑frequency telemetry to lap‑level attributes
- Build an interactive dashboard to compare driver performance

---

## 🛠️ Technologies Used

| Tool | Purpose |
|------|---------|
| **FastF1** | Extract official F1 timing and telemetry data |
| **Python (Pandas)** | Data cleaning and CSV export |
| **SQL Server** | Staging, type conversion, star schema modelling |
| **Power BI** | Dashboard visualisation and KPI analysis |

---

## 📊 Database Schema (Star Schema)

![Star Schema Diagram](images/star_schema_diagram.png)

### Dimension Tables

| Table | Description |
|-------|-------------|
| `dim_driver` | Driver code, name, number, team |
| `dim_session` | Session type (FP1/Race), date, track status |
| `dim_tyre_stint` | Stint number, compound, tyre life, fresh tyre flag |
| `dim_pit` | Pit in/out times |
| `dim_track` | Track status, position, deleted lap flags |
| `dim_data_quality` | Data accuracy and source flags |

### Fact Tables

| Table | Rows | Description |
|-------|------|-------------|
| `laps_clean` | 232 | Lap‑level data (times, speeds, tyre life, stint) |
| `telemetry_clean` | 17,906 | High‑frequency car data (speed, throttle, brake, RPM, gear) |

---

## 🔄 ETL Pipeline Steps

### 1. Data Extraction (Python / FastF1)

```python
import fastf1
session = fastf1.get_session(2025, race_round, 'R')
session.load()
laps = session.laps
telemetry = laps.pick_fastest().get_telemetry()

Extract laps, results, telemetry, pit stops, weather, track status
Save raw data to CSV files

## 2. Staging & Type Conversion (SQL Server)

Created laps_staging and telemetry_staging with all NVARCHAR columns

Used BULK INSERT to load CSV files

Converted data types:

0 days HH:MM:SS.ms → seconds (FLOAT)

'1.0' → 1 (INT)

'True'/'False' → 1/0 (BIT)


## 3. Handling Missing Lap Numbers

Telemetry originally had no LapNumber. Solved using a time‑based join:
UPDATE t SET t.lap_number = l.lap_number
FROM telemetry_clean t
JOIN laps_clean l
  ON t.driver_code = l.driver_code
  AND t.session_type = l.session_type
  AND t.session_time_sec BETWEEN l.lap_start_time_sec 
                              AND l.lap_start_time_sec + l.lap_time_sec;


## 4. Building the Star Schema

Created 6 dimension tables from distinct values in laps_clean

Added foreign keys (driver_key, session_key, stint_key, etc.)

Populated all keys using joins

Applied FOREIGN KEY constraints for referential integrity


## 5. Power BI Dashboard

Overview Page – Key statistics and pipeline summary

Telemetry Comparison – Speed, throttle, brake traces for VER vs NOR

Laps Comparison – Lap time trends, sector analysis, tyre usage

Driver Head‑to‑Head – KPIs and final performance evaluation



📈 Key Results

Total laps processed	232
Total telemetry points	17,906
Dimension tables created	6
Foreign key constraints	13
Dashboard pages	4



Verification:

✅ All 232 laps have valid driver, session, stint, track, and quality keys

✅ All 17,906 telemetry rows have valid driver, session, and lap keys

✅ 12 laps have pit stop data (correct)
