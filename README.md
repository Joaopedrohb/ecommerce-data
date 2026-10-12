# Ecommerce Data

### Building a reliable data foundation for e-commerce analytics

**A personal data engineering project by João Pedro Hieda Bortolotto.**

This project simulates an online store to explore how customer, product, order, payment, and inventory data can be organized into a reliable source for business analysis.

The current implementation uses **Python, SQL, PostgreSQL, and Docker** to create a local database, generate synthetic data, and validate relationships between records. The next stages will introduce Databricks and dbt for analytical processing.

## Project at a glance

| | Overview |
| --- | --- |
| Business scenario | An e-commerce operation with customers, products, orders, payments, and inventory |
| Delivered so far | A local database environment, a data generation notebook, and initial integrity checks |
| Scope | Seven related tables populated with synthetic data |
| Skills demonstrated | Data modeling, Python, SQL, environment configuration, data validation, and troubleshooting |
| Next milestone | Ingestion into Databricks and development of analytical layers |

## What this project demonstrates

- **Understanding a business domain:** translating an e-commerce operation into connected data entities.
- **Building a working data source:** configuring PostgreSQL with Docker and loading records through Python.
- **Checking data reliability:** validating relationships and confirming that records reach the intended tables.
- **Solving implementation problems:** diagnosing connection issues, configuring Python, and correcting loading logic.
- **Documenting progress:** versioning the environment and notebook, with clear milestones for future development.

The documented integrity checks returned **zero orphan records in the relationships tested**. This result applies to the recorded execution; each new load requires validation.

## Business relevance

The model provides a foundation for future analyses of:

- Sales performance by product, category, and period.
- Average order value and customer purchasing frequency.
- Product demand and inventory availability.
- Consistency between order amounts and payment records.

These analytical use cases still require implementation and validated business rules, including how to handle cancellations and payment statuses. All data is synthetic and intended for learning.

## About the author

**João Pedro Hieda Bortolotto — Data Engineer**

My professional experience includes Microsoft Fabric, lakehouses, data notebooks, pipelines, and Power BI. This project expands my hands-on practice with PostgreSQL, Docker, and Python, with Databricks and dbt planned for the next stages.

---

## Technical documentation

### Technology stack

| Technology | Role | Status |
| --- | --- | --- |
| PostgreSQL | Relational source database | Implemented |
| Docker / Docker Compose | Local database environment | Implemented |
| Python | Data generation and loading | Implemented |
| Faker | Synthetic data generation in English | Implemented |
| Psycopg 3 | Python-to-PostgreSQL connection | Implemented |
| SQL | Data inspection and integrity checks | Used |
| Git / GitHub | Version control | Used |
| Databricks / PySpark | Ingestion and analytical processing | Planned |
| dbt | Analytical transformations and model tests | Planned |

### Current architecture

Docker Compose runs PostgreSQL locally. A Python notebook generates records with Faker and loads them through Psycopg 3, respecting table dependencies. SQL queries inspect the resulting data and check relationships.

| Component | Responsibility |
| --- | --- |
| Docker Compose | Configure and run the database environment |
| PostgreSQL | Store simulated transactional data |
| Python notebook | Generate and load records |
| `postgres_data` volume | Preserve database files between sessions |
| SQL queries | Inspect table populations and validate relationships |

### Data model

| Table | Purpose |
| --- | --- |
| `customers` | Customer records |
| `categories` | Product categories |
| `products` | Product catalog associated with categories |
| `orders` | Orders associated with customers |
| `order_items` | Products and quantities within each order |
| `payments` | Payment information related to sales |
| `inventory` | Product inventory information |

Separating orders from order items supports multiple items per purchase. It also establishes distinct order and item granularities that must be respected when calculating metrics to avoid double-counting.

### Repository contents

| File | Purpose |
| --- | --- |
| `docker-compose.yml` | PostgreSQL and persistent volume configuration |
| [`notebooks/generate_ecommerce_data.ipynb`](notebooks/generate_ecommerce_data.ipynb) | Synthetic data generation and loading |
| `README.md` | Overview, setup instructions, and roadmap |

### Local setup

#### 1. Prerequisites

- Docker with Docker Compose support.
- Python and a notebook environment, such as VS Code with the Python and Jupyter extensions.
- A local copy of this repository.

#### 2. Start PostgreSQL

With Docker running, open a terminal at the repository root:

```powershell
docker compose up -d
docker compose ps
```

The documented environment uses:

| Setting | Value |
| --- | --- |
| Container | `ecommerce-postgres` |
| Database | `ecommerce` |
| User | `data_engineer` |
| Host for the local notebook | `localhost` |
| Host port | `5433` |
| Database port inside the container | `5432` |
| Named volume | `postgres_data` |

Host port `5433` avoids a conflict with another local PostgreSQL installation. Check the mapping in `docker-compose.yml` and use matching notebook connection settings, including the password configured for your environment.

#### 3. Prepare Python

Run in PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install faker "psycopg[binary]" ipykernel
```

Open the notebook, select the `.venv` interpreter as its kernel, and verify the database connection settings.

#### 4. Generate and load data

Run [generate_ecommerce_data.ipynb](notebooks/generate_ecommerce_data.ipynb) in its defined cell order.

Tables and constraints must exist before insertion. For a fresh database, execute the schema definitions first if they are not included in the notebook. A standalone schema creation script is a roadmap item.

Generation depends on previously created keys. Re-running insertion cells may add records; check the database state before another full load.

#### 5. Inspect the load

Connect to PostgreSQL:

```powershell
docker exec -it ecommerce-postgres psql -U data_engineer -d ecommerce
```

Check row counts:

```sql
SELECT 'customers' AS table_name, COUNT(*) AS row_count FROM customers
UNION ALL
SELECT 'categories', COUNT(*) FROM categories
UNION ALL
SELECT 'products', COUNT(*) FROM products
UNION ALL
SELECT 'orders', COUNT(*) FROM orders
UNION ALL
SELECT 'order_items', COUNT(*) FROM order_items
UNION ALL
SELECT 'payments', COUNT(*) FROM payments
UNION ALL
SELECT 'inventory', COUNT(*) FROM inventory;
```

Counts depend on generation parameters and database state. Record them per execution rather than assuming a fixed dataset size.

#### 6. Stop and resume

At the end of a session:

```powershell
docker compose stop
```

To resume the existing containers:

```powershell
docker compose start
```

The persistent volume retains the data between sessions, so stopping the environment does not require a new load.

### Data quality and implementation decisions

- **Referential integrity:** primary and foreign keys connect entities, supported by relationship checks.
- **Dependency-aware loading:** reference records are created before records that depend on them.
- **Load inspection:** checks confirm that inserts reach the intended tables, including `order_items`.
- **Persistent storage:** a named volume preserves the database between development sessions.
- **Explicit connection settings:** port mapping directs the notebook to the intended PostgreSQL instance.

Initial checks cover selected relationships. Planned improvements include duplicate detection, required-field checks, financial consistency rules, and business metric validation.

### Planned analytical architecture

| Layer | Intended responsibility |
| --- | --- |
| Bronze | Ingest source data and capture load metadata |
| Silver | Standardize types, handle duplicates, and apply quality rules |
| Gold | Provide analytical models and business metrics |

The transfer mechanism between local PostgreSQL and Databricks remains to be defined and validated. Cloud ingestion, analytical layers, and dbt models are future milestones.

### Roadmap

- [x] Configure PostgreSQL with Docker Compose.
- [x] Build the relational e-commerce database.
- [x] Develop synthetic data generation with Python and Faker.
- [x] Run initial integrity checks.
- [x] Version the environment configuration and notebook.
- [ ] Add a standalone, versioned SQL schema script.
- [ ] Centralize configuration and dependency management.
- [ ] Implement ingestion from PostgreSQL into Databricks.
- [ ] Develop Bronze, Silver, and Gold layers.
- [ ] Implement incremental processing and load tracking.
- [ ] Build analytical models and tests with dbt.
- [ ] Document dataset size, execution time, and quality metrics.
