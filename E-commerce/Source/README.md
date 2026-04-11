# Olist E-Commerce Analytics — Power BI + PostgreSQL

An end-to-end data analytics project built on the [Olist Brazilian E-Commerce dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) from Kaggle. Raw CSV data is loaded into PostgreSQL, transformed via SQL views, and visualized in Power BI using DirectQuery.

---

## Project Structure

```
├── archive/                        # Raw CSV datasets from Kaggle
├── sql/
│   ├── 01_Tables.sql               # CREATE TABLE statements (9 tables)
│   ├── 02_ForeignKeys.sql          # Foreign key constraints
│   ├── 03_Indexes.sql              # Performance indexes
│   ├── 04_LoadCSV.sql              # COPY commands to load CSVs
│   └── 05_BIViews.sql              # BI SQL views for Power BI
├── powerbi/
│   ├── ecom.pbix                   # Power BI report file
│   ├── DAX Measures.dax            # All DAX measures
│   ├── DateDim.dax                 # DimDate calculated table
│   └── Report Views.txt            # Page/visual layout guide
└── README.md
```

---

## Prerequisites

- [PostgreSQL](https://www.postgresql.org/download/) + pgAdmin 4
- [Power BI Desktop](https://powerbi.microsoft.com/desktop/)
- Olist dataset CSVs extracted to your local machine

---

## Setup Guide

### 1. Create the Database

In pgAdmin: right-click Databases → Create → Database → name it `olist_db`

### 2. Create Tables

Open the Query Tool in `olist_db` and run the contents of `project assets/Tables.txt`.

This creates 9 raw tables:

| Table | Description |
|---|---|
| `olist_customers_dataset` | Customer info and location |
| `olist_geolocation_dataset` | Zip code lat/lng mapping |
| `olist_products_dataset` | Product dimensions and category |
| `olist_sellers_dataset` | Seller info and location |
| `olist_orders_dataset` | Order lifecycle and timestamps |
| `olist_order_items_dataset` | Line items per order |
| `olist_order_payments_dataset` | Payment method and value |
| `olist_order_reviews_dataset` | Customer review scores |
| `product_category_name_translation` | Portuguese → English category names |

### 3. Load CSV Data

Run the COPY commands from `project assets/CopyCSV.txt`, updating the path to match your machine. Example:

```sql
COPY olist_customers_dataset
FROM 'E:/Downloads/Source-20260410T034446Z-3-001/Source/archive/olist_customers_dataset.csv'
DELIMITER ',' CSV HEADER;
```

> If you get a permission error, use `\COPY` instead of `COPY` — it runs as your Windows user rather than the postgres service.

Repeat for all 9 CSVs. The geolocation table is large and split into 11 SQL part files — run each one from `archive/olist_geolocation_dataset_sql/`.

### 4. Add Foreign Keys

Run `project assets/ForeignKey.txt` to link tables:

- `olist_orders_dataset` → `olist_customers_dataset`
- `olist_order_items_dataset` → orders, products, sellers
- `olist_order_payments_dataset` → `olist_orders_dataset`
- `olist_order_reviews_dataset` → `olist_orders_dataset`

### 5. Create Indexes

Run `project assets/Index.txt` to add performance indexes on join and filter columns (order_id, customer_id, product_id, seller_id, purchase_timestamp, etc.).

### 6. Create BI Views

Run `project assets/BIViews.txt` to create 5 views used by Power BI:

| View | Purpose |
|---|---|
| `bi_dim_product` | Product with English category name and volume |
| `bi_fact_review_latest` | One review per order (latest) |
| `bi_fact_order` | Order with delivery days, late flag, customer info |
| `bi_fact_sales` | Main fact table joining all dimensions |
| `bi_payments_order` | Aggregated payment per order |

`bi_fact_sales` is the primary table loaded in Power BI.

---

## Power BI Setup

### Connect to PostgreSQL

1. Open `project assets/ecom.pbix` in Power BI Desktop
2. If prompted: Get Data → PostgreSQL database
3. Server: `localhost`, Database: `olist_db`, Mode: `DirectQuery`
4. Credentials: your postgres username and password

### DimDate Table

Create a calculated table in Power BI using the DAX from `project assets/DateDim.txt`. Set its storage mode to Import and relate `DimDate[Date]` to `bi_fact_sales[purchase_date]`.

### DAX Measures

All measures are in `project assets/DAX Measures.txt`, organized by report page:

- Base: Revenue, Orders, Customers, AOV, GMV, Avg Rating, On-time %
- Time Intelligence: YTD, YoY %, Rolling 30D Revenue
- Reviews: Positive %, Negative %, Review Bucket
- Payments: Payment Value, Avg Installments
- Logistics: Avg Delivery Days, Late %
- Sellers: Sellers count, Revenue per Seller

---

## Report Pages

| Page | Key Visuals |
|---|---|
| Overview | KPI cards, revenue trend, top categories, map by state |
| Sales Trends | Revenue vs LY, YTD area chart, category matrix |
| Category & Product | Treemap, scatter (AOV vs Orders), top categories table |
| Customer Geo | Filled map, top states/cities by revenue |
| Delivery & Logistics | On-time %, late orders by month, avg delivery days |
| Reviews | Rating distribution, avg rating by category |
| Payments | Payment type donut, installments, payment trend |
| Sellers | Top sellers, revenue per seller, seller map |
| Drillthrough | Category-level deep dive (optional) |

---

## Data Flow

```
CSV Files → PostgreSQL Raw Tables → SQL BI Views → Power BI DirectQuery → DAX Measures → Report
```
