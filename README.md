# SQL-E-commerce-Data-Insights
## Introduction
This project provides a set of SQL queries designed to analyze e-commerce data using Google BigQuery. The goal of this project is to help data analysts gain insights into customer behavior, sales performance, and key business metrics through SQL queries executed on the Google Analytics Sample dataset.
### Objectives
- Extract and analyze user behavior data on an e-commerce website.
- Utilize Google BigQuery to write optimized and efficient SQL queries.
- Generate detailed reports based on real-world data to support business decision-making.
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




