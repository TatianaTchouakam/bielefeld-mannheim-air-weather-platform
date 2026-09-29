# Mannheim Air & Weather Data Platform

> 🚧 **Work in progress** – this README is updated as the project evolves.

A data engineering project that collects **air-quality** and **weather** data for **Mannheim, Germany** every day, stores it in a Parquet data lake, transforms it with **PySpark**, orchestrates everything with **Apache Airflow**, and runs on **Kubernetes**.

**Business question:** *How does the weather (wind, temperature, rain) influence air pollution (NO₂, PM10, O₃) in Mannheim?*

---

## Data sources (public, German open data)

| Source | Data | Access |
|---|---|---|
| Umweltbundesamt (UBA) | Air quality measurements (NO₂, PM10, PM2.5, O₃) from Mannheim stations | UBA Air Data API |
| Deutscher Wetterdienst (DWD) | Hourly weather observations for Mannheim (temperature, wind, precipitation) | DWD Open Data portal (opendata.dwd.de) |

## Architecture

```
UBA API ─┐
         ├─► Python ingestion ─► Bronze (Parquet) ─► PySpark ─► Silver (clean) ─► PySpark ─► Gold (air + weather)
DWD  ────┘                                                                                      │
                                                                                                ▼
                          Apache Airflow (daily DAG)  ·  running on Kubernetes (kind + Helm)   Dashboard
```

- **Bronze:** raw data exactly as received, partitioned by date
- **Silver:** cleaned, typed, deduplicated, quality-checked data
- **Gold:** air quality joined with weather, daily aggregates ready for analysis

## Tech stack

`Python` · `Apache Airflow` · `PySpark` · `Apache Parquet` · `Kubernetes` · `Helm` · `Docker` · `Streamlit` · `Git & GitHub`

## Project structure

```
ingestion/        Python scripts that call the UBA and DWD sources
transformations/  PySpark jobs (bronze → silver → gold)
dags/             Airflow DAGs
k8s/              Kubernetes manifests and Helm values
dashboard/        Streamlit dashboard
data/             Local data lake (bronze / silver / gold) – not committed
notebooks/        Exploration notebooks
docs/             Architecture notes and learning log
tests/            Unit and data-quality tests
```

## Learning log

I document what I learn at each step in [`docs/learning-log.md`](docs/learning-log.md).

---

**Author:** Tatiana Tchouakam Chouacheu · [LinkedIn](https://www.linkedin.com/in/tatiana-tchouakam-chouacheu-91152935b/) · [GitHub](https://github.com/TatianaTchouakam)

Data: © Umweltbundesamt, © Deutscher Wetterdienst (open data).
