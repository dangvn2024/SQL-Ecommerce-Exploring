# SQL-Ecommerce-Exploring
Using BigQuery to explore and analyze datasets from an e-commerce company.

## Table of Contents:
1. [Introduction and Motivation](#data)
2. [The goal of creating this project](#clean_data)
3. [Import raw data](#cau3)
4. [Read and explain dataset](#cau4)
5. [ Data Processing & Exploratory Data Analysis](#cau5)
6. [Ask questions and solve it](#cau6)
7. [Conclusion](#cau7)

<div id='data'/>
  
## 1. Introduction and Motivation

The eCommerce dataset is stored in a public Google BigQuery dataset. This dataset contains information about user sessions on a website collected from Google Analytics in 2017.

Based on the eCommerce dataset, the author perform queries to analyze website activity in 2017, such as calculating bounce rate, identifying days with the highest revenue, analyzing user behavior on pages, and various other types of analysis. This project aims to have an outlook on the business situation, marketing activity efficiency analyzing the products.

To query and work with this dataset, the author uses the Google BigQuery tool to write and execute SQL queries.

<div id='clean_data'/>

## 2. The goal of creating this project
- Overview of website activity
- Bounce rate analysis
- Revenue analysis
- Transactions analysis
- Products analysis

<div id='cau3'/>

## 3. Import raw data
  
The eCommerce dataset is stored in a public Google BigQuery dataset. To access the dataset, follow these steps:
- Log in to your Google Cloud Platform account and create a new project.
- Navigate to the BigQuery console and select your newly created project.
- Select "Add Data" in the navigation panel and then "Search a project".
- Enter the project ID **"bigquery-public-data.google_analytics_sample.ga_sessions"** and click "Enter".
- Click on the **"ga_sessions_"** table to open it.

<div id='cau4'/>
  
## 4. Read and explain dataset
https://support.google.com/analytics/answer/3437719?hl=en
 
|  Field Name | Data Type | Description |
| --- | --- | --- |
| fullVisitorId                    | STRING    | The unique visitor ID.|
| date                             | STRING    | The date of the session in YYYYMMDD format. |     
| totals                           | RECORD    | This section contains aggregate values across the session.    |
| totals.bounces                   | INTEGER   | Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null.     |
| totals.hits                      | INTEGER   | Total number of hits within the session.   |
| totals.pageviews                 | INTEGER   | Total number of pageviews within the session.    |
| totals.visits                    | INTEGER   | The number of sessions (for convenience). This value is 1 for sessions with interaction events. The value is null if there are no interaction events in the session.   |    
| trafficSource.source             | STRING    | The source of the traffic source. Could be the name of the search engine, the referring hostname, or a value of the utm_source URL parameter.|
| hits                             | RECORD    | This row and nested fields are populated for any and all types of hits|
| hits.eCommerceAction             | RECORD    | This section contains all of the ecommerce hits that occurred during the session. This is a repeated field and has an entry for each hit that was collected.   |
| hits.product                     | RECORD    | This row and nested fields will be populated for each hit that contains Enhanced Ecommerce PRODUCT data. |  
| hits.product.productQuantity     | INTEGER   | The quantity of the product purchased.   |
| hits.product.productRevenue      | INTEGER   | The revenue of the product, expressed as the value passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000). |   
| hits.product.productSKU          | STRING    | Product SKU.    |
| hits.product.v2ProductName       | STRING    | Product Name.   |
| fullVisitorId                    | STRING    | The unique visitor ID.   |

<div id='cau5'/>

## 5. Data Processing & Exploratory Data Analysis
~~~~sql
SELECT COUNT(fullVisitorId) row_num,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
~~~~

| row_num |
|---------|
| 71812   |

~~~~sql
SELECT COUNT(fullVisitorId) row_num,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
~~~~

| row_num |
|---------|
| 467260  |

~~~~sql
SELECT EXTRACT(MONTH FROM PARSE_DATE("%Y%m%d",date)) month
,COUNT(*) AS counts
,ROUND((COUNT(*)/(SELECT COUNT(*) 
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`))*100,1) pct
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
GROUP BY EXTRACT(MONTH FROM PARSE_DATE("%Y%m%d",date))
~~~~

| month | counts | pct  |
|-------|--------|------|
| 6     | 63578  | 13.6 |
| 3     | 69931  | 15.0 |
| 8     | 2556   | 0.5  |
| 2     | 62192  | 13.3 |
| 4     | 67126  | 14.4 |
| 1     | 64694  | 13.8 |
| 7     | 71812  | 15.4 |
| 5     | 65371  | 14.0 |



**UNNEST hits and products**
~~~~sql
SELECT date, 
fullVisitorId,
eCommerceAction.action_type,
product.v2ProductName,
product.productRevenue,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits) AS hits,
UNNEST(hits.product) as product
~~~~

| date     | fullVisitorId       | action_type | v2ProductName                         | productRevenue |
|----------|---------------------|-------------|---------------------------------------|----------------|
| 20170712 | 4080810487624198636 | 1           | YouTube Custom Decals                 |                |
| 20170712 | 4080810487624198636 | 2           | YouTube Custom Decals                 |                |
| 20170712 | 7291695423333449793 | 1           | Keyboard DOT Sticker                  |                |
| 20170712 | 7291695423333449793 | 2           | Keyboard DOT Sticker                  |                |
| 20170712 | 3153380067864919818 | 2           | Google Baby Essentials Set            |                |
| 20170712 | 3153380067864919818 | 1           | Google Baby Essentials Set            |                |
| 20170712 | 5615263059272956391 | 0           | Android Lunch Kit                     |                |
| 20170712 | 5615263059272956391 | 0           | Android Rise 14 oz Mug                |                |
| 20170712 | 5615263059272956391 | 0           | Android Sticker Sheet Ultra Removable |                |
| 20170712 | 5615263059272956391 | 0           | Windup Android                        |                |

<div id='cau6'/>

## 6. Ask questions and solve it
**6.1: calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)**
~~~~sql
WITH raw_data_q1 AS(

      SELECT 
            FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))AS month
            ,totals.visits
            ,totals.pageviews
            ,totals.transactions

      FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
      WHERE _table_suffix BETWEEN '0101' AND '0330'
      )

SELECT 
      month
      ,SUM (visits) AS total_visits
      ,SUM (pageviews) AS total_pageviews
      ,SUM (transactions) AS total_transactions
FROM raw_data_q1
GROUP BY month
ORDER BY month;
~~~~
|Month	|visits	|pageviews	|transactions|
|--------|--------|-----------|--------------|
|201701	|64694	|257708	|713|
|201702	|62192	|233373	|733|
|201703	|69931	|259522	|993|

There is a positive trend from February to March in all three metrics, with notable growth in transactions.
February appears to be the weakest month in terms of site engagement and transactions, which could be worth investigating for potential causes (e.g., seasonal trends, marketing activities).
 
  **Opportunities for Improvement**: While the data shows positive growth, there might be opportunities to optimize user engagement and conversions further. Analyzing specific user journeys, popular landing pages, and exit points could help identify areas for improvement.
  
 **Future Strategy and Focus**: Consider further investigating what strategies, campaigns, or changes were implemented between January and March that contributed to the significant increase in transactions and revenue. These insights can inform future marketing and optimization efforts.



 **6.2 Bounce rate per traffic source in July 2017**
~~~~sql

SELECT
    trafficSource.source
    ,SUM (totals.visits) AS total_visits
    ,SUM (totals.bounces) AS total_no_of_bounces
    ,ROUND (SUM (totals.bounces)/ SUM (totals.visits) * 100  ,3) AS bounce_rate
 
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 WHERE _table_suffix BETWEEN '0701' AND '0731'  
 GROUP BY 1
 ORDER BY total_visits DESC;
~~~~
First Rows
| source                      | total_visits | total_no_of_bounces | bounce_rate |
|-----------------------------|--------------|---------------------|-------------|
| google                      | 38400        | 19798               | 51.56       |
| (direct)                    | 19891        | 8606                | 43.27       |
| youtube.com                 | 6351         | 4238                | 66.73       |
| analytics.google.com        | 1972         | 1064                | 53.96       |
| Partners                    | 1788         | 936                 | 52.35       |
| m.facebook.com              | 669          | 430                 | 64.28       |
| google.com                  | 368          | 183                 | 49.73       |
| dfa                         | 302          | 124                 | 41.06       |
| sites.google.com            | 230          | 97                  | 42.17       |
| facebook.com                | 191          | 102                 | 53.4        |
| reddit.com                  | 189          | 54                  | 28.57       |
| qiita.com                   | 146          | 72                  | 49.32       |
| baidu                       | 140          | 84                  | 60          |
| quora.com                   | 140          | 70                  | 50          |
| bing                        | 111          | 54                  | 48.65       |

The table provides an overview of website traffic from various sources and key metrics that help evaluate user engagement and behavior based on four elements: source, total_visits, total_no_of_bounces, bounce_rate. 

Google website received the most traffic to the website at 38,400 visits in the first row. Of these, 19,798 were single-page visits (bounces), resulting in a bounce rate of approximately 51.56%. Following that Direct website took second place at 19,891 visits from direct traffic, with 8,606 bounces, leading to a bounce rate of about 43.27%. Thereafter, the Youtube.com website had 6,351 visits, with 4,238 bounces, resulting in a bounce rate of approximately 66.73%, higher than Google and Direct. However, search.mysearch.com website had the lowest total visits but the highest bounce rate. It had 12 visits with 11 bounces, resulting in a bounce rate of approximately 91.67%.

This indicates that most users who visited the website from this source left without interacting with other pages. A high bounce rate from a source like search.mysearch.com suggests that users arriving from this source might not find the content they are looking for or that the landing page experience may not be engaging enough to encourage further exploration. It's important to note that while addressing high bounce rates is essential, the context of the source and user behavior should be thoroughly analyzed to determine the most effective strategies for improvement. Because a lower bounce rate generally indicates that users are more engaged with the website content, while a higher bounce rate may suggest that users are leaving the site after viewing only a single page.


**6.3 Revenue by traffic source by week, by month in June 2017**
~~~~sql
 WITH raw_data_q3_week AS (

        SELECT 
            'Week' AS time_type
        ,FORMAT_DATE('%Y%W',PARSE_DATE('%Y%m%d',date)) AS time
        ,trafficSource.source
        ,ROUND (SUM (productRevenue)/ 1000000 ,4) AS revenue
        FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
        UNNEST (hits) hits,
        UNNEST (hits.product) product
        WHERE product.productRevenue is not null
        GROUP BY 2,3
        ORDER BY 3,2,4 DESC
        ),

    raw_data_q3_month AS (
        SELECT 
        'Month' AS time_type
        ,FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))AS time
        ,trafficSource.source
        ,ROUND (SUM (productRevenue)/ 1000000 ,4) AS revenue
        FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
        UNNEST (hits) hits,
        UNNEST (hits.product) product
        WHERE product.productRevenue is not null
        GROUP BY 2,3
        ORDER BY 2,4 DESC
    )

SELECT 
    *
FROM raw_data_q3_month

UNION ALL

SELECT
    *
FROM raw_data_q3_week
ORDER BY source, time;
~~~~
Result first rows 
| time_type | time   | source                        | revenue  |
|-----------|--------|-------------------------------|----------|
| Month     | 201706 | (direct)                      | 97231.62 |
| WEEK      | 201724 | (direct)                      | 30883.91 |
| WEEK      | 201725 | (direct)                      | 27254.32 |
| Month     | 201706 | google                        | 18757.18 |
| WEEK      | 201723 | (direct)                      | 17302.68 |
| WEEK      | 201726 | (direct)                      | 14905.81 |
| WEEK      | 201724 | google                        | 9217.17  |
| Month     | 201706 | dfa                           | 8841.23  |
| WEEK      | 201722 | (direct)                      | 6884.9   |
| WEEK      | 201726 | google                        | 5330.57  |
| WEEK      | 201726 | dfa                           | 3704.74  |
| Month     | 201706 | mail.google.com               | 2563.13  |
| WEEK      | 201724 | mail.google.com               | 2486.86  |
| WEEK      | 201724 | dfa                           | 2341.56  |
| WEEK      | 201722 | google                        | 2119.39  |
| WEEK      | 201722 | dfa                           | 1670.65  |


This table represents revenue data collection with various attributes, including time type, time period, source, and revenue amount. Each row corresponds to a specific time period (week or month) and provides information about the revenue generated from different sources during that time period. The author found the insights bellow through the table above.

 **Direct Revenue and Source Breakdown:** The dataset includes revenue from various sources, such as (direct), Google, DFA, mail.google.com, and search engines like myway.com. Revenue comes from different sources, including direct website visits, organic search traffic, and referrals from various websites.
 
**Time Period Analysis:**
Revenue is reported on a weekly and monthly basis. It seems like the data spans multiple months, including the month labeled "201706."


**6.4 Average number of pageviews by purchaser type**
~~~~sql
WITH rd_q4_purchase AS (
      SELECT  
            FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))AS month
            ,ROUND (SUM (totals.pageviews) / COUNT ( DISTINCT fullVisitorId ) ,4) AS avg_pageviews_purchase
      FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
              UNNEST (hits) AS hits,
              UNNEST (hits.product) AS product
      WHERE 
            totals.transactions >= 1 
        AND  product.productRevenue is not null
        AND _table_suffix BETWEEN '0601' AND '0731'
      GROUP BY 1
      ),

    rd_q4_non_purchase AS (
        SELECT  
            FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))AS month
            ,ROUND (SUM (totals.pageviews) / COUNT ( DISTINCT fullVisitorId ) ,4) AS avg_pageviews_non_purchase
      FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
              UNNEST (hits) AS hits,
              UNNEST (hits.product) AS product
      WHERE 
            totals.transactions IS NULL
        AND  product.productRevenue IS NULL
        AND _table_suffix BETWEEN '0601' AND '0731'
      GROUP BY 1
      )
  
SELECT
    rd_q4_purchase.month
    ,rd_q4_purchase.avg_pageviews_purchase
    ,rd_q4_non_purchase.avg_pageviews_non_purchase
FROM rd_q4_purchase
FULL JOIN rd_q4_non_purchase 
USING (month)
ORDER BY month;
~~~~

| month  | avg_pageviews_purchase | avg_pageviews_non_purchase |
|--------|------------------------|----------------------------|
| 201706 | 25.73            | 4.07               |
| 201707 | 27.72            | 4.19               |

The table shows user behavior in a tabular format with two columns  "avg_pageviews_purchase," and "avg_pageviews_non_purchase"  in two consecutive months, June 2017 (201706) and July 2017 (201707).

 **Pageviews for Users Making a Purchase vs. Non-Purchasers:** The data highlights a significant difference in average pageviews between users who made a purchase and those who did not. On average, users who made a purchase tend to view more pages on the website than those who did not. In June 2017, for instance, the average pageviews for purchasers (25.74) were much higher compared to non-purchasers (4.07).
 
 **Engagement Patterns:** The higher average pageviews for purchasers suggest that users who are more engaged with the website's content are more likely to make a purchase. This could indicate a positive correlation between engagement and conversion.
 
 **Behavioral Insights:** The data reflect different user behaviors. Users interested in making a purchase might spend more time exploring product details, reading reviews, and comparing options, leading to a higher number of pageviews.
 
 **Conversion Optimization:** The business can leverage this data to optimize its website's design and content to encourage higher engagement among users likely to purchase. This could involve improving navigation, showcasing relevant products, and providing valuable content to guide users toward conversion.


**6.5 Average number of transactions per user that purchased in July 2017**
~~~~sql

SELECT  
    FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))AS month
    ,ROUND (SUM (totals.transactions) / COUNT ( DISTINCT fullVisitorId ) ,4) AS avg_total_transactions_per_user

FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) AS hits,
  UNNEST (hits.product) AS product
WHERE 
       totals.transactions >= 1 
  AND  product.productRevenue is not null
  AND _table_suffix BETWEEN '0701' AND '0731'
GROUP BY 1;
~~~~

| Month  | Avg_total_transactions_per_user |
|--------|---------------------------------|
| 201707 | 1.11                   |

The table shows the average total transactions per user in July. 
This data suggests that, during July 2017, the typical user conducted about 1.11 transactions on average. This could be useful for understanding user behavior, tracking user engagement with your platform, or evaluating the effectiveness of marketing campaigns or promotions during that specific month.

**6.6 Average amount of money spent per session. Only include purchaser data in 2017**
~~~~sql

SELECT  
    FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))AS month
    ,ROUND (SUM (productRevenue) / SUM (totals.visits) /1000000 ,2)  AS avg_total_transactions_per_user

FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) AS hits,
  UNNEST (hits.product) AS product
WHERE 
       totals.transactions IS NOT NULL
  AND  product.productRevenue IS NOT NULL
  AND _table_suffix BETWEEN '0701' AND '0731'
GROUP BY 1;

~~~~

The average total transactions per user for July 2017 is 43.86. This suggests that, on average, each user conducted approximately 44 transactions during that month. This could be an important metric for businesses to measure user engagement and activity.

**6.7 Other products purchased by customers who purchased product” Youtube Men’s Vintage Henley” in July 2017**
~~~~sql

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
~~~~

First Rows 
| other_purchased_products                                 | quantity |
|----------------------------------------------------------|----------|
| Google Sunglasses                                        | 20       |
| Google Womens Vintage Hero Tee Black                     | 7        |
| SPF-15 Slim & Slender Lip Balm                           | 6        |
| Google Womens Short Sleeve Hero Tee Red Heather          | 4        |
| YouTube Mens Fleece Hoodie Black                         | 3        |
| Google Mens Short Sleeve Badge Tee Charcoal              | 3        |
| YouTube Twill Cap                                        | 2        |
| Red Shine 15 oz Mug                                      | 2        |
| Google Doodle Decal                                      | 2        |
| Recycled Mouse Pad                                       | 2        |
| Google Mens Short Sleeve Hero Tee Charcoal               | 2        |
| Android Womens Fleece Hoodie                             | 2        |
| 22 oz YouTube Bottle Infuser                             | 2        |
| Android Mens Vintage Henley                              | 2        |
| Crunch Noise Dog Toy	                                   | 2        |
| Android Wool Heather Cap Heather/Black                   | 2        |
| Google Mens Vintage Badge Tee Black                      | 1        |
| Google Twill Cap                                         | 1        |


Overall, the data provides valuable insights into customer preferences, product popularity, and potential areas for marketing and merchandising strategies. Further analysis of historical data and integration with customer demographics could provide a more comprehensive understanding of these trends.


 **6.8 Calculate cohort map from product view to add_to_cart/number_product_view.**

~~~~sql

with product_data as(
select
    format_date('%Y%m', parse_date('%Y%m%d',date)) as month,
    count(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) as num_product_view,
    count(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) as num_add_to_cart,
    count(CASE WHEN eCommerceAction.action_type = '6' and product.productRevenue is not null THEN product.v2ProductName END) as num_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
,UNNEST(hits) as hits
,UNNEST (hits.product) as product
where _table_suffix between '20170101' and '20170331'
and eCommerceAction.action_type in ('2','3','6')
group by month
order by month
)

select
    *,
    round(num_add_to_cart/num_product_view * 100, 2) as add_to_cart_rate,
    round(num_purchase/num_product_view * 100, 2) as purchase_rate
from product_data;
~~~~

| month  | num_product_view | num_addtocart | num_purchase | add_to_cart_rate | purchase_rate |
|--------|------------------|---------------|--------------|------------------|---------------|
| 201701 | 25787            | 7342          | 4328         | 28.47            | 16.78         |
| 201702 | 21489            | 7360          | 4141         | 34.25            | 19.27         |
| 201703 | 23549            | 8782          | 6018         | 37.29            | 25.56         |


The table illustrates five various metrics and rates related to user behavior from January to March 2017.

Overall, the number of product views from January 2017 to March 2017 increased gradually. The add-to-cart rate and purchase rate also increased over the same period, indicating improved user engagement and conversion.

However, the add-to-cart rate and purchase rate are notably higher in March 2017, suggesting potential improvements in the website's user experience or marketing efforts.


<div id='cau7'/>


## 7. Conclusion

- This is the author's opportunity to learn about the marketing industry and the customer journey through this e-commerce dataset

- In analyzing the e-commerce dataset using BigQuery, the author understands customer behavior through the bounce rate, transaction, revenue, visit, and purchase.
  
- The author gained insights into which marketing channels drive traffic and sales by examining referral sources. Investing resources in effective channels and optimizing underperforming ones can improve marketing ROI.

- In conclusion, exploring the e-commerce dataset on BigQuery unearthed a wealth of insights critical for strategic decision-making to help the business can optimize operations, enhance customer experiences, and drive revenue.
