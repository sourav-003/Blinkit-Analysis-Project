create database dcluttr;
use dcluttr;
CREATE TABLE all_blinkit_category_scraping_stream (
    created_at DATETIME,
    l1_category_id INT,
    l2_category_id INT,
    store_id INT,
    sku_id INT,
    sku_name TEXT,
    selling_price FLOAT,
    mrp FLOAT,
    inventory INT,
    image_url TEXT,
    brand_id INT,
    brand TEXT,
    unit VARCHAR(50)
);
LOAD DATA INFILE "C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/all_blinkit_category_scraping_stream.csv"
INTO TABLE all_blinkit_category_scraping_stream
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"' 
LINES TERMINATED BY '\n' 
IGNORE 1 ROWS
(created_at, l1_category_id, l2_category_id, store_id, sku_id, sku_name, selling_price, mrp, inventory, image_url, brand_id, brand, unit);

CREATE TABLE blinkit_categories (
    l1_category_name VARCHAR(100),
    l1_category_id INT,
    l2_category_name VARCHAR(100),
    l2_category_id INT
);
INSERT INTO blinkit_categories (l1_category_name, l1_category_id, l2_category_name, l2_category_id) VALUES
('Munchies', 1237, 'Bhujia & Mixtures', 1178),
('Munchies', 1237, 'Papad & Fryums', 80);


CREATE TABLE blinkit_city_map (
    dark_store_id INT,
    city VARCHAR(100)
);
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/blinkit_city_map.csv'
INTO TABLE blinkit_city_map
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"' 
LINES TERMINATED BY '\n' 
IGNORE 1 ROWS
(dark_store_id, city);

-- Step 1: Calculate Inventory Change Over Time
CREATE TEMPORARY TABLE inventory_diff AS
SELECT
    created_at,
    sku_id,
    store_id,
    inventory,
    selling_price,
    mrp,
    LAG(inventory) OVER (PARTITION BY sku_id, store_id ORDER BY created_at) AS prev_inventory
FROM all_blinkit_category_scraping_stream;
SELECT* FROM inventory_diff; 

-- Step 2: Estimate Quantity Sold
CREATE TEMPORARY TABLE est_sales_base AS
SELECT
    DATE(created_at) AS date,
    sku_id,
    store_id,
    selling_price,
    mrp,
    CASE 
        WHEN prev_inventory IS NOT NULL AND prev_inventory > inventory THEN prev_inventory - inventory
        ELSE 0
    END AS est_qty_sold
FROM inventory_diff;
SELECT* from est_sales_base;

-- Step 3: Join with blinkit_city_map
CREATE TEMPORARY TABLE sales_with_city AS
SELECT 
    e.*, 
    c.city AS city_name
FROM est_sales_base e
JOIN blinkit_city_map c 
    ON e.store_id = c.dark_store_id;
select* from sales_with_city;

-- Join with Category Info    
CREATE TEMPORARY TABLE enriched_sales AS
SELECT 
    s.date,
    s.sku_id,
    s.store_id,
    s.selling_price,
    s.mrp,
    s.est_qty_sold,
    s.city_name,
    acs.sku_name,
    acs.brand,
    acs.brand_id,
    acs.image_url,
    acs.unit,
    acs.l1_category_id,
    acs.l2_category_id,
    bc.l1_category_name AS category_name,
    bc.l2_category_name AS sub_category_name
FROM sales_with_city s
JOIN all_blinkit_category_scraping_stream acs 
    ON s.sku_id = acs.sku_id 
    AND s.store_id = acs.store_id 
    AND DATE(acs.created_at) = s.date
JOIN blinkit_categories bc 
    ON acs.l1_category_id = bc.l1_category_id 
   AND acs.l2_category_id = bc.l2_category_id
LIMIT 10000;
select* from enriched_sales;

-- Step 5: Aggregate Metrics Per SKU, Date, City
CREATE TABLE blinkit_city_insights AS
SELECT
    date,
    city_name,
    sku_id,
    sku_name,
    brand,
    brand_id,
    image_url,
    unit,
    l1_category_id AS category_id,
    category_name,
    l2_category_id AS sub_category_id,
    sub_category_name,
    
    -- Core metrics
    SUM(est_qty_sold) AS est_qty_sold,
    MAX(selling_price) AS sp,
    MAX(mrp) AS mrp,
    SUM(est_qty_sold * selling_price) AS est_sales_sp,
    SUM(est_qty_sold * mrp) AS est_sales_mrp,
    
    -- Count of unique dark stores where SKU was listed (could include out-of-stock)
    COUNT(DISTINCT store_id) AS listed_ds_count
    
FROM enriched_sales
GROUP BY
    date, city_name, sku_id, sku_name, brand, brand_id, image_url, unit,
    l1_category_id, category_name, l2_category_id, sub_category_name;

SELECT COUNT(DISTINCT store_id) AS ds_count FROM all_blinkit_category_scraping_stream;

ALTER TABLE blinkit_city_insights ADD COLUMN ds_count INT;
ALTER TABLE blinkit_city_insights ADD COLUMN wt_osa FLOAT;
ALTER TABLE blinkit_city_insights ADD COLUMN wt_osa_ls FLOAT;

UPDATE blinkit_city_insights
SET 
    ds_count = 956, 
    wt_osa = listed_ds_count / 956,
    wt_osa_ls = listed_ds_count / listed_ds_count; 
SELECT * FROM blinkit_city_insights;

SELECT COUNT(*) FROM blinkit_city_insights;

SELECT date, sku_id, city_name, COUNT(*) 
FROM blinkit_city_insights
GROUP BY date, sku_id, city_name
HAVING COUNT(*) > 1;

SELECT * FROM blinkit_city_insights
ORDER BY RAND()
LIMIT 10;

-- Summary by Date/City (Sanity Check)
SELECT date, city_name, COUNT(*) AS sku_count
FROM blinkit_city_insights
GROUP BY date, city_name
ORDER BY sku_count DESC;

 -- Uniqueness Check for Keys
SELECT date, city_name, sku_id, COUNT(*) AS cnt
FROM blinkit_city_insights
GROUP BY date, city_name, sku_id
HAVING cnt > 1;

-- Sanity Check on Sales Values
SELECT 
    est_qty_sold,
    sp,
    mrp,
    ROUND(est_qty_sold * sp, 2) AS expected_sales_sp,
    ROUND(est_qty_sold * mrp, 2) AS expected_sales_mrp,
    est_sales_sp,
    est_sales_mrp
FROM blinkit_city_insights
LIMIT 20;

-- Check Price Range
SELECT 
    MIN(sp) AS min_sp, MAX(sp) AS max_sp,
    MIN(mrp) AS min_mrp, MAX(mrp) AS max_mrp
FROM blinkit_city_insights;

-- Verify wt_osa and wt_osa_ls Ranges
SELECT 
    MIN(wt_osa) AS min_osa,
    MAX(wt_osa) AS max_osa,
    MIN(wt_osa_ls) AS min_osa_ls,
    MAX(wt_osa_ls) AS max_osa_ls
FROM blinkit_city_insights;

-- Top-Selling SKUs (Validation)
SELECT sku_name, SUM(est_qty_sold) AS total_qty, SUM(est_sales_sp) AS total_sales
FROM blinkit_city_insights
GROUP BY sku_name
ORDER BY total_sales DESC
LIMIT 10;

-- Date Coverage
SELECT DISTINCT date FROM blinkit_city_insights ORDER BY date;

-- City Spread
SELECT city_name, COUNT(*) AS sku_count
FROM blinkit_city_insights
GROUP BY city_name
ORDER BY sku_count DESC;

-- Listed DS Count vs. DS Count
SELECT 
    listed_ds_count, 
    ds_count, 
    ROUND(100 * listed_ds_count / ds_count, 2) AS listed_ratio_pct
FROM blinkit_city_insights
ORDER BY listed_ratio_pct DESC
LIMIT 10;

-- Most Common Categories
SELECT category_name, COUNT(*) AS sku_count
FROM blinkit_city_insights
GROUP BY category_name
ORDER BY sku_count DESC;































