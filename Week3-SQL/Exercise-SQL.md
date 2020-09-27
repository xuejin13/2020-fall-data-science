# SQL: Structured Query Language Exercise

### Getting Started

1. Go to BigQuery UI https://console.cloud.google.com/bigquery
2. Add in the public data sets. 3. Click the Add Data icon 4. Add any dataset 5. `bigquery-public-data` should become visible and populate in the BigQuery UI.
3. Add your queries where it says [YOUR QUERY HERE].
4. Make sure you add your query in between the triple tick marks.

---

For this section of the exercise we will be using the `bigquery-public-data.austin_311.311_service_requests` table.

5. Write a query that tells us how many rows are in the table.

   ```
   SELECT
     COUNT(*)
   FROM
     `bigquery-public-data.austin_311.311_service_requests`
   LIMIT
     1000 
   ```

6. Write a query that tells us how many _distinct_ values there are in the complaint_description column.

   ```
   SELECT
     complaint_description,
     COUNT(*)
   FROM
     `bigquery-public-data.austin_311.311_service_requests`
   GROUP BY
     complaint_description	
   ```

7. Write a query that counts how many times each owning_department appears in the table and orders them from highest to lowest.

   ```
   WITH
     T AS (
     SELECT
       owning_department,
       COUNT(*) AS count
     FROM
       `bigquery-public-data.austin_311.311_service_requests`
     GROUP BY
       owning_department )

   SELECT
     *
   FROM
     T
   ORDER BY 
     count DESC
   ```

8. Write a query that lists the top 5 complaint_description that appear most and the amount of times they appear in this table. (hint... limit)

   ```
   WITH 
     T as (
     SELECT
       complaint_description,
       COUNT(*) as count
     FROM
       `bigquery-public-data.austin_311.311_service_requests`
     GROUP BY 
       complaint_description 
     )
 
   SELECT 
     *
   FROM 
     T
   ORDER BY 
     count DESC 
   LIMIT  
     5
   ```

9. Write a query that lists and counts all the complaint_description, just for the where the owning_department is 'Animal Services Office'.

   ```
   WITH
     T AS (
     SELECT
       complaint_description,
       COUNT(*) AS count
     FROM
       `bigquery-public-data.austin_311.311_service_requests`
     WHERE 
       owning_department in ('Animal Services Office')
     GROUP BY 
       complaint_description  )
   SELECT
     *
   FROM
     T
   ```

10. Write a query to check if there are any duplicate values in the unique_key column (hint.. There are two was to do this, one is to use a temporary table for the groupby, then filter for values that have more than one count, or, using just one table but including the `having` function).
    
   ```
    -- WITH
    --   T AS (
    --   SELECT
    --     unique_key,
    --     COUNT(*) as count
    --   FROM
    --     `bigquery-public-data.austin_311.311_service_requests`
    --   GROUP BY
    --     unique_key
    --   )
    -- SELECT
    --   *
    -- FROM
    --   T
    -- WHERE
    --   count > 1
  
   WITH
     T AS (
   SELECT
     unique_key,
     COUNT(*) AS count,
   FROM
     `bigquery-public-data.austin_311.311_service_requests`
   GROUP BY
     unique_key
   HAVING
     COUNT(unique_key) > 1 
   )

   SELECT
     *
   FROM
     T
   ```

### For the next question, use the `census_bureau_usa` tables.

1. Write a query that returns each zipcode and their population for 2000 and 2010.
   ```
   WITH
     Table2000 AS (
     SELECT
       zipcode,
       SUM(population) as total_population_2000
     FROM
       `bigquery-public-data.census_bureau_usa.population_by_zip_2000`
     GROUP BY
       zipcode
     ),
     Table2010 AS (
     SELECT 
       zipcode,
       SUM(population) as total_population_2010
     FROM 
       `bigquery-public-data.census_bureau_usa.population_by_zip_2010` 
     GROUP BY 
       zipcode 
     )
    
   SELECT
     A.zipcode,
     A.total_population_2000,
     B.total_population_2010  
   FROM
     Table2000 as A 
     JOIN 
     Table2010 as B
     ON A.zipcode = B.zipcode 
   ```

### For the next section, use the `bigquery-public-data.google_political_ads.advertiser_weekly_spend` table.

1. Using the `advertiser_weekly_spend` table, write a query that finds the advertiser_name that spent the most in usd.
   ```
   WITH 
     T as (
     SELECT
       advertiser_name,
       SUM(spend_usd) as usd,
     FROM
       `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
     GROUP BY 
       advertiser_name 
     )
  
   SELECT 
     *
   FROM 
     T
   ORDER BY 
     T.usd DESC
   LIMIT 1
   ```
2. Who was the 6th highest spender? (No need to insert query here, just type in the answer.)

   ```
   TOM STEYER 2020
   ```

3. What week_start_date had the highest spend? (No need to insert query here, just type in the answer.)

   ```
   2020-09-13 : 21885400
   ```

4. Using the `advertiser_weekly_spend` table, write a query that returns the sum of spend by week (using week_start_date) in usd for the month of August only.

   ```
   WITH 
     T as (
     SELECT
       EXTRACT(MONTH FROM week_start_date) as month,
       EXTRACT(YEAR FROM week_start_date) as year,
       SUM(spend_usd) as usd,
     FROM
       `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
     GROUP BY 
       week_start_date  
     )
  
   SELECT 
     T.year,
     SUM(T.usd) as august_total
   FROM 
     T 
   WHERE 
     T.month = 8
   GROUP BY 
     T.year
   ```

5. How many ads did the 'TOM STEYER 2020' campaign run? (No need to insert query here, just type in the answer.)

   ```
   50
   ```

6. Write a query that has, in the US region only, the total spend in usd for each advertiser_name and how many ads they ran. (Hint, you're going to have to join tables for this one).

   ```
   WITH
     adv_money AS (
     SELECT
       advertiser_name ,
       SUM(spend_usd) AS usd_spent
     FROM
       `bigquery-public-data.google_political_ads.advertiser_weekly_spend` 
     GROUP BY 
       advertiser_name
     ),
     adv_ad_count AS (
     SELECT 
       advertiser_name,
       count(*) as num_ads
     FROM
       `bigquery-public-data.google_political_ads.advertiser_weekly_spend` 
     GROUP BY 
       advertiser_name 
     )

   SELECT
     A.advertiser_name,
     A.usd_spent,
     B.num_ads 
   FROM
     adv_money as A
     JOIN
     adv_ad_count as B
     ON 
     A.advertiser_name = B.advertiser_name
   WHERE 
     A.usd_spent > 0

   ```

7. For each advertiser_name, find the average spend per ad.

   ```
   SELECT
     advertiser_name,
     ROUND(AVG(spend_usd), 2) as avg_usd_spent
   FROM
     `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
   GROUP BY
     advertiser_name
   ```

8. Which advertiser_name had the lowest average spend per ad that was at least above 0.

   ```
	Ans: GREG CHANEY

   WITH 
     T AS (
     SELECT
       advertiser_name,
       ROUND(AVG(spend_usd), 2) as avg_usd_spent
     FROM
       `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
     GROUP BY
       advertiser_name
     )
  
   SELECT 
     * 
   FROM 
     T
   WHERE 
     T.avg_usd_spent > 0
   ORDER BY 
     T.avg_usd_spent 
   ```

## For this next section, use the `new_york_citibike` datasets.

1. Who went on more bike trips, Males or Females?

   ```
	Ans: Males

   SELECT
     gender,
     count(*) AS count
   FROM
     `bigquery-public-data.new_york_citibike.citibike_trips`
   group by 
     gender 
   ```

2. What was the average, shortest, and longest bike trip taken in minutes?

   ```
   WITH
     T AS (
   SELECT
     ROUND(tripduration / 60, 2) as trip_duration
   FROM
     `bigquery-public-data.new_york_citibike.citibike_trips` )
    
   SELECT
     MIN(trip_duration) as shortest_trip,
     AVG(trip_duration) as average_trip,
     MAX(trip_duration) as longest_trip,
   FROM
     T
   ```

3. Write a query that, for every station_name, has the amount of trips that started there and the amount of trips that ended there. (Hint, use two temporary tables, one that counts the amount of starts, the other that counts the number of ends, and then join the two.)
   ```
   WITH
     T_start AS (
     SELECT
       start_station_name,
       count(start_station_name) AS started_here
     FROM
       `bigquery-public-data.new_york_citibike.citibike_trips`
     GROUP BY
       start_station_name  
     ),
     T_end AS (
     SELECT
       end_station_name ,
       count(end_station_name) AS ended_here
     FROM
       `bigquery-public-data.new_york_citibike.citibike_trips`
     GROUP BY
       end_station_name   
     )
    
   SELECT
     A.start_station_name,
     A.started_here,
     B.ended_here 
   FROM
     T_start AS A
     JOIN
     T_end AS B
     ON  
     A.start_station_name  = B.end_station_name 
   ```

# The next section is the Google Colab section.

1. Open up this [this Colab notebook](https://colab.research.google.com/drive/1kHdTtuHTPEaMH32GotVum41YVdeyzQ74?usp=sharing).
2. Save a copy of it in your drive.
3. Rename your saved version with your initials.
4. Click the 'Share' button on the top right.
5. Change the permissions so anyone with link can view.
6. Copy the link and paste it right below this line.
   - YOUR LINK: https://colab.research.google.com/drive/1UE61qSYh4jXXHMBK2Up2avGgtNwqFJxE?usp=sharing
7. Complete the two questions in the colab notebook file.
