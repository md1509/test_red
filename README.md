# **📈 Stock Trend Prediction Pipeline**

A complete data engineering pipeline that processes historical stock data to detect market trends. The system ingests Apple stock data, performs distributed feature engineering, and provides analytics through an interactive dashboard.

---

## **1. Architecture Overview (Medallion Structure)**

The pipeline follows the **Medallion Architecture**, ensuring scalable, repeatable, and reliable processing:

1. **Data Ingestion (Bronze):**  
   Downloads raw Apple stock data from a source (e.g., GitHub).

2. **Raw Storage (Bronze):**  
   Breaks data into monthly chunks and uploads them to **MinIO**.  
   Includes MD5-based integrity validation.

3. **Transformation (Silver/Gold):**  
   **Apache Spark** computes features such as moving averages and volatility while enforcing schema quality.

4. **Feature Store (Gold):**  
   **PostgreSQL** holds the analytics-ready dataset.

5. **Visualization:**  
   **Metabase** provides dashboards and business intelligence.

The full workflow is orchestrated with **Apache Airflow**.

---

## **2. Tech Stack**

All components are fully containerized with **Docker + Docker Compose**.

| Component | Technology | Role |
|----------|------------|------|
| **Orchestration** | Apache Airflow | Task scheduling and pipeline dependency management |
| **Data Lake** | MinIO | S3-like object storage for raw/intermediate files |
| **Processing** | Apache Spark | Distributed computations and feature engineering |
| **Data Warehouse** | PostgreSQL | Stores the transformed analytics dataset |
| **Visualization** | Metabase | Interactive dashboards |
| **Deployment** | Docker & Docker Compose | Environment orchestration |

---

## **3. Features Calculated**

The pipeline focuses on **trend classification** (Up, Down, Neutral), not exact price prediction.

| Feature | Category | Basis | Purpose |
|--------|----------|--------|---------|
| `daily_return` | Momentum | % change in closing price | Core performance metric |
| `ma_10d`, `ma_20d` | Trend | 10-day & 20-day SMA | Identifies trend & crossovers |
| `volatility_10d` | Risk | 10-day rolling std of returns | Measures short-term risk |
| `trend_label` | Classification | Based on `daily_return` threshold | Target variable |

### **Trend Label Classification Logic**

Using a ±0.1% threshold on daily returns:

- **Uptrend:** `daily_return > 0.1%`  
- **Downtrend:** `daily_return < -0.1%`  
- **Neutral:** between −0.1% and +0.1%

---

## **4. Conceptual Setup Instructions**

Ensure **Docker** and **Docker Compose** are installed.

### **A. Deployment Steps**

1. **Clone the repository:**
   ```bash
   git clone https://github.com/asmaaelrasif/Stock-trend-prediction.git
   cd Stock-trend-prediction
````

2. **Create & configure the `.env` file:**
   Must exist in the root directory.
   Contains ports, credentials, and config variables.

3. **Build and start all services:**

   ```bash
   docker-compose up -d --build
   ```

4. **Initialize Airflow:**

   ```bash
   docker compose up airflow-init
   ```

5. **Verify running containers:**

   ```bash
   docker ps
   ```

6. **Access the interfaces:**

   * **Airflow:** [http://localhost:8080](http://localhost:8080)
   * **pgAdmin:** [http://localhost:5050](http://localhost:5050)
   * **MinIO:** [http://localhost:9001](http://localhost:9001)
   * **Metabase:** [http://localhost:3000](http://localhost:3000)

---

### **B. How to Use**

1. Open the Airflow UI.
2. Enable and trigger the `stock_prediction_dag`.
3. Watch the pipeline steps:
   **Ingest → Chunk → Spark Transform → Load to Postgres**
4. After completion, open Metabase to explore the dashboards.

---

## **5. Pipeline Details and Limitations**

### **Incremental Processing**

* Runs **daily** and only processes *new* monthly chunks.
* Uses a **PostgreSQL tracking table** to avoid reprocessing.
* **MD5 checksums** detect modified raw data.
* Spark always loads the **last 20 days** from previous months to ensure correct rolling-window calculations.

### **Database Schema**

Fields include:

* Core OHLCV fields
* `ticker`, `name`
* `prev_close`, `daily_return`
* `trend_label`
* `ma10`, `ma20`
* `volatility_10d`
* `source_file` (lineage tracking)

### **Known Limitations**

1. Single-ticker (AAPL only).
2. Rolling-window lookback size fixed at 20 days.
3. PostgreSQL loads use simple append mode.
4. Batch-only; no streaming support.

---

## **6. Project Structure**

```
Stock-trend-prediction/
├── config/              # Service configs (Spark, Airflow, etc.)
├── dags/                # Airflow DAGs
├── data/                # Local raw data storage
├── logs/                # Airflow & service logs
├── plugins/             # Custom Airflow hooks/operators
├── scripts/             # Utility scripts (e.g., MinIO setup)
├── spark-scripts/       # PySpark feature engineering scripts
├── Dockerfile.airflow   # Custom Airflow image
├── Dockerfile.spark     # Custom Spark image
├── docker-compose.yml   # Main orchestration file
├── init.sql             # PostgreSQL initialization
└── .env                 # Environment variables (required)
```

---

## **7. Authors**

* **Mohammed Darwish**
* **Asmaa Elrasef**

*DSAl 5102 — Data Architecture & Engineering
Fall 2025*
