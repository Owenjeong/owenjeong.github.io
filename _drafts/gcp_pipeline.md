---
layout: single
title:  "GCP Pipeline"
categories: Cloud
toc: true
toc_sticky: true
toc_label: Contents
author_profile: false
---

# GCP Data Pipeline
---

### *1. Overview*
This post covers how I built a data pipeline on Google Cloud Platform. The idea was to create an end-to-end flow that pulls data from various sources, processes it, and lands it in BigQuery for analytics — leaning on GCP's managed services to keep operations lightweight.

The pipeline follows an **ELT (Extract → Load → Transform)** approach, where raw data is loaded into BigQuery first and transformations happen in SQL — which plays to BigQuery's strengths.


### *2. Architecture*

```
Data Sources → Cloud Functions (Ingest) → GCS (Raw) → BigQuery (Load) → dbt (Transform) → Looker / Dashboard
                                                          ↑
                                                  Cloud Composer (Orchestration)
                                                          ↑
                                                  Cloud Scheduler (Trigger)
```

| Component | GCP Service | Role |
|---|---|---|
| Ingestion | Cloud Functions | Lightweight, event-driven data pulls |
| Storage | Cloud Storage (GCS) | Staging area for raw files |
| Warehouse | BigQuery | Central analytics warehouse |
| Transform | dbt (on BigQuery) | SQL-based data modeling and transformation |
| Orchestration | Cloud Composer (Airflow) | Schedule and coordinate pipeline steps |
| Scheduling | Cloud Scheduler | Cron-based triggers for pipelines |
| Streaming | Pub/Sub + Dataflow | Real-time event ingestion |
| Monitoring | Cloud Monitoring + Logging | Alerts, dashboards, and log analysis |
| IAM | IAM + Service Accounts | Least-privilege access control |

<!-- ![image]({{site.url}}/assets/images/gcp_pipeline/architecture.png) -->


### *3. Data Ingestion*
I use Cloud Functions to pull data from APIs and drop it into GCS as JSON or CSV. Each function is small, single-purpose, and triggered by Cloud Scheduler on a cron.

GCS bucket structure follows a clean partition layout:

```
gs://my-data-lake/
  raw/
    source=api_a/
      dt=2026-02-21/
        data.json
  staging/
    table_name/
      dt=2026-02-21/
        data.parquet
```

For real-time use cases, Pub/Sub captures events and Dataflow (Apache Beam) streams them into BigQuery with minimal latency.

A simple Cloud Function for API ingestion:

```python
import requests
from google.cloud import storage
from datetime import datetime
import json

def ingest_api(request):
    response = requests.get("https://api.example.com/data")
    data = response.json()

    client = storage.Client()
    bucket = client.bucket("my-data-lake")
    dt = datetime.utcnow().strftime("%Y-%m-%d")
    blob = bucket.blob(f"raw/source=api_a/dt={dt}/data.json")
    blob.upload_from_string(json.dumps(data), content_type="application/json")

    return "OK", 200
```


### *4. Loading into BigQuery*
Once raw files land in GCS, they're loaded into BigQuery using either:

- **BigQuery Load Jobs** — For batch, scheduled loads from GCS.
- **External Tables** — Query GCS files directly without loading (good for exploration).
- **Streaming Inserts** — For real-time data via Pub/Sub + Dataflow.

Batch load example using the Python client:

```python
from google.cloud import bigquery

client = bigquery.Client()

job_config = bigquery.LoadJobConfig(
    source_format=bigquery.SourceFormat.NEWLINE_DELIMITED_JSON,
    autodetect=True,
    write_disposition=bigquery.WriteDisposition.WRITE_APPEND,
)

uri = "gs://my-data-lake/raw/source=api_a/dt=2026-02-21/*.json"

load_job = client.load_table_from_uri(
    uri, "my_project.raw_dataset.api_a_events", job_config=job_config
)
load_job.result()  # wait for completion

print(f"Loaded {load_job.output_rows} rows.")
```

BigQuery handles schema detection automatically, but I usually define schemas explicitly for production tables to catch upstream changes early.


### *5. Transformation with dbt*
Once raw data is in BigQuery, dbt handles all the transformation in SQL. I organize models in layers:

```
models/
  staging/
    stg_api_a_events.sql      -- clean, rename, cast
  intermediate/
    int_user_activity.sql      -- joins, business logic
  marts/
    fct_daily_summary.sql      -- final analytical tables
```

A staging model example:

```sql
-- stg_api_a_events.sql
SELECT
    CAST(event_id AS STRING) AS event_id,
    user_id,
    TIMESTAMP(event_time) AS event_at,
    LOWER(event_type) AS event_type,
    COALESCE(amount, 0) AS amount
FROM {{ source('raw', 'api_a_events') }}
WHERE event_id IS NOT NULL
```

dbt handles dependencies, testing, and documentation — so if `stg_api_a_events` changes, everything downstream rebuilds in the right order.


### *6. Orchestration with Cloud Composer*
Cloud Composer (managed Airflow) ties everything together. A typical DAG:

1. **Trigger** — Cloud Scheduler kicks off the DAG daily.
2. **Ingest** — Cloud Function pulls fresh data into GCS.
3. **Load** — BigQuery load job moves data from GCS to raw tables.
4. **Transform** — `dbt run` builds staging → intermediate → mart models.
5. **Test** — `dbt test` validates data quality (nulls, uniqueness, accepted values).
6. **Notify** — Send a Slack or email alert on success or failure.

Each task has retries and SLAs configured, so failures are caught and escalated quickly.


### *7. Querying & Dashboards*
BigQuery is the single source of truth. Analysts and dashboards hit the mart tables directly:

```sql
SELECT
    event_date,
    COUNT(DISTINCT user_id) AS active_users,
    SUM(amount) AS total_amount,
    AVG(amount) AS avg_amount
FROM `my_project.marts.fct_daily_summary`
WHERE event_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
GROUP BY event_date
ORDER BY event_date DESC;
```

For dashboarding, Looker Studio (or any BI tool) connects to BigQuery — the mart layer is designed to be dashboard-ready with clean naming and pre-aggregated metrics.


### *8. Cost & Performance Tips*

- **Partition BigQuery tables** — By date or ingestion time. Cuts query costs and speeds up scans.
- **Cluster on high-cardinality columns** — `user_id`, `event_type`, etc. BigQuery sorts data for faster filters.
- **Use `SELECT` only what you need** — BigQuery charges per column scanned, not per row.
- **Set slot reservations for production** — Predictable cost vs. on-demand pricing spikes.
- **GCS lifecycle rules** — Archive or delete old raw files after processing.
- **dbt incremental models** — Only process new rows instead of full refreshes.
- **Monitor with BigQuery INFORMATION_SCHEMA** — Track query costs, slot usage, and slow queries.


### *9. What I Learned*

- **ELT > ETL on BigQuery** — Loading raw data first and transforming in SQL is faster and more flexible than processing before loading. BigQuery's compute is cheap and powerful.
- **dbt is a game-changer** — Version-controlled SQL, built-in testing, and automatic dependency graphs made the transformation layer way more maintainable than ad-hoc scripts.
- **Cloud Composer is heavy** — It's a full Airflow cluster under the hood. For simpler pipelines, Cloud Workflows or even chained Cloud Functions can save money and complexity.
- **IAM service accounts need discipline** — One service account per service, least-privilege permissions. It's tedious to set up but saves you from debugging weird access errors later.
- **Start with batch, add streaming when needed** — Pub/Sub + Dataflow is powerful but adds operational overhead. Most analytics use cases are fine with daily or hourly batch runs.
