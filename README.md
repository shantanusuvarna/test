PySpark ETL Pipeline

This project implements a containerized ETL pipeline using **Apache Spark**, **PostgreSQL**, **Docker**, and **Poetry**. It extracts customer transaction data from a CSV file, performs transformations using **PySpark**, and loads the final results into a **PostgreSQL** database.

---

## 🗂️ Project Structure

```
.
├── Dockerfile                  # Defines the ETL container environment
├── docker-compose.yml         # Orchestrates services: Spark, PostgreSQL, ETL
├── main.py                    # Entry point for the ETL job
├── pyproject.toml             # Poetry-managed dependencies
├── etl/                       # Python module for transformation logic
│   ├── __init__.py            # Marks this directory as a Python package
│   └── transformations.py     # Core transformation logic
├── data/                      # Folder to store source CSV files
├── tests/                     # Unit and integration tests
│   ├── test_transform.py      # Unit test for transformation logic
│   └── test_integration.py    # Full pipeline test using dummy CSV
```

---

## 🚀 Setup Instructions

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd <your-repo>
```

### 2. Add Your Data

Place your source CSV file (pipe-delimited with the following headers) into the `data/` folder:

```text
custid|transactiondate|productsold|unitssold
```

---

## 🐳 Running the Pipeline with Docker Compose

### 1. Build and Start All Services

```bash
docker-compose up --build
```

This starts the following services:
- **Spark master and worker** (Bitnami Spark 3.3.2)
- **PostgreSQL** database (`warehouse`)
- **ETL container** (paused until command is issued)

### 2. Execute the ETL Job

Access the running ETL container:

```bash
docker exec -it etl bash
```

Run the ETL script inside the container:

```bash
python main.py --source /app/data/your_file.csv --table customers
```

> Replace `your_file.csv` with the actual filename placed in `/data`.

---

## ⚙️ Configurable Environment Variables

These are set in the `docker-compose.yml` and can also be overridden via `.env`:

| Variable            | Default                     | Description                        |
|---------------------|-----------------------------|------------------------------------|
| `POSTGRES_DB`       | `warehouse`                 | Target PostgreSQL database         |
| `POSTGRES_USER`     | `postgres`                  | PostgreSQL username                |
| `POSTGRES_PASSWORD` | `mypassword`                | PostgreSQL password                |
| `POSTGRES_HOST`     | `db`                        | Hostname for PostgreSQL service    |
| `SPARK_MASTER`      | `spark://spark-master:7077` | Spark master cluster URL           |

---

## 🧪 Running Tests

### Unit Test (Transform Logic)

```bash
pytest tests/test_transform.py
```

### Integration Test (Full Pipeline with Dummy CSV)

```bash
pytest tests/test_integration.py
```

> Ensure that a dummy input file is available in `data/` for the integration test.

---

## 🔁 What the ETL Pipeline Does

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

## 📦 Dependencies

Dependencies are managed with **Poetry** and listed in `pyproject.toml`:

- `pyspark==3.3.2`
- `psycopg2-binary`
- `pytest`
- `pandas`

To install locally (optional):

```bash
poetry install
```

---

## 📬 Contact / License

MIT License • Developed for Sertis Data Engineering Take-Home Test  
Feel free to contribute or reach out with questions!
