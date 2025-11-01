ELT pipeline: GitHub → S3 → Snowflake (Bronze/Silver/Gold) orchestrated by AWS Managed Airflow. CSVs auto-downloaded; idempotent loads; SQL transforms.

<img width="939" height="432" alt="image" src="https://github.com/user-attachments/assets/c4a06042-f7b6-4ea3-a211-c36b922d4445" />

Simple diagram (HTTP→python→S3→Airflow→Snowflake Bronze→Silver→Gold).

What’s included (exactly what you did)

Managed Apache Airflow (MWAA) connection to Snowflake, tested with SELECT CURRENT_TIMESTAMP DAG. 

ETL Airflow + Snowflake AWS Pro…

Python task downloads CSVs from a public repo and uploads to your S3 data bucket. 

ETL Airflow + Snowflake AWS Pro…

 

healthcare_elt_pipeline

Bronze COPY from an external stage using a storage integration (secure AssumeRole with ExternalId). 

ETL Airflow + Snowflake AWS Pro…

Silver stored procs, Gold agg stored procs triggered by SnowflakeOperator. 

healthcare_elt_pipeline

 

healthcare_elt_pipeline

Snowflake setup (exact)
Paste the runnable bits you used:

00_db_and_schemas.sql: create AIRFLOW_DB and BRONZE/SILVER/GOLD. 

ETL Airflow + Snowflake AWS Pro…

01_storage_integration.sql: CREATE STORAGE INTEGRATION AIRFLOW_S3_INT … STORAGE_AWS_ROLE_ARN … STORAGE_ALLOWED_LOCATIONS. 

ETL Airflow + Snowflake AWS Pro…

Then DESC INTEGRATION AIRFLOW_S3_INT to copy STORAGE_AWS_IAM_USER_ARN and STORAGE_AWS_EXTERNAL_ID into the IAM trust policy. 

ETL Airflow + Snowflake AWS Pro…

 

ETL Airflow + Snowflake AWS Pro…

02_stage.sql: external stage pointing at your data bucket (the one Airflow writes into).

03_raw_tables.sql: create DOCTORS_RAW, HOSPITALS_RAW, PATIENTS_RAW, TREATMENTS_RAW, VISITS_RAW (you hit a miss here once; document it).

10_silver_transforms.sql & 20_gold_aggregates.sql: the procs you call from Airflow (names match the DAG). 

healthcare_elt_pipeline

 

healthcare_elt_pipeline

Airflow bits

Where you created snowflake_conn in the Airflow UI (MWAA → Admin → Connections), with account/warehouse/db/role/schema fields. 

ETL Airflow + Snowflake AWS Pro…

How the DAG works:

upload_csvs_github_to_s3 (Python callable)

truncate_and_copy_*_raw_table (parallel copies from @AIRFLOW_S3_STAGE/file.csv) 

healthcare_elt_pipeline

Silver SPs → Gold SPs with dependency fan-ins/outs. 

healthcare_elt_pipeline

Validation

scripts/verify_run.sql: union row counts of RAW tables + a COPY_HISTORY sample (you used this).

Also instruct to run LIST @AIRFLOW_S3_STAGE to confirm stage visibility.

Screenshots to include

S3 bucket object list (doctors.csv, …)

Airflow DAG graph with a successful run

Snowflake LIST @stage and SELECT COUNT(*) from RAW

Optional: Query History showing proc calls and COPY statements. 

ETL Airflow + Snowflake AWS Pro…

Troubleshooting (document your real fixes)

Empty tables after “success” → Stage pointed to the wrong bucket; recreate stage to correct data bucket.

AssumeRole error → Fix IAM role trust policy using Snowflake’s STORAGE_AWS_IAM_USER_ARN + STORAGE_AWS_EXTERNAL_ID. 

ETL Airflow + Snowflake AWS Pro…

 

ETL Airflow + Snowflake AWS Pro…

42S02 on DOCTORS_RAW → Create missing table / match quoted case exactly.

CI (small but legit)

.github/workflows/ci.yaml that:

installs Airflow + Snowflake provider

runs flake8 on dags/

parses DAGs with DagBag (fails PRs if DAG breaks).
