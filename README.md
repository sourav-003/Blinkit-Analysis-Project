# 📊 Dcluttr - Data Analyst Task

This project was completed as part of a Data Analyst task for Dcluttr. The goal was to create a SQL-based data pipeline to analyze inventory and sales data for Blinkit dark stores across cities.

---

## 🚀 Project Objective

The objective was to generate a **derived insights table (`blinkit_city_insights`)** using complex SQL queries. The output table had to include metrics such as estimated quantity sold, sales values, and store availability across cities and dates.

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
