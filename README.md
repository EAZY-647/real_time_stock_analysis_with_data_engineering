📈 Real-Time Stock Market Data Engineering Project

This project delivers a real-time, end-to-end data engineering pipeline for ingesting, processing, transforming, and visualizing live stock market data using a fully containerized event-driven architecture. The system reduces data latency from hours → seconds by replacing batch ETL with modern streaming and ELT patterns.

🏗️ Architecture Overview

Data Flow (High-Level):

Extraction: Python producer polls Finnhub API for real-time stock quotes.

Streaming Buffer: Events are pushed into Apache Kafka.

Orchestration: Apache Airflow triggers a DAG every minute consuming Kafka messages.

Data Lake (Bronze): Raw JSON is stored in MinIO (S3).

Data Warehouse: Airflow loads files into Snowflake using COPY INTO.

Transformations: dbt parses JSON → Silver tables → Gold KPIs.

Visualization: Power BI (DirectQuery) displays real-time dashboards.

📂 Project Structure
├── dbt_stocks/              # dbt project (silver & gold models)
├── infra/                   # Docker infrastructure
│   ├── docker-compose.yml   # Airflow, Kafka, Zookeeper, MinIO
│   ├── producer/            # Real-time API → Kafka producer
│   └── dags/                # Airflow DAGs
└── requirements.txt         # Python dependencies

🚀 Setup Guide
1️⃣ Requirements

Docker Desktop (≥4GB RAM)

Python 3.9+

Snowflake account (free trial OK)

Finnhub API Key

2️⃣ Start Infrastructure (Kafka, Airflow, MinIO)
cd infra
docker-compose up -d

3️⃣ Prepare Snowflake
USE ROLE ACCOUNTADMIN;
CREATE WAREHOUSE IF NOT EXISTS COMPUTE_WH WITH WAREHOUSE_SIZE = 'XSMALL';
CREATE DATABASE IF NOT EXISTS STOCKS_MDS;
CREATE SCHEMA IF NOT EXISTS STOCKS_MDS.COMMON;

CREATE TABLE IF NOT EXISTS STOCKS_MDS.COMMON.BRONZE_STOCK_QUOTES_RAW (
  V VARIANT
);

4️⃣ Run the Real-Time Producer
python -m venv venv
source venv/bin/activate    # Windows: venv\Scripts\activate
pip install -r requirements.txt
python infra/producer/producer.py

5️⃣ Airflow Orchestration

Open: http://localhost:8080

Login: airflow / airflow

Unpause DAG: minio_to_snowflake

Runs every minute: Kafka → MinIO → Snowflake

6️⃣ Transform Data with dbt
cd dbt_stocks
dbt deps
dbt run

7️⃣ Real-Time Dashboard (Power BI)

Get Data → Snowflake

Use DirectQuery mode

Load:

GOLD_KPI

GOLD_CANDLESTICK

GOLD_TREECHART

🛠️ Tech Stack
Layer	Technology
Streaming	Apache Kafka
Orchestration	Apache Airflow
Data Lake	MinIO (S3)
Warehouse	Snowflake
Transformations	dbt Core
Dashboard	Power BI
Language	Python 3.9+
Infra	Docker
💡 Key Learnings

Use host.docker.internal for container → host communication.

Snowflake PUT/COPY requires explicit roles (e.g., ACCOUNTADMIN).

Power BI slicers require “Edit Interactions” to avoid incorrect aggregation.

🎥 Reference

This project is inspired by Data With Jay — adapted and extended for real-time processing.

📝 License

Licensed under MIT License — free for personal and commercial use.
