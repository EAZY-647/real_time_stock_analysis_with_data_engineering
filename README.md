# 📈 End-to-End Real-Time Stock Market Data Engineering Project

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python)
![Docker](https://img.shields.io/badge/Docker-24.0%2B-2496ED?logo=docker)
![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-7.4-231F20?logo=apachekafka)
![Apache Airflow](https://img.shields.io/badge/Apache%20Airflow-2.9-017CEE?logo=apacheairflow)
![Snowflake](https://img.shields.io/badge/Snowflake-Enterprise-29B5E8?logo=snowflake)
![dbt](https://img.shields.io/badge/dbt-Core%201.8-FF694B?logo=dbt)
![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi)

## 📖 Executive Summary

This project implements a scalable, fault-tolerant data engineering pipeline capable of ingesting, processing, and visualizing real-time stock market data.

Traditional batch-based ETL pipelines often suffer from data latency, making them unsuitable for intraday financial analysis. This solution replaces batch processing with a modern **Event-Driven Architecture (ELT)** to reduce data latency from hours to seconds.

The system streams live stock data from the **Finnhub API** into **Apache Kafka**, orchestrates loading into a **MinIO Data Lake** via **Apache Airflow**, warehouses the data in **Snowflake**, transforms it using **dbt**, and visualizes volatility metrics in a real-time **Power BI** dashboard.

---

## 🏗️ System Architecture

The pipeline follows a microservices architecture orchestrated by Docker Containers.

### Data Flow Breakdown
1.  **Extraction:** The Producer Service polls the **Finnhub API** for stock quotes (Apple, Amazon, Google, Microsoft, Tesla).
2.  **Buffering:** Data is pushed to **Apache Kafka**, acting as a fault-tolerant buffer to handle traffic spikes.
3.  **Orchestration:** **Apache Airflow** triggers a DAG every minute to consume data from Kafka.
4.  **Staging (Bronze):** Raw JSON files are stored in **MinIO** (S3-compatible object storage) as an immutable Data Lake.
5.  **Loading:** Airflow loads the JSON files into **Snowflake** using internal stages and the `COPY INTO` command.
6.  **Transformation (Silver/Gold):** **dbt** executes SQL models to parse the JSON (Silver) and calculate KPIs like volatility (Gold).
7.  **Reporting:** **Power BI** queries the Gold tables via DirectQuery for live visualization.

---

## 📂 Project Structure

```bash
├── dbt_stocks/             # dbt Project (Transformation Layer)
│   ├── dbt_project.yml     # dbt configuration
│   ├── models/
│   │   ├── bronze/         # Sources definition (Snowflake Raw Tables)
│   │   ├── silver/         # SQL logic to parse JSON & deduplicate
│   │   └── gold/           # Analytical Views (KPIs, Charts)
├── infra/                  # Infrastructure as Code (Docker)
│   ├── docker-compose.yml  # Blueprint for Airflow, Kafka, Zookeeper, MinIO
│   ├── producer/           # Python script for API -> Kafka ingestion
│   └── dags/               # Airflow DAGs (minio_to_snowflake.py)
└── requirements.txt        # Python dependencies for the producer

🚀 Getting Started

1. Prerequisites

Ensure you have the following installed/configured:

Docker Desktop: Allocated with at least 4GB RAM (Linux containers mode).

Python 3.9+: Installed locally for running the producer.

Snowflake Account: A free trial account is sufficient.

Finnhub API Key: Sign up for a free API key at Finnhub.io.

2. Infrastructure Setup

Navigate to the infra directory and spin up the containerized environment. This initializes Airflow (Webserver/Scheduler), Kafka, Zookeeper, and MinIO.

cd infra
docker-compose up -d


Wait for a few minutes for all health checks to pass.

3. Snowflake Configuration

Log into your Snowflake console (app.snowflake.com) and execute the following SQL to prepare the warehouse environment.
Note: This setup uses ACCOUNTADMIN for simplicity to avoid permission issues.

-- Create Warehouse, Database, and Schema
USE ROLE ACCOUNTADMIN;
CREATE WAREHOUSE IF NOT EXISTS COMPUTE_WH WITH WAREHOUSE_SIZE = 'XSMALL' AUTO_SUSPEND = 60 AUTO_RESUME = TRUE;
CREATE DATABASE IF NOT EXISTS STOCKS_MDS;
CREATE SCHEMA IF NOT EXISTS STOCKS_MDS.COMMON;

-- Create Raw Table for JSON Data (Schema-on-Read)
CREATE TABLE IF NOT EXISTS STOCKS_MDS.COMMON.BRONZE_STOCK_QUOTES_RAW (
    V VARIANT
);


4. Running the Data Producer

The producer script simulates a real-time data feed. It must run continuously in a separate terminal window.

# Create and activate virtual environment
python -m venv venv

# Windows:
.\venv\Scripts\activate
# Mac/Linux:
source venv/bin/activate

# Install requirements
pip install -r requirements.txt

# Run the producer
python infra/producer/producer.py


You should see logs indicating data is being pushed to the stock-quotes topic.

5. Orchestration (Airflow)

Access the Airflow UI at http://localhost:8080.

Username: airflow

Password: airflow

Locate the minio_to_snowflake DAG.

Unpause the DAG (toggle the switch on the left).

It will run every minute, moving data from Kafka -> MinIO -> Snowflake.

6. Transformation (dbt)

Once raw data is populated in Snowflake, use dbt to clean and structure it.

cd dbt_stocks

# Install dbt dependencies
dbt deps

# Run the models
dbt run


Expected Output: Completed successfully for Silver and Gold models.

7. Visualization (Power BI)

Open Power BI Desktop.

Select Get Data -> Snowflake.

Enter your Snowflake Server URL (e.g., xy12345.us-east-1.aws.snowflakecomputing.com) and Warehouse (COMPUTE_WH).

CRUCIAL: Select DirectQuery mode to ensure real-time updates.

Import the GOLD_KPI, GOLD_CANDLESTICK, and GOLD_TREECHART views.

Data Modeling: Ensure relationships are set to "Both" directions for cross-filtering.

🛠️ Tech Stack Details

Language: Python 3.9+ (Producer scripts & Airflow DAGs)

Containerization: Docker 24.0+ (Microservices orchestration)

Streaming: Apache Kafka 7.4 (Distributed event streaming)

Orchestration: Apache Airflow 2.9 (Workflow scheduling & monitoring)

Storage: MinIO (S3-compatible object storage for Bronze Lake)

Warehouse: Snowflake (Cloud Data Warehouse)

Transformation: dbt Core 1.8 (SQL-based transformations)

BI: Power BI Desktop (Dashboarding & Analytics)

💡 Troubleshooting & Key Learnings

Docker Networking:
Airflow containers use the internal DNS host.docker.internal to communicate with services running on the host machine or other containers.

Snowflake Permissions:
The Python connector inside Airflow requires explicit Role enforcement (role='ACCOUNTADMIN') to perform PUT operations into the staging area if default user roles are restricted.

Power BI Aggregations:
When using KPI cards, ensure the "Slicer" visual interacts correctly with the cards by checking "Edit Interactions" format options to avoid summing up values for all stocks.

🎥 Reference & Acknowledgements

This project was built following the comprehensive tutorial by Data with Jay. The architectural patterns for integrating Airflow with MinIO and Snowflake were adapted from his guidance.

Video: End-to-End Stock Market Data Engineering Project

Channel: Data with Jay

📝 License

This project is open-source and free to use. Licensed under the MIT License.
