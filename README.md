## etl_pipeline_assignment_shantanu

This project implements a containerized ETL pipeline using **Apache Spark**, **PostgreSQL**, **Docker**, and **Poetry**. It extracts customer transaction data from a CSV file, performs transformations using **PySpark**, and loads the final results into a **PostgreSQL** database.

---

## Project Structure

```
.
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── main.py
├── pyproject.toml
├── etl/
│   ├── __init__.py
│   └── transformations.py
├── data/
│   ├── transaction.csv           
│   └── dummy_transactions.csv    
├── tests/
│   ├── test_transform.py
│   └── test_integration.py
└── docs/                         
```

---

## Setup Instructions

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd <your-repo>
```

### 2. Add Your Data

transaction.csv and dummy_transactions.csv files are already included.

Place your source CSV file (pipe-delimited with the following headers) into the `data/` folder:

```text
custid|transactiondate|productsold|unitssold
```

---

## Running the Pipeline with Docker Compose

### 1. Build and Start All Services

```bash
docker-compose up --build
```

This starts the following services:
- **Spark master and worker** (Bitnami Spark 3.3.2)
- **PostgreSQL** database (`warehouse`)
- **ETL container** (paused until command is issued)

### 2. Execute the ETL Job

Accessing and running the ETL script inside the container:

```bash
docker compose exec etl python main.py --source /app/data/<file_name>.csv --table customers
```

> Replace `file_name` with the actual filename placed in `/data`.

---

## Configurable Environment Variables

These are set in the `docker-compose.yml` and can also be overridden via `.env`:

| Variable            | Default                     | Description                        |
|---------------------|-----------------------------|------------------------------------|
| `POSTGRES_DB`       | `warehouse`                 | Target PostgreSQL database         |
| `POSTGRES_USER`     | `postgres`                  | PostgreSQL username                |
| `POSTGRES_PASSWORD` | `mypassword`                | PostgreSQL password                |
| `POSTGRES_HOST`     | `db`                        | Hostname for PostgreSQL service    |
| `SPARK_MASTER`      | `spark://spark-master:7077` | Spark master cluster URL           |

---

## Running Tests

### Unit Test (Transform Logic)

```bash
docker compose exec etl pytest tests/test_transformations.py
```

### Integration Test (Full Pipeline with Dummy CSV)

```bash
docker compose exec etl pytest tests/test_etl_integration.py
```

> Ensure that a dummy input file is available in `data/` for the integration test.
> 1 dummy file (dummy_transactions.csv) already exists in app/data/ 

---

## What the ETL Pipeline Does

- **Extract**: Loads transactional data from a pipe-delimited CSV.
- **Transform**:
  - Derives `favourite_product` per customer (highest total units sold).
  - Computes `longest_streak` of consecutive daily purchases.
- **Load**: Writes the final DataFrame into a PostgreSQL table with schema:

| Column              | Description                                              |
|---------------------|----------------------------------------------------------|
| `customer_id`       | Mapped from `custid`                                     |
| `favourite_product` | Product with the highest total `unitsSold` per customer  |
| `longest_streak`    | Longest consecutive-day streak of purchases              | 

---

## Database Query to verify

```bash
docker compose exec db psql --user postgres -d warehouse -c 'select * from customers limit 10'
```

## Dependencies

Dependencies are managed with **Poetry** and listed in `pyproject.toml`:

- `pyspark==3.3.2`
- `psycopg2-binary`
- `pytest`
- `pandas`

