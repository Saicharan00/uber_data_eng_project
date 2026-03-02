# 🚖 Uber End-To-End Data Engineering Project (Real-Time Streaming)



## 📌 Project Overview

A comprehensive, **end-to-end data engineering solution** that processes real-time ride-booking data simulating an application like Uber. Built on a modern cloud stack using **Azure** and **Databricks**, this project handles both real-time streaming events and batch mapping data — ultimately serving analytical data through a **Medallion Architecture (Bronze → Silver → Gold)**.

A core feature of this project is its **metadata-driven architecture** using **Jinja2 templating** to make data pipelines modular, reusable, and scalable without hardcoding future requirements.

---

## 🏗️ Architecture Overview
```
                        ┌─────────────────────────────────────────────────────┐
                        │               DATA GENERATION & INGESTION           │
                        │                                                     │
   Real-Time Events     │   FastAPI App (Producer)                            │
   ──────────────►      │        │                                            │
                        │        ▼                                            │
                        │   Azure Event Hubs  ◄──── (Pub/Sub like Kafka)     │
                        │                                                     │
   Batch / Static Data  │   GitHub API (Simulated Internal API)               │
   ──────────────►      │        │                                            │
                        │        ▼                                            │
                        │   Azure Data Factory (ADF) ──► ADLS Gen 2          │
                        └──────────────────────┬──────────────────────────────┘
                                               │
                        ┌──────────────────────▼──────────────────────────────┐
                        │          DATABRICKS — MEDALLION ARCHITECTURE        │
                        │                                                     │
                        │  🥉 BRONZE   →   Raw ingestion (streams + batch)    │
                        │       │                                             │
                        │  🥈 SILVER   →   Cleaned OBT via Jinja SQL joins    │
                        │       │                                             │
                        │  🥇 GOLD     →   Star Schema (Fact + Dimensions)    │
                        │                  SCD Type 2 via Spark autoCDC      │
                        └─────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| **Language** | Python, SQL, Jinja2 |
| **Streaming & Messaging** | Azure Event Hubs (Managed Apache Kafka) |
| **Data Orchestration** | Azure Data Factory (ADF) |
| **Data Storage** | Azure Data Lake Storage Gen 2 (ADLS Gen 2) |
| **Data Transformation** | Databricks (Free Edition), Apache Spark, Spark Declarative Pipelines (SDP) |
| **Web Application** | FastAPI, HTML, Uvicorn |
| **Package Management** | UV Package Manager |

---

## 🚀 Key Features

- ⚡ **Real-Time Kafka Streaming** — FastAPI producers push live taxi ride events to Azure Event Hubs continuously
- 🔄 **Metadata-Driven ADF Pipelines** — Dynamic ForEach loops ingest multiple API files without hardcoding individual copy activities
- 🧩 **Jinja-Powered SQL Pipelines** — Complex join logic in Databricks is auto-generated from a config file, preventing pipeline breaks when new tables are added
- 📜 **Slowly Changing Dimensions (SCD Type 2)** — Full history tracking of city/location changes using Databricks `autoCDC`
- 🏅 **Medallion Architecture** — Clean separation of raw, cleansed, and business-ready data layers

---

## 📐 Data Model (Gold Layer — Star Schema)
```
                        ┌──────────────────┐
                        │   dim_passenger  │
                        └────────┬─────────┘
                                 │
┌──────────────┐    ┌────────────▼──────────────┐    ┌──────────────────┐
│  dim_driver  ├───►│        fact_booking        │◄───┤  dim_vehicle     │
└──────────────┘    └────┬──────────────┬────────┘    └──────────────────┘
                         │              │
              ┌──────────▼──┐      ┌────▼─────────────┐
              │ dim_payment │      │  dim_location     │
              └─────────────┘      │  (SCD Type 2)     │
                                   └───────────────────┘
```

---

## 📋 Prerequisites

Before you begin, ensure you have the following:

- ✅ **Python** and **Git** installed locally
- ✅ **UV Package Manager** — `pip install uv`
- ✅ **Azure Free Account** — for Event Hubs, ADF, and ADLS Gen 2
- ✅ **Databricks Free Edition** account

---

## ⚙️ Setup & Installation

### 1. Clone the Repository
```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
cd YOUR_REPO_NAME
```

### 2. Set Up Virtual Environment & Install Dependencies
```bash
uv init          # Initialize UV environment (if not already done)
uv sync          # Install dependencies from pyproject.toml
```

### 3. Configure Environment Variables

Create a `.env` file in the root directory and add your Azure credentials:
```env
EVENT_HUB_CONNECTION_STRING=your_azure_event_hub_connection_string
EVENT_HUB_NAME=your_event_hub_topic_name
```

> ⚠️ Make sure your Shared Access Policy has **Send** permission enabled on Event Hubs.

### 4. Run the Simulator App
```bash
uvicorn app.main:app --reload
```


---

## 🗂️ Medallion Pipeline Execution

### Step 1 — Azure Data Factory
Trigger the ADF pipeline to load static mapping files into ADLS Gen 2:
- `map_cities.json`
- `bulk_rides.json`
- Vehicle types, payment methods, etc.

### Step 2 — Databricks: Bronze Layer
Run the **Bronze notebooks** to ingest raw data:
- Streaming events from Event Hubs → Bronze streaming table
- Batch mapping files from ADLS Gen 2 → Bronze batch tables

### Step 3 — Databricks: Silver Layer
Run the **Silver Jinja Template notebook** to:
- Dynamically generate SQL join queries from the config file
- Merge incremental streaming data with historic mapping files
- Produce the **One Big Table (OBT)**

### Step 4 — Databricks: Gold Layer
Run the **Gold Model notebook** to:
- Define and populate the **Fact Table** and **Dimension Tables**
- Apply **SCD Type 2** logic to location/city dimension via `autoCDC`

---

## 📁 Project Structure
```
📦 uber-data-engineering
 ┣ 📂 app/                    # FastAPI simulator application
 ┃ ┣ 📜 main.py               # FastAPI entry point
 ┃ ┗ 📜 producer.py           # Event Hub producer logic
 ┣ 📂 databricks/
 ┃ ┣ 📂 bronze/               # Bronze ingestion notebooks
 ┃ ┣ 📂 silver/               # Silver Jinja-powered transformation
 ┃ ┗ 📂 gold/                 # Gold star schema model
 ┣ 📂 adf_pipelines/          # ADF pipeline JSON definitions
 ┣ 📂 config/                 # Jinja config & metadata files
 ┣ 📜 .env.example            # Sample environment variables
 ┣ 📜 pyproject.toml          # UV/Python dependencies
 ┗ 📜 README.md
```





