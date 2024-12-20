## Project Title: Website Ecommerce Performance  
## Introduction: 

Welcome to my first project on exploring e-commerce data! This repository features the SQL script titled "Explore E-Commerce Dataset", which delves into analyzing e-commerce data using Google Analytics samples. Our goal is to derive valuable insights into customer behaviors, website performance, and transactional trends.

## Objective: 
- Analyze and explore user behavior patterns in an eCommerce dataset through specific queries. 
- Gain insights into website performance, customer shopping behavior, and revenue generation.
- Assess metrics such as total visits, bounce rates, and average transactions per user.
- Identify purchasing trends and conduct a cohort analysis to understand the customer journey.
- Use the findings to inform strategic recommendations that enhance marketing and improve customer satisfaction.

## Analysis Focus:

- Traffic Analysis:

  - Calculate total visits, page views, and transactions for January, February, and March 2017 to assess trends and changes in user behavior.
- Bounce Rate:

  - Determine the bounce rate by traffic source for July 2017 to evaluate the effectiveness of marketing channels.
- Revenue by Traffic Source:

  - Analyze weekly and monthly revenue for June 2017 to identify the highest value channels.
- Average Page Views:

  - Calculate average page views by buyer type (buyers vs. non-buyers) for June and July 2017.
- Average Transactions per User:

  - Evaluate the average number of transactions per user who made purchases in July 2017.
- Average Spend per Session:

  - Calculate the average amount spent by buyers per session, focusing on buyer data for July 2017.
- Other Products Purchased:

  - Identify other products customers bought when they purchased the "YouTube Men's Vintage Henley" in July 2017.
- Cohort Mapping:

  - Calculate cohort mapping from product views to add-to-cart and purchases for January, February, and March 2017.

## Tools and Techniques Used: 

- SQL
- BigQuery

## Execution Process: 
- Data Collection: Gather the eCommerce dataset from relevant sources and load it into BigQuery.
- Data Cleaning: Use SQL queries to clean the dataset by removing duplicates, handling missing values, and ensuring data consistency.
- Metric Calculation: Calculate key metrics such as total visits, bounce rates, and average transactions per user using SQL aggregation functions.
- Trend Analysis: Execute specific queries to identify purchasing trends over different time periods.
- Cohort Analysis: Perform cohort analysis to track the customer journey, focusing on user behavior from product views to purchases.

## Results Achieved: 
- Learn how to manipulate data on the BigQuery platform.
- Be able to use SQL to clean and query data according to the requirements of the task.
- Calculate the necessary metrics to serve the analysis.
- Identify insights from the data to support the next steps in the workflow.

## Project processing workflow
### Step 1: Document review, comprehension, and problem identification.
- [Table Schema](https://support.google.com/analytics/answer/3437719?hl=en)

### Step 2: Identify the questions and issues that need to be addressed and resolved.
#### Query 1: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
```SQL
SELECT
  format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
  SUM(totals.visits) AS visits,
  SUM(totals.pageviews) AS pageviews,
  SUM(totals.transactions) AS transactions,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _TABLE_SUFFIX BETWEEN '0101' AND '0331'
GROUP BY 1
ORDER BY 1;
```
- Result_Q1:
  
![image](https://github.com/user-attachments/assets/60106f62-8980-4641-8d63-ed698acff49d)

##### Insight:
  - Growth Trend:
      - The number of visits, pageviews, and transactions are increasing each month, indicating the website is experiencing growth. March had the highest activity, which could be the peak month for the website.
  - Potential:
      - With the clear growth trend, the website has significant potential for further growth.


#### Query 2: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
```SQL
SELECT
    trafficSource.source as source,
    sum(totals.visits) as total_visits,
    sum(totals.Bounces) as total_no_of_bounces,
    (sum(totals.Bounces)/sum(totals.visits))* 100 as bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY source
ORDER BY total_visits DESC;
```
- Result_Q2:

![image](https://github.com/user-attachments/assets/2903d011-6590-45cf-8bf2-e8946d129b5b)

##### Insight:
  - Main Traffic Source: The largest traffic source is from google, with 38,400 visits. This indicates that google is the most effective marketing channel.

  - High Bounce Rates: The bounce rates of the traffic sources are quite high, especially for youtube.com at 66.73%. This suggests the need to improve content quality and user experience to better retain visitors.

  - Growth Potential: The direct source has 19,891 visits, with a lower bounce rate compared to other sources (43.266%). This could be a promising source to focus on and develop, such as by strengthening direct marketing efforts.

#### Query 3: Revenue by traffic source by week, by month in June 2017
```SQL
with 
month_data as(
  SELECT
    "Month" as time_type,
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  WHERE p.productRevenue is not null
  GROUP BY 1,2,3
  order by revenue DESC
),

week_data as(
  SELECT
    "Week" as time_type,
    format_date("%Y%W", parse_date("%Y%m%d", date)) as week,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  WHERE p.productRevenue is not null
  GROUP BY 1,2,3
  order by revenue DESC
)

SELECT * 
FROM month_data
union all
SELECT * 
FROM week_data;
```
- Result_Q3:

![image](https://github.com/user-attachments/assets/ef24ab31-532d-41e9-9075-822dd2ebb720)

##### Insight:
  - The data is aggregated both monthly and weekly, allowing analysis of revenue trends at different granularities.
  - The primary data source is direct ((direct)), but there is also one source from google.
  - The revenue figures show significant variation across time periods and data sources. Further analysis is needed to understand the driving factors.
  - With this detailed data, in-depth analysis can be performed on revenue trends, comparisons between data sources, and identification of the causes behind the fluctuations.

#### Query 4: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
```SQL
WITH avg_pageview6_nonpurchaser as (select 
      COUNT(DISTINCT(fullVisitorId)) as uniq_userid
      , sum(totals.pageviews) AS total_pageview_6   
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) as hits,
UNNEST (product) as product
WHERE 
      (_table_suffix BETWEEN '0601' AND '0630')
      AND totals.transactions IS NULL  
      AND product.productRevenue is null ),

Last_result6_Avg_pageview_non_purchaser as (SELECT 
      (total_pageview_6 / uniq_userid) as Avg_pageview_purchaser
FROM avg_pageview6_nonpurchaser),
avg_pageview7_nonpurchaser as (select 
      COUNT(DISTINCT(fullVisitorId)) as uniq_userid
      , sum(totals.pageviews) AS total_pageview_7  
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) as hits,
UNNEST (product) as product
WHERE 
      (_table_suffix BETWEEN '07011' AND '0731')
      AND totals.transactions IS NULL  
      AND product.productRevenue is null ),

last_result7_Avg_pageview_non_purchaser as (SELECT 
      (total_pageview_7 / uniq_userid) as Avg_pageview_purchaser
FROM avg_pageview7_nonpurchaser)

SELECT *
FROM Last_result6_Avg_pageview_non_purchaser
UNION ALL
SELECT *
FROM last_result7_Avg_pageview_non_purchaser
ORDER BY Avg_pageview_purchaser
```
- Result_Q4:

![image](https://github.com/user-attachments/assets/2c739087-20bf-458c-bb1a-1e2b01c08927)

##### Insight:

  - The average number of page views for customers who have made a purchase increased from 94.02 to 124.24, an increase of around 32%.
  - The average number of page views for customers who have not made a purchase increased from 316.87 to 334.06, an increase of around 5.4%. 
  - This shows that the increase in the average number of page views for customers who have made a purchase is much higher than for customers who have not made a purchase.
  - The data shows that customers who have not made a purchase view an average of more pages than customers who have made a purchase, which could be because they are searching for information and considering a purchase.
  - Understanding the behavior of the group of customers who have not made a purchase better can help improve the user experience and increase the conversion rate.
  - Additionally, tracking and comparing the trends in page views between the two customer groups can also provide useful information to adjust marketing and sales strategies.

#### Query 5: Average number of transactions per user that made a purchase in July 2017
```SQL
WITH avg_tra as (
SELECT 
        COUNT(DISTINCT(fullVisitorId))as user_id
        , COUNT(DISTINCT(totals.transactions)) as total_transaction
            
FROM 
        `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
        UNNEST (hits) as hits,
        UNNEST (product) as product 
WHERE
        _table_suffix BETWEEN '0701' AND '0731'
        AND product.productRevenue is not null 
),
date_edited as 
(
    SELECT 
    PARSE_DATE('%Y%m%d', date ) AS time_type
    `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
    WHERE 
    _table_suffix BETWEEN '0701' AND '0731' 
)


SELECT 
    DISTINCT(FORMAT_DATE("%Y%m",date_edited.time_type)) as Month
    , (avg_tra.total_transaction / avg_tra.user_id ) as Avg_total_transactions_per_user
FROM avg_tra, date_edited
```
- Result_Q5:

![image](https://github.com/user-attachments/assets/21caf6e9-120c-422d-884e-3ff11b66f6de)

##### Insight:
  - Average total transactions per user: In July 2017, each unique user performed an average of more than 4 transactions.

#### Query 6: Average amount of money spent per session. Only include purchaser data in July 2017
```SQL
SELECT
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    ((sum(product.productRevenue)/sum(totals.visits))/power(10,6)) as avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  ,unnest(hits) hits
  ,unnest(product) product
WHERE product.productRevenue is not null
and totals.transactions>=1
GROUP BY month;
```
- Result_Q6:

![image](https://github.com/user-attachments/assets/26fa4f41-f8ad-410a-97c2-a5f71545ba5a)

##### Insight:
  - The SQL query calculates the average revenue per user visit (avg_revenue_by_user_per_visit) by dividing the total product revenue (sum(product.productRevenue)) by the total number of visits (sum(totals.transactions)).
  - The result shows that in July 2017, the average revenue per user visit was $43.85.

#### Query 7: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.
```SQL
with buyer_list as(
    SELECT
        distinct fullVisitorId
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , UNNEST(hits) AS hits
    , UNNEST(hits.product) as product
    WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions>=1
    AND product.productRevenue is not null
)

SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
, UNNEST(hits) AS hits
, UNNEST(hits.product) as product
JOIN buyer_list using(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
 and product.productRevenue is not null
GROUP BY other_purchased_products
ORDER BY quantity DESC;
```
- Result_7:

 ![image](https://github.com/user-attachments/assets/8e667935-0623-44ee-924a-61ceaa924fed)

 ##### Insight:
 
  - The SQL query first identifies the set of customers who purchased the "YouTube Men's Vintage Henley" product and had at least one transaction.
  - It then aggregates the quantities of other products purchased by these customers, excluding the "YouTube Men's Vintage Henley" product.
  - The results show the top 4 most frequently purchased products by these customers, providing insights into their buying behavior and potential cross-selling opportunities.
 
#### Query 8: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase. Add_to_cart_rate = number product  add to cart/number product view. Purchase_rate = number product purchase/number product view. The output should be calculated in product level.
```SQL
with
product_view as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_product_view
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '2'
  GROUP BY 1
),

add_to_cart as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_addtocart
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '3'
  GROUP BY 1
),

purchase as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '6'
  and product.productRevenue is not null 
  GROUP BY 1
)

SELECT
    pv.*,
    num_addtocart,
    num_purchase,
    round(num_addtocart*100/num_product_view,2) as add_to_cart_rate,
    round(num_purchase*100/num_product_view,2) as purchase_rate
FROM product_view pv
LEFT JOIN add_to_cart a on pv.month = a.month
LEFT JOIN purchase p on pv.month = p.month
ORDER BY pv.month;
```
- Result_Q8

![image](https://github.com/user-attachments/assets/23aae79d-0d66-4e37-bc2a-de6015ecb66d)

##### Insight 1: Add-to-cart rate and purchase rate

  - The add-to-cart rate ranged from 28.47% to 37.29% during the first 3 months of 2017.
  - The purchase rate ranged from 8.31% to 12.64% during the same period.
##### Insight 2: Product views, add-to-cart, and purchases

  - The number of product views decreased from 25,787 in January to 23,549 in March.
  - The number of products added to cart increased from 7,342 in January to 8,782 in March.
  - The number of products purchased increased from 2,143 in January to 2,977 in March.
#### These insights provide information about customer behavior during the online shopping process, including add-to-cart rate, purchase rate, and the volume of product views, add-to-cart, and purchases. This information can help the business improve its marketing strategies and optimize the sales funnel.



