# Databricks PySpark Customer Analytics

An exploratory customer and order analytics project built with PySpark and designed to run in Databricks. The notebook cleans customer records, engineers date features, explores customer geography and activity, joins orders to customers, and ranks customers using purchasing behavior.

## Project overview

The analysis answers questions such as:

- Where are customers located, and which cities or state-country combinations have the most customers?
- How many customers are active or inactive in each state?
- Which customers registered most recently within their state?
- How many customers registered on or after January 1, 2025?
- Which customers place the most orders or generate the most revenue?
- What is each customer's average order value?
- How are orders distributed by status and calendar month?
- Which customers order frequently but have relatively low total spend?

## Repository structure

```text
.
├── analytics.ipynb       # Main Databricks/PySpark analysis notebook
├── data/
│   ├── customers.csv     # Customer dimension data (10,000 rows)
│   └── orders.csv        # Order transaction data (10,000 rows)
├── .gitignore
├── README.md
└── requirements.txt      # Dependencies for running the notebook locally
```

## Data

The repository contains synthetic-style CSV data suitable for learning and demonstration. Both files include headers and contain 10,000 records.

### `customers.csv`

| Column | Description | Inferred type |
|---|---|---|
| `customer_id` | Unique customer identifier | string |
| `name` | Customer name | string |
| `city` | Customer city | string |
| `state` | State, province, or region | string |
| `country` | Customer country | string |
| `registration_date` | Customer registration date (`yyyy-MM-dd`) | date |
| `is_active` | Whether the customer is active | boolean |

The included data spans registrations from **2020-01-01 through 2026-09-04**, covers 20 countries, and contains 8,433 active and 1,567 inactive customers.

### `orders.csv`

| Column | Description | Inferred type |
|---|---|---|
| `order_id` | Unique order identifier | string |
| `customer_id` | Customer identifier used to join to customers | string |
| `order_date` | Date on which the order was placed | date |
| `product_name` | Ordered product | string |
| `category` | Product category | string |
| `quantity` | Number of units ordered | integer |
| `unit_price` | Price per unit | decimal/double |
| `total_amount` | Extended order amount | decimal/double |
| `status` | Order lifecycle status | string |

The included orders span **2020-01-31 through 2026-09-07**. Status values are `Completed`, `Cancelled`, `Returned`, `Shipped`, and `Processing`.

> PySpark uses schema inference in the notebook. For production workloads, define an explicit schema to make parsing and validation deterministic.

## Analysis workflow

The notebook performs the following operations:

1. Creates a Spark session and reads both CSV datasets.
2. Converts `registration_date` to a Spark date.
3. Replaces missing city, state, and country values with `Unknown`.
4. Adds registration year and month features.
5. Measures geographic diversity and identifies the leading locations.
6. Pivots active/inactive customer counts by state.
7. Demonstrates `rank`, `dense_rank`, and `row_number` over state-based windows.
8. Filters customers registered since 2025 and calculates city-level registration ranges.
9. Writes processed and recent-customer datasets to Parquet.
10. Adds order month and inner-joins orders to customers on `customer_id`.
11. Calculates order frequency, total spend, average order value, status totals, monthly volume, and spend rankings.

## Prerequisites

The intended runtime is a Databricks workspace with:

- A cluster or serverless notebook compute resource
- A Databricks Runtime that includes Python and PySpark
- Permission to create or write to a Unity Catalog volume

No extra Python packages are required when using a standard Databricks Runtime.

## Run in Databricks

The notebook currently expects these input paths:

```text
/Volumes/workspace/default/dataset/raw/customers.csv
/Volumes/workspace/default/dataset/raw/orders.csv
```

It writes Parquet output beneath:

```text
/Volumes/workspace/default/dataset/processed_customers
/Volumes/workspace/default/dataset/recent_customers
```

To run it:

1. Create the `workspace.default.dataset` volume if it does not already exist.
2. Create or use its `raw` directory and upload both files from `data/`.
3. Import or open `analytics.ipynb` in Databricks.
4. Attach the notebook to compute.
5. Run the cells from top to bottom.

If your catalog, schema, or volume has a different name, update `output_path` and the two CSV paths in the notebook before running it.

## Run with local PySpark

Local execution is possible with Python, Java, Jupyter, and PySpark installed, but two small adaptations are required:

1. Replace the `/Volumes/.../raw/*.csv` paths with `data/customers.csv` and `data/orders.csv`.
2. Replace Databricks-only calls such as `customers_df.display(5)` with `customers_df.show(5)`.

One example environment setup is:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
jupyter notebook analytics.ipynb
```

Use a Java version supported by the PySpark version you install.

## Generated outputs

The notebook writes two Parquet datasets in overwrite mode:

- `processed_customers`: cleaned customers with registration features and window-ranking columns
- `recent_customers`: the processed subset registered on or after `2025-01-01`

Because overwrite mode replaces the target dataset, use a separate path if existing output must be preserved.

## Notes and limitations

- The customer-order join is an inner join, so customers without matching orders and orders without matching customers are excluded.
- Month analysis groups by month number across all years; add `year(order_date)` when year-over-year seasonality matters.
- Currency is not specified in the source data, so monetary results should be treated as currency-neutral.
- The average-spend result is currently aliased as `total_spend`; renaming it to `average_order_value` would make downstream use clearer.
- The `top_products` line stores a reference to the DataFrame `count` method and does not yet implement product analysis.
- The checked-in source CSV files are intentionally not ignored so the notebook remains reproducible.
