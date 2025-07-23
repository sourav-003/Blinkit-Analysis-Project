# Quick-Commerce Operational Data Analysis (Blinkit)
---

## Project Overview
This project provides a comprehensive operational data analysis for a quick-commerce platform, drawing insights from sales, product, and inventory data. Leveraging advanced SQL, the project focuses on ensuring data integrity, identifying key performance indicators, and extracting actionable intelligence to optimize business operations, inventory management, and sales strategies within the fast-paced quick-commerce environment.
---

## Dataset Description
The analysis utilizes a detailed dataset covering various aspects of Blinkit's operations, including:
* **Date & City Information**: Tracks operational data by specific dates and cities, allowing for geographical and temporal analysis.
* **Product Information (SKUs)**: Includes `sku_id`, `sku_name`, `brand`, `category_id`, and `category_name` for granular product-level insights.
* **Sales & Pricing Data**: Contains `est_qty_sold` (estimated quantity sold), `sp` (selling price), `mrp` (maximum retail price), `est_sales_sp` (estimated sales based on selling price), and `est_sales_mrp` (estimated sales based on MRP).
* **Inventory & Availability Metrics**: Features `wt_osa` (weighted on-shelf availability) and `wt_osa_ls` (weighted on-shelf availability for last scan), crucial for assessing product availability and stock health.
---

## Detailed Analyses & Key Features

This project executes a series of robust SQL analyses to ensure data quality and derive critical business insights:

1.  **Database & Schema Creation**:
    * Designed and implemented a normalized **MySQL database schema** (`dcluttr` database) to efficiently store raw operational data, including `all_blinkit_category_scraping_stream` and `blinkit_categories` tables.
    * Facilitated **bulk data loading** from CSV files (`Final Output.csv`) into the respective database tables, ensuring efficient data ingestion.

2.  **Data Validation & Sanity Checks**:
    * **Duplicate Record Identification**: Implemented queries to identify and address duplicate entries based on `date`, `city_name`, and `sku_id`, ensuring data uniqueness.
    * **Sales Value Sanity Check**: Performed detailed validation comparing estimated sales (based on `est_qty_sold` * `sp`/`mrp`) against recorded sales values (`est_sales_sp`, `est_sales_mrp`) to flag discrepancies.
    * **Price Range Verification**: Conducted checks on `min(sp)`, `max(sp)`, `min(mrp)`, and `max(mrp)` to identify any unrealistic or erroneous price entries.
    * **Inventory Health (`wt_osa`) Validation**: Verified the ranges and consistency of `wt_osa` and `wt_osa_ls` metrics, which are crucial indicators of product availability and inventory accuracy.

3.  **Core Business Analytics**:
    * **Top-Selling SKUs Identification**: Employed **complex SQL queries** involving `SUM()` and `GROUP BY` to pinpoint the top-performing products (SKUs) by total quantity sold and total sales, providing insights into product popularity and revenue drivers.
    * **Geographical & Temporal Coverage Analysis**: Examined `date` coverage and `city` spread within the dataset to understand the scope and completeness of the collected operational data.
    * **Operational Performance Insights**: Analyzed various metrics to understand the efficiency of product listing, delivery, and overall quick-commerce operations.
---

## Technologies Used
* **SQL (MySQL)**: Utilized for database schema creation, data ingestion, complex analytical querying, data validation, and business metric computation.
---

## Business Impact & Outcomes
This project's detailed analysis provides invaluable intelligence for quick-commerce stakeholders:
* **Improved Data Reliability**: Through rigorous validation and sanity checks, the project ensures the underlying data is accurate and trustworthy for reporting and decision-making.
* **Optimized Inventory Management**: Identifying top-selling SKUs and monitoring on-shelf availability (`wt_osa`) can directly inform stocking decisions, reduce stockouts, and minimize carrying costs.
* **Enhanced Sales Strategy**: Understanding sales patterns, pricing impacts, and geographical performance enables targeted marketing campaigns and localized operational adjustments.
* **Foundation for Further Analysis**: The clean, validated, and insight-rich dataset serves as a robust foundation for advanced analytics, predictive modeling, and dashboarding initiatives.
---

## Project Files (Conceptual)
* `Dcluttr Blinkit Analysis Sql Script.sql`: Contains all SQL DDL (Data Definition Language) for schema creation, DML (Data Manipulation Language) for data loading, and DQL (Data Query Language) for analytical queries and validation checks.
* `Final Output.csv`: The primary dataset used for the analysis.
* `Dcluttr-Data Analyst Task.txt`: Original task description or additional notes related to the project objectives and methodology.
---

## Getting Started
To run this project:
1.  **Set up MySQL**: Ensure you have a MySQL server running and accessible.
2.  **Database Creation**: Execute the `CREATE DATABASE blinkit;` command.
3.  **Load Data**: Update the `LOAD DATA INFILE` path in the SQL script (`Dcluttr Blinkit Analysis Sql Script.sql`) to point to the location of your `Final Output.csv` file.
4.  **Execute SQL Script**: Run the entire `Dcluttr Blinkit Analysis Sql Script.sql` to create tables, load data, and perform all defined analyses.
5.  **Explore Insights**: Review the results of the various `SELECT` statements in your MySQL client to observe the derived insights and validation outcomes.

---

## 📂 Data Sources

The task required working with three raw datasets:

| Table Name | Description |
|------------|-------------|
| `all_blinkit_category_scraping_stream` | Contains raw SKU-level data with timestamps, pricing, and inventory across dark stores. |
| `blinkit_categories` | Contains mapping between category IDs and category/subcategory names. |
| `blinkit_city_map` | Maps each dark store ID to a specific city. |

---

## 🏗️ Database Setup

Create and use a local MySQL database named `dcluttr`, then execute the provided schema and data import queries.

```sql
CREATE DATABASE dcluttr;
USE dcluttr;
-- See `Dcluttr-Data Analyst Task.txt` for full schema and inserts
```
**Author:** Sourav Kumar
