---
layout: single
title:  "AWS Data Engineering"
categories: Cloud
toc: true
toc_sticky: true
toc_label: Contents
author_profile: false
---

# AWS Data Engineering
---

### *1. Overview*
This post walks through how I set up a data engineering pipeline on AWS. The goal was to build a reliable, scalable system that ingests raw data, transforms it, and serves it up for analytics — all using managed AWS services so I'm not babysitting servers.

The pipeline follows a classic **Extract → Transform → Load (ETL)** pattern, with a few extras like orchestration, monitoring, and cost controls baked in.


### *2. Architecture*

Here's the high-level flow:

```
Data Sources → S3 (Raw) → Glue (Transform) → S3 (Processed) → Athena / Redshift → Dashboard
                                  ↑
                          Step Functions (Orchestration)
                                  ↑
                         EventBridge (Scheduling)
```

| Component | AWS Service | Role |
|---|---|---|
| Storage | S3 | Landing zone for raw and processed data |
| Catalog | Glue Data Catalog | Schema discovery and metadata management |
| ETL | Glue Jobs (PySpark) | Data cleaning, transformation, aggregation |
| Orchestration | Step Functions | Coordinate multi-step ETL workflows |
| Scheduling | EventBridge | Trigger pipelines on a schedule or event |
| Query | Athena | Serverless SQL queries on S3 data |
| Warehouse | Redshift (optional) | Heavy analytical workloads |
| Monitoring | CloudWatch | Logs, metrics, and alerts |
| IAM | IAM Roles & Policies | Least-privilege access across services |

<!-- ![image]({{site.url}}/assets/images/aws_de/architecture.png) -->


### *3. Data Ingestion*
Raw data lands in S3 — could be CSVs, JSON, or Parquet files from APIs, databases, or streaming sources. I organize the bucket with a partitioned structure:

```
s3://my-data-lake/
  raw/
    source=api_a/
      year=2025/month=11/day=08/
        data.json
  processed/
    table_name/
      year=2025/month=11/
        part-00000.parquet
```

For real-time ingestion, Kinesis Data Firehose can stream data directly into S3 with automatic batching and compression.


### *4. Transformation with Glue*
AWS Glue handles the heavy lifting. I write PySpark jobs that:

- **Clean** — Drop nulls, fix data types, deduplicate.
- **Transform** — Join datasets, calculate derived fields, aggregate.
- **Partition** — Write output in Parquet format, partitioned by date for fast querying.

A typical Glue job looks something like this:

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# Read from catalog
df = glueContext.create_dynamic_frame.from_catalog(
    database="my_database",
    table_name="raw_table"
)

# Transform
df_clean = df.toDF() \
    .dropDuplicates() \
    .filter("amount > 0") \
    .withColumn("processed_date", current_date())

# Write back to S3 as Parquet
df_clean.write \
    .mode("overwrite") \
    .partitionBy("year", "month") \
    .parquet("s3://my-data-lake/processed/clean_table/")

job.commit()
```

The Glue Data Catalog keeps track of all the schemas automatically — once a Crawler runs, Athena can query the data right away.


### *5. Orchestration with Step Functions*
Step Functions tie the pipeline together. A typical workflow:

1. **Trigger** — EventBridge fires on a schedule (e.g. daily at 2 AM UTC).
2. **Crawl raw data** — Glue Crawler updates the catalog.
3. **Run ETL job** — Glue job transforms and writes processed data.
4. **Crawl processed data** — Update the catalog with new partitions.
5. **Notify** — SNS sends a success/failure alert.

Each step has built-in retry logic and error handling, so if a Glue job fails it retries before alerting.


### *6. Querying with Athena*
Once data is in processed S3 buckets and cataloged, Athena lets me run SQL directly on it — no loading into a database, no infrastructure to manage.

```sql
SELECT
    user_id,
    COUNT(*) AS total_events,
    SUM(amount) AS total_amount
FROM processed_db.clean_table
WHERE year = '2025' AND month = '11'
GROUP BY user_id
ORDER BY total_amount DESC
LIMIT 100;
```

Athena charges per TB scanned, so partitioning and Parquet compression make a real difference in cost.


### *7. Cost & Performance Tips*
A few things I picked up along the way:

- **Parquet over CSV** — Columnar format = faster queries and way less data scanned.
- **Partition everything** — Date-based partitions cut Athena costs dramatically.
- **Right-size Glue DPUs** — Start with the minimum workers and scale up only if jobs are slow.
- **Use Glue bookmarks** — Process only new data instead of re-processing the full dataset every run.
- **Set S3 lifecycle rules** — Move old raw data to Glacier after 90 days.
- **Tag everything** — Makes cost tracking and cleanup much easier.


### *8. What I Learned*

- **Serverless doesn't mean zero ops** — You still need to monitor, tune, and debug. CloudWatch dashboards and alerts are essential.
- **Schema evolution is tricky** — When source data changes shape, the Glue Catalog and downstream queries can break. Adding schema validation early saves headaches.
- **Start simple, then optimize** — It's tempting to over-engineer from day one. A basic S3 → Glue → Athena pipeline handles most use cases before you need Redshift or EMR.
- **IAM is the real boss** — Getting permissions right across S3, Glue, Step Functions, and Athena took more time than the actual data work.