# ✈️ Airlines Data Analysis Project Using SQL and Power BI
## 📌 1. Project Overview -
This project analyzes an airline dataset of **300,000+ flight records** to uncover business insights using **PostgreSQL** and visualize them with **Power BI**.  
The dataset includes flight details such as airline, route, stops, duration, ticket class, price, and booking lead time.


## 🎯 2. Objectives
- Perform **data cleaning and transformation** in SQL.
- Write **50+ SQL business queries** (basic to advanced).
- Build **analytical views** in PostgreSQL for optimized BI reporting.
- Apply **Window Functions** (LAG, LEAD, RANK, NTILE, etc.).
- Create **Views** for BI-ready datasets.
- Build an **interactive Power BI dashboard** for decision-making

---
## 📂 3. Dataset Description
| Column Name       | Type      | Description                             |
|-------------------|-----------|-----------------------------------------|
| indexs            | int       | Indexing(0,1,2,3,.....)                 |
| airline           | text      | Airline name                            |
| flight            | text      | Flight number/code                      |
| source_city       | text      | Departure city                          |
| departure_time    | text      | Departure time category                 |
| stops             | text      | Stops (Non-stop, 1 stop, etc.)          |
| arrival_time      | text      | Arrival time category                   |
| destination_city  | text      | Arrival city                            |
| classes           | text      | Ticket class (Economy, Business)        |
| duration          | float     | Flight duration in hours                |
| days_left         | int       | Days between booking and travel         |
| price             | float     | Ticket price (in INR)                   |

---
## 🧮 SQL Analysis

## 📜 4. SQL Work Summary
* **Basic Queries**: Total flights, avg. price per airline, etc.
* **Intermediate Queries**: Price trends by booking days, duration vs. price.
* **Advanced Queries**: Rank routes, moving averages, seasonal demand.
* **Window Functions**: RANK, DENSE_RANK, LAG, LEAD, NTILE.
* **Views**: Pre-aggregated BI tables for fast reporting.
---

-- Create Database & Tables
```sql
DROP DATABASE IF EXISTS AirlinesDataset;
CREATE DATABASE AirlinesDataset;
```
---
```sql
DROP TABLE IF EXISTS airlines;
CREATE TABLE airlines(
indexs INT PRIMARY KEY,
airline VARCHAR(30),
flight VARCHAR(30),
source_city VARCHAR(20),
departure_time VARCHAR(25),
stops VARCHAR(15),
arrival_time VARCHAR(20),
destination_city VARCHAR(15),
classes VARCHAR(15),
duration DECIMAL(10,2),
days_left INT,
price DECIMAL(10,2)
);
```
---
## BUSINESS QUERIES

-- Data Exploration & Cleaning(if required) --

```sql
-- Check How Many Rows in the Dataset
SELECT COUNT(*) FROM airlines;

-- Get Table Schema (Column Names & Data Types)
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'airlines';

-- Count of NULLs per COLUMN
SELECT 
  COUNT(*) FILTER (WHERE indexs IS NULL) AS null_indexs,
  COUNT(*) FILTER (WHERE airline IS NULL) AS null_airline,
  COUNT(*) FILTER (WHERE flight IS NULL) AS null_flight,
  COUNT(*) FILTER (WHERE source_city IS NULL) AS null_source_city,
  COUNT(*) FILTER (WHERE departure_time IS NULL) AS null_dep_tym,
  COUNT(*) FILTER (WHERE stops IS NULL) AS null_stops,
  COUNT(*) FILTER (WHERE arrival_time IS NULL) AS null_arr_tym,
  COUNT(*) FILTER (WHERE destination_city IS NULL) AS null_dst_city,
  COUNT(*) FILTER (WHERE classes IS NULL) AS null_class,
  COUNT(*) FILTER (WHERE duration IS NULL) AS null_duration,
  COUNT(*) FILTER (WHERE days_left IS NULL) AS null_days,
  COUNT(*) FILTER (WHERE price IS NULL) AS null_price
  FROM airlines;
```
-- Check Summary Statistics for Numeric Columns (if required)

```sql
-- Q1. List all distinct airlines
SELECT DISTINCT(airline) FROM airlines;

-- Q2. Count total flights
SELECT COUNT(*) FROM airlines;

-- Q3. Total Flights by source city
SELECT source_city,COUNT(*) AS total_flights FROM airlines 
GROUP BY source_city;

-- Q4. Total Flights by destination city
SELECT destination_city, COUNT(*) AS total_flights FROM airlines 
GROUP BY 1;

-- Q5. Flight count per airline
SELECT airline,COUNT(*) AS Total_flight_count FROM airlines
GROUP BY airline;

-- Q6. Class-wise ticket distribution
SELECT  classes, COUNT(*) AS Total_class
FROM airlines 
GROUP BY 1;

-- Q7. Top 5 most common routes
SELECT source_city, destination_city, COUNT(*) AS total_flights
FROM airlines
GROUP BY 1,2
ORDER BY total_flights DESC
LIMIT 5; 

-- Q8. Average ticket price by class
SELECT classes, ROUND(AVG(price),2) FROM airlines
GROUP BY 1;

-- Q9. Minimum and maximum price
SELECT classes, MIN(price) AS min_price,
MAX(price) AS max_price 
FROM airlines
GROUP BY classes;

-- Q10. How Many Flights with non-stop route
SELECT airline,flight,COUNT(*) AS non_stop_flights FROM airlines
WHERE stops = 'zero'
GROUP BY 1,2;

-- Q11. How many Flights with 1 or more stops
SELECT airline,flight,COUNT(*) AS total_flights FROM airlines
WHERE stops <> 'zero'
GROUP BY 1,2;

-- Q12. Flights departing from a specific city with the specific time(e.g., morning)
SELECT * FROM airlines 
WHERE source_city = 'Mumbai' 
AND departure_time = 'Evening';

-- Q13. Flights arriving from a specific city with the specific time(e.g., morning)
SELECT * FROM airlines 
WHERE source_city = 'Kolkata'
AND departure_time = 'Morning';

-- Q14 Total Revenue per Airlines
SELECT airline, SUM(price) AS revenue FROM airlines
GROUP BY airline
ORDER BY revenue ;
```
---
-- Intermediate SQL Queries --
---
```sql
-- Q1. Average price per route
SELECT source_city, destination_city, 
ROUND(AVG(price),2) AS avg_price
FROM airlines
GROUP BY source_city, destination_city;

-- Q2. Average days left before departure
SELECT FLOOR(AVG(days_left)) FROM airlines;

-- Q3. Class-wise average days left
SELECT classes, FLOOR(AVG(days_left)) 
FROM airlines 
GROUP BY classes;

-- Q4. Routes with average price above 10,000
SELECT source_city, destination_city, 
ROUND(AVG(price),2) AS avg_price
FROM airlines
GROUP BY source_city, destination_city
HAVING AVG(price) > 10000;

-- Q5. Flight count per stop type
SELECT stops, COUNT(*) FROM airlines 
GROUP BY stops;

-- Q6. Average price per stop type
SELECT stops, ROUND(AVG(price),2) AS avg_price FROM airlines 
GROUP BY stops;

-- Q7. Top 5 most expensive flights
SELECT airline,price FROM airlines 
ORDER BY price DESC 
LIMIT 5;

-- Q8. Flights with more than 15 days left and price < 5000
SELECT * FROM airlines 
WHERE days_left > 15
AND price < 5000;

-- Q9. Revenue by source city
SELECT source_city, 
SUM(price) AS Revenue
FROM airlines 
GROUP BY source_city
ORDER BY Revenue;

-- Q10. Revenue by destination city
SELECT destination_city, 
SUM(price) AS revenue
FROM airlines 
GROUP BY destination_city
ORDER BY revenue;

-- Q11. Route profitability ranking
SELECT source_city, destination_city, 
SUM(price) AS total_revenue
FROM airlines
GROUP BY source_city, destination_city
ORDER BY total_revenue DESC;

-- Q12. Flights by time of day
SELECT departure_time, 
COUNT(*) 
FROM airlines 
GROUP BY departure_time;

-- Q13. Flight distribution by class and stop type
SELECT classes, stops, COUNT(*) AS total_distribution
FROM airlines 
GROUP BY classes, stops
ORDER BY total_distribution;

-- Q14. Price range per airline
SELECT airline, 
MIN(price) AS min_price, 
MAX(price) AS max_price 
FROM airlines 
GROUP BY airline;

-- Q15. Average duration per route
SELECT source_city, destination_city, 
ROUND(AVG(duration),2) AS avg_hours
FROM airlines
GROUP BY source_city, destination_city
ORDER BY avg_hours;

-- Q16. Flights within a specific price range
SELECT airline,price FROM airlines 
WHERE price BETWEEN 20000 AND 40000;

-- Q17. Flights grouped by days left buckets
SELECT
  CASE
    WHEN days_left <= 7 THEN '0-7 Days'
    WHEN days_left <= 15 THEN '8-15 Days'
    WHEN days_left <= 30 THEN '16-30 Days'
    ELSE '30+ Days'
  END AS days_left_bucket,
  COUNT(*)
FROM airlines
GROUP BY days_left_bucket;

-- Q18. Airline with highest average ticket price
SELECT airline, ROUND(AVG(price),2) AS avg_price 
FROM airlines 
GROUP BY airline 
ORDER BY avg_price DESC LIMIT 1;
```
---
-- Advanced SQL Queries --
---

```sql
-- Q1. Rank flights by price per route 
SELECT *,
  RANK() OVER (PARTITION BY source_city, destination_city ORDER BY price DESC) AS price_rank
FROM airlines;

-- Q2. Running total revenue per airline
SELECT airline, price,
  SUM(price) OVER (PARTITION BY airline ORDER BY indexs) AS running_total
FROM airlines;

-- Q3. Moving average of price (last 3 flights by airline)
SELECT airline,price,
  ROUND(AVG(price) OVER (
PARTITION BY airline ORDER BY indexs ROWS BETWEEN 2 PRECEDING AND CURRENT ROW),2) AS moving_avg_price
FROM airlines;

-- Q4. Compare price of non-stop vs 1-stop flights
SELECT stops, ROUND(AVG(price),2) FROM airlines 
WHERE stops IN ('zero', 'one', 'two_or_more') 
GROUP BY stops;

-- Q5. Time-based price analysis (day buckets vs price)
SELECT
  CASE
    WHEN departure_time IN ('Early Morning', 'Morning') THEN 'Morning'
    WHEN departure_time IN ('Afternoon', 'Evening') THEN 'Daytime'
    ELSE 'Night'
  END AS time_bucket,
  ROUND(AVG(price),2) AS avg_price
FROM airlines
GROUP BY time_bucket;

-- Q6. Rank cheapest flights per airline
SELECT airline,price,
       RANK() OVER (PARTITION BY airline ORDER BY price ASC) AS price_rank
FROM airlines;

-- Q7. Dense rank of flight duration per route
SELECT source_city,destination_city,duration,
       DENSE_RANK() OVER (PARTITION BY source_city, destination_city ORDER BY duration) AS duration_rank
FROM airlines;

-- Q8.  Compare current price with previous flight on same route --[LAG()>value in previous row]
SELECT airline,source_city,destination_city,days_left,price,
       price - LAG(price) OVER (
PARTITION BY source_city, destination_city ORDER BY days_left DESC) AS price_diff_prev
FROM airlines;

-- Q9. Show future price for same route --[LEAD()> value in next row]
SELECT airline,source_city,destination_city,days_left,price,
       LEAD(price) OVER (PARTITION BY source_city, destination_city ORDER BY days_left ASC) AS future_price
FROM airlines;

-- Q10. Divide all flights into 4 price quartiles --[NTILE(n)-Divides rows into n equal buckets]
SELECT airline,source_city,destination_city,days_left,price,stops,
       NTILE(4) OVER (ORDER BY price) AS price_quartile
FROM airlines;

-- Q11. Show first and last recorded price for each route --[FIRST_VALUE() | LAST_VALUE()]
SELECT airline,source_city,destination_city,days_left,price,
       FIRST_VALUE(price) OVER (
PARTITION BY source_city, destination_city ORDER BY days_left DESC) AS first_price,
       LAST_VALUE(price) OVER (
PARTITION BY source_city, destination_city
ORDER BY days_left DESC ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS last_price
FROM airlines;

-- Q12.Show 3rd cheapest flight per airline --[NTH_VALUE() Returns the nth value in the window]
SELECT airline,source_city,destination_city,days_left,price,
       NTH_VALUE(price, 3) OVER (
PARTITION BY airline ORDER BY price ASC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS third_cheapest
FROM airlines;

-- Q13. Flight count per route visible on each row
SELECT flight,source_city, destination_city,
       COUNT(*) OVER (PARTITION BY source_city, destination_city) AS total_flights_on_route
FROM airlines;

-- Q14. flight moving average per airline
SELECT *,
       AVG(price) OVER (
           PARTITION BY airline
           ORDER BY indexs
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg_3
FROM airlines;
```
---
-- BONUS --
---
```sql
--Create View for Airlines Analysis--
CREATE OR REPLACE VIEW vw_airlines_analysis AS
SELECT
    indexs,
    airline,
    flight,
    source_city,
    destination_city,
    departure_time,
    arrival_time,
    stops,
    classes,
    duration,
    days_left,
    price,

    -- 1. Row number per route
    ROW_NUMBER() OVER (PARTITION BY source_city, destination_city ORDER BY price) AS route_row_number,

    -- 2. Rank of price per airline
    RANK() OVER (PARTITION BY airline ORDER BY price ASC) AS price_rank_airline,

    -- 3. Dense rank of duration per route
    DENSE_RANK() OVER (PARTITION BY source_city, destination_city ORDER BY duration) AS duration_rank_route,

    -- 4. Price change from previous flight on same route (LAG)
    price - LAG(price) OVER (PARTITION BY source_city, destination_city ORDER BY days_left DESC) AS price_change_prev,

    -- 5. Price in the next flight for same route (LEAD)
    LEAD(price) OVER (PARTITION BY source_city, destination_city ORDER BY days_left ASC) AS price_next,

    -- 6. Quartile based on price
    NTILE(4) OVER (ORDER BY price) AS price_quartile,

    -- 7. First & Last price recorded per route
    FIRST_VALUE(price) OVER (PARTITION BY source_city, destination_city ORDER BY days_left DESC) AS first_price_route,
    LAST_VALUE(price) OVER (
        PARTITION BY source_city, destination_city 
        ORDER BY days_left DESC 
        ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
    ) AS last_price_route,

    -- 8. 3rd cheapest flight per airline
    NTH_VALUE(price, 3) OVER (
        PARTITION BY airline 
        ORDER BY price ASC 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS third_cheapest_airline,

    -- 9. Cumulative revenue per airline
    SUM(price) OVER (PARTITION BY airline ORDER BY indexs) AS cumulative_revenue_airline,

    -- 10. Average price per route
    AVG(price) OVER (PARTITION BY source_city, destination_city) AS avg_price_route,

    -- 11. Number of flights per route
    COUNT(*) OVER (PARTITION BY source_city, destination_city) AS flights_count_route,

    -- 12. Moving average of price (last 3 flights by airline)
    AVG(price) OVER (
        PARTITION BY airline 
        ORDER BY indexs 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_price_3,

    -- 13. Price deviation from average route price
    price - AVG(price) OVER (PARTITION BY source_city, destination_city) AS price_deviation

FROM airlines;
```
-- Call View Table
```sql
-- Selecting window function columns from a view
SELECT airline, flight, price, price_rank_airline
FROM vw_airlines_analysis
WHERE price_rank_airline <= 5;  -- Top 5 cheapest flights per airline
```
---
-- CODE END --
---



  





