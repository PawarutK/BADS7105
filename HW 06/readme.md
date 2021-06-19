# Customer Segmentation


## Import Data from Google BigQuery

The supermarket dataset contains 956K records between year 2006 to 2008. Features are created during query data from Google BigQuery which contain:
- **TOTAL_SPEND** = Total spending for each customer
- **AVG_MONTHLY_SPEND** = Average monthly spending for each customer
- **STD_MONTLY_SPEND** = Standard deviation of monthly spending for each customer
- **TOTAL_VISIT** = Total visit for each customer
- **AVG_MONTHLY_VISIT** = Average monthly visit for each customer
- **STD_MONTLY_VISIT** = Standard deviation of monthly visit for each customer
- **MODE_BASKET_SIZE** = Favorite basket site for each customer
- **DURATION_FROM_FIRST_PURCHASE** = Time since first purchase to now (2008-07-06) for each customer
- **DURATION_FROM_LAST_PURCHASE** = Time since last purchase to now (2008-07-06) for each customer
- **CUST_LIFETIME** = Time between first purchase and last purchase for each customer

```sql
SELECT
    S.CUST_CODE,
    SUM(S.SPEND) AS TOTAL_SPEND,
    AVG(M.MONTHLY_SPEND) AS AVG_MONTHLY_SPEND,
    STDDEV_POP(M.MONTHLY_SPEND) AS STD_MONTHLY_SPEND,
    COUNT(DISTINCT S.BASKET_ID) AS TOTAL_VISIT,
    AVG(M.MONTHLY_VISIT) AS AVG_MONTHLY_VISIT,
    STDDEV_POP(M.MONTHLY_VISIT) AS STD_MONTHLY_VISIT,
    CASE
    WHEN SUM(CASE WHEN S.BASKET_SIZE='L' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='S' THEN 1 ELSE 0 END)
     AND SUM(CASE WHEN S.BASKET_SIZE='L' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='M' THEN 1 ELSE 0 END) THEN 3
    WHEN SUM(CASE WHEN S.BASKET_SIZE='M' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='S' THEN 1 ELSE 0 END)
     AND SUM(CASE WHEN S.BASKET_SIZE='M' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='L' THEN 1 ELSE 0 END) THEN 2
    WHEN SUM(CASE WHEN S.BASKET_SIZE='S' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='M' THEN 1 ELSE 0 END)
     AND SUM(CASE WHEN S.BASKET_SIZE='S' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='L' THEN 1 ELSE 0 END) THEN 1
    END AS MODE_BASKET_SIZE,
    DATE_DIFF(PARSE_DATE('%Y%m%d', CAST(MAX(S.SHOP_DATE) AS STRING)), PARSE_DATE('%Y%m%d', CAST(MIN(S.SHOP_DATE) AS STRING)), DAY) AS CUST_LIFETIME,
    DATE_DIFF(DATE'2008-07-06', PARSE_DATE('%Y%m%d', CAST(MIN(S.SHOP_DATE) AS STRING)), DAY) AS DURATION_FROM_FIRST_PURCHASE,
    DATE_DIFF(DATE'2008-07-06', PARSE_DATE('%Y%m%d', CAST(MAX(S.SHOP_DATE) AS STRING)), DAY) AS DURATION_FROM_LAST_PURCHASE
FROM
    `hw10-cma.customer.supermarket` AS S
INNER JOIN (
    SELECT
        CUST_CODE,
        LEFT(CAST(SHOP_DATE AS STRING), 6) AS SHOP_MONTH,
        SUM(SPEND) AS MONTHLY_SPEND,
        COUNT(DISTINCT BASKET_ID) AS MONTHLY_VISIT
    FROM
        `hw10-cma.customer.supermarket`
    GROUP BY
        CUST_CODE, SHOP_MONTH
) AS M ON S.CUST_CODE = M.CUST_CODE
WHERE
    S.CUST_CODE IS NOT NULL
GROUP BY
    S.CUST_CODE
ORDER BY
    S.CUST_CODE
```

## Feature Extraction

Regarding all features queried from Google BigQuery, some features are self correlated. This is trial and error step to see which parameters are neccessary and meaningful for K-Mean clustering model in next step. Here, 7 features are selected as detailed below.

- ~~TOTAL_SPEND~~
- AVG_MONTHLY_SPEND
- STD_MONTLY_SPEND
- ~~TOTAL_VISIT~~
- AVG_MONTHLY_VISIT
- STD_MONTLY_VISIT
- MODE_BASKET_SIZE
- ~~DURATION_FROM_FIRST_PURCHASE~~
- DURATION_FROM_LAST_PURCHASE
- CUST_LIFETIME

## Clustering Model

To select how many cluster should be used for the supermarket dataset, Elbow method and Silhouette is applied. From the pictures below, 3-cluster (K=3) is selected because the more number of clusters, the less Silhouette.

![]()

## Clustering Analysis

This step is to analyze which feature is importance and how each cluster behaves.

### Feature Importance

![]()

It is found that top 5 the most importance features are
- CUST_LIFETIME
- DURATION_FROM_LAST_PURCHASE
- STD_MONTHLY_SPEND
- STD_MONTHLY_VISIT
- AVG_MONTHLY_VISIT

### Exploratory Data Analysis

![]()

## Interpretation

From the density plots, it can be concluded as below.
- **CHURNED**
  - One time purchase
  - Not active for over 1 year
- **ACTIVE**
  - Still active within 3 months
  - Visit not frequently (< 4 times/month)
  - Spend less money per visit (< 300 USD/time)
- **PREMIUM**
  - Still active within 3 months
  - Visit frequently (>= 4 times/month)
  - Spend much money per visit (>= 300 USD/time)

| GROUP | COUNT | TOTAL_SPEND | AVG_MONTHLY_SPEND | STD_MONTHLY_SPEND | TOTAL_VISIT | AVG_MONTHLY_VISIT | STD_MONTHLY_VISIT | MODE_BASKET_SIZE | DURATION_FROM_FIRST_PURCAHSE | DURATION_FROM_LAST_PURCHASE | CUST_LIFETIME | NAME |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|1|3136|23.6 USD/MONTH <br />|8.2 USD/MONTH|0.9 USD/MONTH|2 TIMES/MONTH|1 TIMES/MONTH|0 TIMES/MONTH|SMALL-MEDIUM|479.4 DAYS|421.7 DAYS|57.7 DAYS|CHURNED|
|2|2480|3,215.3 USD/MONTH|18.0 USD/MONTH|11.4 USD/MONTH|18 TIMES/MONTH|2 TIMES/MONTH|0 TIMES/MONTH|MEDIUM-LARGE|658.1 DAYS|62.5 DAYS|595.6 DAYS|ACTIVE|
|3|484|52,218.7 USD/MONTH|86.6 USD/MONTH|44.7 USD/MONTH|147 TIMES/MONTH|6 TIMES/MONTH|3 TIMES/MONTH|MEDIUM-LARGE|751.8 DAYS|10.9 DAYS|740.9 DAYS|PREMIUM|
