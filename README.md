# SQL-E-commerce-Data-Insights
## Introduction
This project provides a set of SQL queries designed to analyze e-commerce data using Google BigQuery. The goal of this project is to help data analysts gain insights into customer behavior, sales performance, and key business metrics through SQL queries executed on the Google Analytics Sample dataset.
### Objectives
- Optimize marketing channels for better decision-making: Analyze bounce rates, traffic revenue, and session spending to allocate budgets effectively and boost conversions.
- Enhance purchase journey & maximize order value: Improve conversion rates and use data-driven product recommendations to drive higher sales.
### Dataset Used
- This project uses the Google Analytics Sample Dataset from BigQuery, which contains data from the Google Merchandise Store. namely "bigquery-public-data.google_analytics_sample.ga_sessions_2017*"
- Data tables:
    | Field Name           | Data Type                                 | Description                                    |
    |-----------------------------|---------------------------------------------------|-------------------------------------------------|
    | fullVisitorId  | STRING                 | The unique visitor ID.                          |
    | date | STRING                 |The date of the session in YYYYMMDD format.                  |
    | totals      | RECORD                        | This section contains aggregate values across the session.                      |
    | totals.bounces  |     INTEGER                                              | Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null.                            |
    |totals.hits |        INTEGER                                           | Total number of hits within the session.                           |
    | totals.pageviews   |        INTEGER                                    | Total number of pageviews within the session.                  |
    |totals.visits      |      INTEGER                                     | The number of sessions (for convenience). This value is 1 for sessions with interaction events. The value is null if there are no interaction events in the session.  |
     |totals.transactions      |       INTEGER                                         |  Total number of ecommerce transactions within the session. |
    |trafficSource.source      |        STRING                              | The source of the traffic source. Could be the name of the search engine, the referring hostname, or a value of the utm_source URL parameter.|
    | hits  | RECORD                  | This row and nested fields are populated for any and all types of hits.                          |
    | hits.eCommerceAction | RECORD               |This section contains all of the ecommerce hits that occurred during the session. This is a repeated field and has an entry for each hit that was collected. |
    | hits.eCommerceAction.action_type      | STRING                         |The action type. Click through of product lists = 1, Product detail views = 2, Add product(s) to cart = 3, Remove product(s) from cart = 4, Check out = 5, Completed purchase = 6, Refund of purchase = 7, Checkout options = 8, Unknown = 0. |
    | hits.product |                  RECORD                 |This row and nested fields will be populated for each hit that contains Enhanced Ecommerce PRODUCT data.|
    |hits.product.productQuantity |       INTEGER                           |The quantity of the product purchased.|
    | hits.product.productRevenue   |          INTEGER                                |The revenue of the product, expressed as the value passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000).|
    |hits.product.productSKU      |          STRING                                  |  Product SKU.     |
     |hits.product.v2ProductName      |      STRING                                      |Product Name.  |
## Data Exploration
#### 1. Total Visits, Pageviews, and Transactions (Jan, Feb, Mar 2017)
- Calculates the total number of visits, pageviews, and transactions for the first three months of 2017.
- Expected Output: Aggregated data for each month, ordered chronologically.
```sql
SELECT distinct format_date('%Y%m', PARSE_DATE('%Y%m%d', date)) as month  
  ,count (totals.visits)  as visits
  ,sum (totals.pageviews) as pageviews
  ,sum (totals.transactions) as transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
group by month
order by month;
```
<details>
  <summary>Results</summary>
    
|month |	visits|	pageviews| transactions |
|----------------|	--------------|	----------------| ----------------|
|201701|	64694	|257708| 713 |
|201702|	62192	|233373| 733 |
|201703|	69931	|259522| 993 |
|201704|	67126	|242576| 959 |
|201705|	65371	|255077| 1160 |
|201706|	63578	|233210| 971 |
|201707|	71812	|270554| 1072 |
|201708|	2556	|10939| 45 |
</details>

#### 2. Bounce Rate by Traffic Source (July 2017)
- Determines the bounce rate per traffic source to understand the effectiveness of each marketing channel.
- Formula: Bounce Rate = total_bounces / total_visits
```sql
select distinct trafficSource.source as trafficSource
  ,count(totals.visits) as total_visits
  ,sum(totals.bounces) as total_no_of_bounces
  ,round(100*(sum(totals.bounces) / count(totals.visits)),3) as bounce_rate
from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
where parse_date('%Y%m%d',date) between '2017-07-01' AND '2017-07-31'
group by trafficSource 
order by total_visits desc;
```
<details>
  <summary>Results</summary>
    
|trafficSource |	total_visits|	total_no_of_bounces| bounce_rate |
|----------------|	--------------|	----------------| ----------------|
| google |	38400 |	19798 |	51.557 |
| (direct) |	19891 |	8606 |	43.266 |
| youtube.com |	6351 |	4238 |	66.73 |
| analytics.google.com |	1972 |	1064 |	53.955 |
| Partners |	1788  |	936 |	52.349 |
</details>

#### 3. Revenue by Traffic Source (Weekly and Monthly, June 2017)
- Analyzes revenue generated from different traffic sources on a weekly and monthly basis.
- Uses UNNEST(hits) and UNNEST(hits.product) to access productRevenue
```sql
select 'Month'as time_type
  ,FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) as time
  ,trafficSource.source as source
  ,round((SUM(product.productRevenue) / 1000000),4) as revenue
from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) hits,
UNNEST (hits.product) product
WHERE FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) = '201706'
  and product.productRevenue is not null
GROUP BY time, source
union all
select 'Week'as time_type
  ,FORMAT_DATE('%Y%V', PARSE_DATE('%Y%m%d', date)) as time
  ,trafficSource.source as source
  ,round((SUM(product.productRevenue) / 1000000),4) as revenue
from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) hits,
UNNEST (hits.product) product
WHERE FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) = '201706'
  and product.productRevenue is not null
GROUP BY time, source
ORDER BY revenue desc;
```
<details>
  <summary>Results</summary>
    
|time_type |	time|	source| revenue |
|----------------|	--------------|	----------------| ----------------|
|Month|	201706|	(direct)|	97333.6197|
|Week|	201724|	(direct)|	30908.9099|
|Week|	201725|	(direct)|	27295.3199|
|Month|	201706|	google|	18757.1799|
|Week|	201723|	(direct)|	17325.6799|
</details>

#### 4. Average Pageviews by Purchaser Type (June, July 2017)
- Compares the average number of pageviews between purchasers and non-purchasers.
```sql
with purchaser as (
  select FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) as month
    ,round((sum(totals.pageviews) / count(distinct fullVisitorId)),8) as avg_pageviews_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  where (totals.transactions >=1) 
    and (product.productRevenue is not null) 
    and (_table_suffix between '0601' and '0731')
  group by month
  order by month
)
, non_purchaser as (
  select FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) as month
    ,round((sum(totals.pageviews) / count(distinct fullVisitorId)),8) as avg_pageviews_non_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  where (totals.transactions is null) 
    and (product.productRevenue is  null) 
    and (_table_suffix between '0601' and '0731')
  group by month
  order by month
)
select p.month
  ,p.avg_pageviews_purchase
  ,np.avg_pageviews_non_purchase
from purchaser p
join non_purchaser np
on p.month = np.month
order by month;
```
<details>
  <summary>Results</summary>
    
|month |	avg_pageviews_purchase|	avg_pageviews_non_purchase|
|----------------|	--------------|	----------------|
|201706|	94.02050114|	316.86558846|
|201707|	124.23755187|	334.0565598|
</details>

### 5. Average Transactions per User (July 2017)
- Computes the average number of transactions per user who made a purchase in July 2017.
```sql
select FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) as month
  ,round((sum(totals.transactions) / count(distinct fullVisitorId)),9) as Avg_total_transactions_per_user
from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) hits,
UNNEST (hits.product) product
where (totals.transactions >=1) 
  and (product.productRevenue is not null) 
  and (_table_suffix between '0701' and '0731')
group by month;
```
<details>
  <summary>Results</summary>
    
|month |	Avg_total_transactions_per_user|
|----------------|	--------------|
|201707|	4.163900415|
</details>

### 6. Average Revenue per Session (July 2017)
- Calculates the average revenue generated per session, considering only purchasing users.
```sql
select FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) as month
  ,round((SUM((product.productRevenue) / 1000000) / count(totals.visits)),2) as avg_revenue_by_user_per_visit
 from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) hits,
UNNEST (hits.product) product 
where (totals.transactions is not null) 
  and (product.productRevenue is not null)
  and  (_table_suffix between '0701' and '0731')
group by month;
```
<details>
  <summary>Results</summary>
    
|month |	avg_revenue_by_user_per_visit|
|----------------|	--------------|
|201707|	43.86|
</details>

### 7. Other Products Purchased by Customers Who Bought "YouTube Men's Vintage Henley" (July 2017)
- Identifies other products bought by customers who purchased the "YouTube Men's Vintage Henley" product.
```sql
with users_buy_the_product as (
  select  fullVisitorId
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product 
  where product.v2ProductName = "YouTube Men's Vintage Henley"
    and (_table_suffix between '0701' and '0731')
    and product.productRevenue is not null
  )

select product.v2ProductName as other_purchased_products
  ,sum(product.productQuantity) as quantity
from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) hits,
UNNEST (hits.product) product 
where fullVisitorId in (select fullVisitorId from users_buy_the_product)
  and product.v2ProductName != "YouTube Men's Vintage Henley" 
  and (_table_suffix between '0701' and '0731') 
  and product.productRevenue is not null
group by other_purchased_products
order by quantity desc;
```
<details>
  <summary>Results</summary>
    
|other_purchased_products |	quantity|
|----------------|	--------------|
|Google Sunglasses|	20|
|Google Women's Vintage Hero Tee Black |	7|
|SPF-15 Slim & Slender Lip Balm |	6|
|Google Women's Short Sleeve Hero Tee Red Heather	|4|
|Google Men's Short Sleeve Badge Tee Charcoal |	3|
</details>

### 8. Cohort Analysis: Product View to Add to Cart to Purchase (Jan, Feb, Mar 2017)
- Computes the conversion rates from product views to cart additions and purchases at the product level.
```sql
with month_view_data as (
  select FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) as month
        ,count(hits.eCommerceAction.action_type) as num_product_view
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) hits
  where hits.eCommerceAction.action_type = "2"
  group by month
  order by month
)
  , addtocart_data as (
  select FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) as month
        ,count(hits.eCommerceAction.action_type) as num_addtocart
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) hits
  where hits.eCommerceAction.action_type = "3" 
  group by month
  order by month
)
, purchase_data as (
  select FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) as month
      ,count(hits.eCommerceAction.action_type) as num_purchase
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product 
  where hits.eCommerceAction.action_type = "6" 
    and product.productRevenue is not null
  group by month
  order by month
)

select m. month
  ,m.num_product_view
  ,a.num_addtocart
  ,p.num_purchase
  ,round(100*(a.num_addtocart / m.num_product_view ),2) as add_to_cart_rate
  ,round(100*(p.num_purchase / m.num_product_view),2) as purchase_rate
from  month_view_data m
left join addtocart_data a 
on m.month = a.month
left join purchase_data p   
on m.month = p.month 
order by month;
```
<details>
  <summary>Results</summary>
    
|month |	num_product_view|	num_addtocart| num_purchase | add_to_cart_rate | purchase_rate |
|----------------|	--------------|	----------------| ----------------| ----------------| ----------------|
|201701|	25787|	7342|	2143|	28.47|	8.31|
|201702|    21489|	7360|	2060|	34.25|	9.59|
|201703|	23549|	8782|	2977|	37.29|	12.64|
|201704|	24587|	10291|	2906|	41.86|	11.82|
|201705|	25469|	10083|	3285|	39.59|	12.9|
|201706|	22148|	9020|    2785|	40.73|	12.57|
|201707|	28576|	11860|	3669|	41.5|    12.84|
|201708|	1267|	494|	186|	38.99|	14.68|
</details>

