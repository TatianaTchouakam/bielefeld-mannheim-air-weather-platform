# Air & Weather Data Platform – Bielefeld & Mannheim

> 🚧 **Work in progress** – this README is updated as the project evolves.

A data engineering project that collects **air-quality** and **weather** data for **Bielefeld** and **Mannheim** (Germany) every day, stores it in a Parquet data lake, transforms it with **PySpark**, orchestrates everything with **Apache Airflow**, and runs on **Kubernetes**.

The pipeline is **configurable by city**: new cities can be added by changing a single config file.

**Business question:** *How does the weather (wind, temperature, rain) influence air pollution (NO₂, PM10, O₃), and how do Bielefeld and Mannheim compare?*

---

## Data sources (public, German open data)

| Source | Data | Access |
|---|---|---|
| Umweltbundesamt (UBA) | Air quality measurements (NO₂, PM10, PM2.5, O₃) from Bielefeld and Mannheim stations | UBA Air Data API |
| Deutscher Wetterdienst (DWD) | Hourly weather observations (temperature, wind, precipitation) | DWD Open Data portal (opendata.dwd.de) |

## Configuration

Cities and their measuring stations are defined in `config/cities.yaml`:

```yaml
default_city: bielefeld

cities:
  bielefeld:
    uba_stations: []        # to be confirmed
    dwd_station_id: ""      # to be confirmed
    timezone: Europe/Berlin
  mannheim:
    uba_stations: []        # to be confirmed
    dwd_station_id: ""      # to be confirmed
    timezone: Europe/Berlin
```

Adding a new city = adding a new entry to this file. No code changes needed.

## Architecture

```
UBA API ─┐
         ├─► Python ingestion ─► Bronze (Parquet) ─► PySpark ─► Silver (clean) ─► PySpark ─► Gold (air + weather)
DWD  ────┘          ▲                                                                           │
                    │                                                                           ▼
           config/cities.yaml      Apache Airflow (daily DAG) · Kubernetes (kind + Helm)    Dashboard
```

- **Bronze:** raw data exactly as received, partitioned by city and date
- **Silver:** cleaned, typed, deduplicated, quality-checked data
- **Gold:** air quality joined with weather, daily aggregates ready for analysis and city comparison

### Target architecture (planned)

```
UBA / DWD ─► Python producers ─► Kafka (Avro + Schema Registry) ─► Consumers ─► Iceberg tables on MinIO (S3)
                                                                                     │
                                       Airflow (batch) ─► PySpark / dbt ─► Gold ─► Dashboard
                                                                                     │
                        Kubernetes + Helm · Terraform · Prometheus/Grafana · OpenLineage/Marquez
```

## Tech stack

**Current:** `Python` · `Apache Airflow` · `PySpark` · `Apache Parquet` · `Kubernetes` · `Helm` · `Docker` · `Streamlit` · `Git & GitHub`

**Planned:** `Kafka (Redpanda)` · `Avro` · `Schema Registry` · `Apache Iceberg` · `MinIO` · `dbt` · `Terraform` · `GitHub Actions` · `Prometheus` · `Grafana` · `OpenLineage` · `Marquez`

## Project structure

```
config/           City and station configuration (cities.yaml)
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

## Roadmap

### Phase 1 – Batch pipeline (core, Bielefeld first)
- [ ] Identify UBA and DWD stations for Bielefeld and fill `config/cities.yaml`
- [ ] Ingestion scripts for UBA and DWD (city read from config)
- [ ] PySpark jobs: bronze → silver → gold
- [ ] Daily Airflow DAG
- [ ] Deployment on Kubernetes (kind + Helm)
- [ ] Streamlit dashboard
- [ ] Unit and data-quality tests

### Phase 2 – Streaming ingestion
- [ ] Run Kafka (Redpanda) locally with Docker Compose
- [ ] Define Avro schemas for air-quality and weather events
- [ ] Set up Schema Registry and test schema evolution (e.g. add an optional field)
- [ ] Python producer (API → Kafka) and consumer (Kafka → bronze)

### Phase 3 – Lakehouse storage
- [ ] Store bronze/silver/gold as Apache Iceberg tables on MinIO (S3-compatible)
- [ ] Partitioning by city, date and station

### Phase 4 – Transformation & orchestration
- [ ] Airflow best practices: retries, backfills, SLAs
- [ ] dbt models and tests on the gold layer (freshness, not_null, unique)
- [ ] Optional: load gold data into a warehouse (DuckDB locally, Redshift Serverless on AWS)

### Phase 5 – Platform & infrastructure
- [ ] Infrastructure as Code with Terraform
- [ ] Secrets management (Kubernetes Secrets / Vault)
- [ ] CI/CD with GitHub Actions (lint, tests, dbt build)

### Phase 6 – Reliability & governance
- [ ] Observability: Prometheus + Grafana dashboards
- [ ] Define one SLO (e.g. data freshness < 1 hour) with an alert
- [ ] Write a runbook for a pipeline failure
- [ ] Data lineage with OpenLineage + Marquez
- [ ] Data catalog and ownership documentation

### Phase 7 – City comparison
- [ ] Add Mannheim via config only (no code changes)
- [ ] Compare air quality and weather effects between Bielefeld and Mannheim
- [ ] Comparison view in the Streamlit dashboard

## Learning log

I document what I learn at each step in [`docs/learning-log.md`](docs/learning-log.md).

---

**Author:** Tatiana Tchouakam Chouacheu · [LinkedIn](https://www.linkedin.com/in/tatiana-tchouakam-chouacheu-91152935b/) · [GitHub](https://github.com/TatianaTchouakam)

Data: © Umweltbundesamt, © Deutscher Wetterdienst (open data).