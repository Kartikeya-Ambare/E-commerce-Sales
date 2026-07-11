# E-commerce Sales Analysis (Databricks)

A PySpark + SQL pipeline built in Databricks that cleans a raw online retail transaction dataset, builds a set of aggregated analytics tables, and saves them to Unity Catalog for downstream use in dashboards and Genie Spaces.

## Overview

This notebook processes the classic **Online Retail** dataset (UK-based online retailer, worldwide customers) and produces:

- A cleaned, enriched transactions table
- Sales aggregations by month, quarter, and country
- Product performance rankings
- Customer-level summaries (spend, order count, recency)
- A library of ready-to-use SQL analysis queries (revenue trends, market share, customer segmentation, loyalty, UK vs. international split)
- Two Unity Catalog schemas containing all base and analysis tables

## Data Source

| | |
|---|---|
| **Input path** | `/Volumes/workspace/default/ecommerece/OnlineRetail.csv` |
| **Format** | CSV, header + inferred schema |
| **Key columns** | `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country` |

## Pipeline Steps

### 1. Data Cleaning (`orders_cleaned`)
- Removes rows with non-positive `Quantity` or `UnitPrice`
- Drops rows with null `CustomerID` or `Description` (returns/cancellations and bad records)
- Parses `InvoiceDate` into a proper timestamp (`M/d/yyyy H:mm`)
- Derives `TotalPrice` (`Quantity × UnitPrice`), plus `Year`, `Month`, and `Quarter`

### 2. Aggregation Tables
| Table | Grain | Key metrics |
|---|---|---|
| `monthly_sales_by_country` | Year, Month, Country | total_revenue, order_count, avg_order_value |
| `quarterly_sales_by_country` | Year, Quarter, Country | total_revenue, order_count, avg_order_value, total_quantity |
| `country_sales` | Country | total_revenue, total_orders, unique_customers, total_quantity_sold, avg/min/max order value |
| `top_products` | StockCode, Description | total_quantity_sold, total_revenue, times_ordered, avg_unit_price |
| `customer_summary` | CustomerID | total_orders, total_spent, avg_order_value, total_items_purchased, first/last order date |

### 3. Unity Catalog Storage
All tables are written twice (overwrite mode):
- `workspace.default.*` — initial working schema
- `workspace.ecommerce_analytics.*` — dedicated schema created for this project, holding both the **base tables** above and a set of **analysis tables** (prefixed `analysis_`) generated from the SQL queries below

### 4. SQL Analyses
The notebook includes `%sql` cells for:
- **Revenue Trend** — monthly revenue, order count, and average order value over time
- **Top 10 Countries by Revenue** — with market share percentage
- **Top 20 Best-Selling Products** — by revenue, with revenue-per-order
- **Customer Segmentation** — High / Medium / Low value tiers based on total spend
- **Quarterly Performance** — revenue, orders, and countries served per quarter
- **Customer Loyalty** — one-time, occasional, regular, and loyal buyer cohorts with revenue contribution %
- **UK vs. International Revenue** — monthly split and international revenue share

## Requirements

- Databricks workspace with Unity Catalog enabled
- A Unity Catalog volume containing `OnlineRetail.csv` at the path above (update `path` if different)
- Cluster/runtime with PySpark and Spark SQL (standard Databricks runtime)

## Usage

1. Upload `OnlineRetail.csv` to the Unity Catalog volume referenced in `path`.
2. Run all cells top to bottom in a Databricks notebook.
3. Query the resulting tables under `workspace.ecommerce_analytics` for dashboards, Genie Spaces, or further analysis.

## Output Tables (`workspace.ecommerce_analytics`)

**Base tables (6):** `orders_cleaned`, `monthly_sales_by_country`, `quarterly_sales_by_country`, `country_sales`, `top_products`, `customer_summary`

**Analysis tables (up to 6):** `analysis_monthly_revenue_trends`, `analysis_top_countries`, `analysis_customer_segmentation`, and others derived from the SQL queries above (the notebook's final analysis-table-creation cell is truncated/incomplete — see Notes).

## Notes

- The last code cell (creating `analysis_customer_segmentation` and any subsequent analysis tables) appears to be **cut off mid-query** in the source notebook — verify and complete it before relying on the full set of `analysis_*` tables.
- `count(col("CustomerID"))` in `country_sales` counts non-null CustomerIDs per country rather than distinct customers; treat `unique_customers` as an approximation unless changed to `countDistinct`.
