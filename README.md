# India Smart City Analytics Pipeline

An end-to-end ETL pipeline that combines city demographics, infrastructure data, and live weather data for major Indian cities, transforms it into analytics-ready metrics, and loads it into a MySQL database.

---

## Overview

This project extracts data from three different sources, cleans and enriches it, merges everything into a single dataset, and loads the result into a MySQL table for downstream analysis (BI dashboards, reporting, etc.).

**Data sources:**
| Source | Format | Description |
|---|---|---|
| `City_Demographics.csv` | CSV | Population, area, literacy rate, hospitals, schools, GDP |
| `City_infrastructure.json` | JSON | Roads, metro lines, airports, railway stations, EV charging stations, smart city score |
| OpenWeatherMap API | REST API | Live temperature, humidity, and weather condition per city |

**Pipeline flow:**

```
Extract (CSV + JSON + Weather API)
        │
        ▼
Transform (clean → engineer features → merge → score)
        │
        ▼
Load (MySQL via SQLAlchemy)
```

---

## Features

- **Multi-source extraction** — reads local CSV/JSON files and calls a live weather API per city.
- **Data cleaning** — handles missing values in population, literacy rate, GDP, smart city score, and EV station counts.
- **Feature engineering**, including:
  - Population density (people/km²)
  - City tier classification (Tier 1 / 2 / 3)
  - Literacy, GDP, smart city, and EV readiness labels
  - Infrastructure score (weighted composite of metro, airports, railway stations, EV stations)
  - Weather-based livability scoring
  - Overall city score and city rank
  - Investment potential classification
- **Robust logging** — every stage (extract/transform/load) logs progress and errors to a `Logs` file.
- **Environment-variable driven config** — API keys and DB credentials are never hardcoded (via `.env` + `python-dotenv`).
- **MySQL loading** — final dataset is written to the `india_smart_city_analytics` table using SQLAlchemy.

---

## Tech Stack

- **Python 3**
- **pandas** — data manipulation
- **NumPy** — vectorized transformations
- **Requests** — weather API calls
- **SQLAlchemy** — MySQL connection and load
- **python-dotenv** — environment variable management
- **logging** — execution tracking

---

## Project Structure

```
├── Csv_File.py               # Generates City_Demographics.csv (synthetic demographic data)
├── Json_file.py               # Generates City_infrastructure.json (synthetic infrastructure data)
├── City_Demographics.csv      # Demographic dataset
├── City_infrastructure.json   # Infrastructure dataset
├── Pipeline.py                 # Main ETL pipeline (extract → transform → load)
├── Logs                        # Execution log file (auto-generated)
├── .env                        # Environment variables (not committed)
└── README.md
```

---

## Setup & Installation

### 1. Clone the repository
```bash
git clone https://github.com/SaurabhJadhav-sql/<repo-name>.git
cd <repo-name>
```

### 2. Install dependencies
```bash
pip install pandas numpy requests sqlalchemy python-dotenv pymysql
```

### 3. Configure environment variables
Create a `.env` file in the project root:

```env
API_KEY=your_openweathermap_api_key
WEATHER_URL=https://api.openweathermap.org/data/2.5/weather

SQL_CONN=mysql+pymysql://
DB_USER=your_db_username
DB_PASSWORD=your_db_password
DB_HOST=localhost
DB_PORT=3306
DB_NAME=your_database_name
```

### 4. Generate the source data (optional — files are already included)
```bash
python Csv_File.py
python Json_file.py
```

### 5. Run the pipeline
```bash
python Pipeline.py
```

The final enriched dataset will be loaded into the `india_smart_city_analytics` table in your MySQL database, and execution details will be recorded in `Logs`.

---

## Output

The pipeline produces a merged, analytics-ready table with columns including:

`City`, `State`, `Population_Millions`, `Area_Square_km2`, `Literacy_Rate`, `Hospitals`, `Schools`, `GDP_USD_Billions`, `Population_Density_per_km2`, `City_Tier`, `Literacy_Label`, `GDP_Category`, `smart_city_score`, `Smart_City_Label`, `Infra_Score`, `EV_Readiness`, `Temperature_C`, `Humidity_Percent`, `Condition`, `Weather_Score`, `Overall_City_Score`, `Livability_Score`, `investment_Potential`, `City_Rank`

---

## Known Limitations

- Weather data depends on live API availability — a failed request for a city is logged and that city is skipped rather than filled with defaults.
- One record in `City_infrastructure.json` (Nagpur) uses a mismatched key (`smart_city_analysis` instead of `smart_city_score`), which is treated as missing and filled with `0` during transformation.
- Demographic and infrastructure datasets are synthetic/sample data generated for portfolio purposes, not official government statistics.

---

## Future Improvements

- Add data validation/schema checks before load
- Parallelize weather API calls for faster extraction
- Add a PySpark implementation for larger-scale processing
- Build a dashboard (Power BI / Streamlit) on top of the MySQL table
- Add unit tests for transformation logic

---

## Author

**Saurabh Jadhav**
Data Engineering enthusiast | BCA Student
