# Atliq Hardware Data Analysis

## Project Background

Atliq Hardware is one of the leading computer hardware producers in India and well expanded in other countries too.
However, the management noticed that they do not get enough insights to make quick and smart data-informed decisions, which could help them stay ahead of the competition. They have decided to expand their data analytics team by hiring several junior data analysts. Tony Sharma, data analytics director is looking for candidates with both technical and soft skills. Hence, he has organized a SQL challenge as part of the recruitment process, where applicants will have to answer ten ad hoc requests to provide insights for the business.

## Understanding Data

After downloading, I imported the dataset into MYSQL workbench and explored the tables. It contains 209 customers, 397 products and 968796 sales records spanning over years 2019 to 2021. For join queries, I also looked at their relations and figured following about the tables:

- dim_customer: Contains customer-related data.
- dim_product: Contains product-related data.
- fact_gross_price: Contains gross price information for each product.
- fact_manufacuring_cost: Contains the cost incurred in the production of each product.
- fact_pre_invoice_deductions: Contains pre-invoice deductions information for each product.
- fact_sales_monthly: Contains monthly sales data for each product.

## Learnings from this SQL project

- Learnt how to create data modeling with an ER diagram.
- Got hands-on experience of working and dealing with large data sets.
- Learnt how to write SQL queries for getting output from tables.
- Understood some important concepts in SQL like Group By, Order By, Subqueries, Joins, CTE's, Aggregate Functions, etc.

## Data Model

I created the below ER diagram for referring to how these tables are interrelated to each other and used it for joins.

## Ad-hoc requests, query results and insights

### Q1. Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.

```SQL
SELECT
    DISTINCT market
FROM
    dim_customer
WHERE
    customer = "Atliq Exclusive"
    AND region = "APAC";
```

| market      |
| ----------- |
| India       |
| Indonesia   |
| Japan       |
| Philiphines |
| South Korea |
| Australia   |
| Newzealand  |
| Bangladesh  |

### Q2. What is the percentage of unique product increase in 2021 vs. 2020?

```SQL
WITH product_count as (
    SELECT
        COUNT(DISTINCT CASE WHEN b.fiscal_year = '2020' THEN a.product END) AS product_count_2020,
        COUNT(DISTINCT CASE WHEN b.fiscal_year = '2021' THEN a.product END) AS product_count_2021
    FROM dim_product AS a
    INNER JOIN fact_gross_price AS b
        ON a.product_code = b.product_code)

SELECT
    product_count_2020,
    product_count_2021,
    ROUND((product_count_2021 - product_count_2020) * 100 / product_count_2020, 2) AS percentage_change
FROM product_count
ORDER BY percentage_change DESC;
```

| product_count_2020 | product_count_2021 | percentage_change |
| ------------------ | ------------------ | ----------------- |
| 51                 | 68                 | 33.33             |

The table shows that there were 51 unique products in 2020, and this number increased to 68 in 2021 which is a 33.33% increase.
The growth in number of unique products is a positive indicator of Atliq Hardware’s performance. Atliq is producing more new products to meet customer’s demand.

### Q3. Provide a report with all the unique product counts for each segment and sort them in descending order of product counts.

```sql
SELECT
    segment,
    count(DISTINCT product_code) AS product_count
FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;
```

| segment     | product_count |
| ----------- | ------------- |
| Accessories | 20            |
| Peripherals | 20            |
| Notebook    | 17            |
| Storage     | 9             |
| Desktop     | 4             |
| Networking  | 3             |

The segment with the highest number of products are "Accessories" and "Peripherals" with 20 products each followed by "Notebook" with 17 products. The segment with the lowest number of products is "Networking" with only 3 products.

### Q4. Follow-up: Which segment had the most increase in unique products in 2021 vs 2020?

```sql
WITH product_count AS (
    SELECT
        segment,
        COUNT(DISTINCT CASE WHEN b.fiscal_year = '2020' THEN a.product END) AS product_count_2020,
        COUNT(DISTINCT CASE WHEN b.fiscal_year = '2021' THEN a.product END) AS product_count_2021
    FROM dim_product AS a
    INNER JOIN fact_gross_price AS b
        ON a.product_code = b.product_code
    GROUP BY segment)

SELECT
    segment,
    product_count_2020,
    product_count_2021,
    (product_count_2021 - product_count_2020) AS difference
FROM product_count
ORDER BY difference DESC, product_count_2021 DESC;
```

| segment     | product_count_2020 | product_count_2021 | difference |
| ----------- | ------------------ | ------------------ | ---------- |
| Accessories | 13                 | 19                 | 6          |
| Peripherals | 15                 | 20                 | 5          |
| Notebook    | 14                 | 16                 | 2          |
| Desktop     | 1                  | 3                  | 2          |
| Storage     | 6                  | 7                  | 1          |
| Networking  | 2                  | 3                  | 1          |

Accessories segment saw the largest increase of 6 products closely followed by Peripherals with 5 new products.

### Q5. Get the products that have the highest and lowest manufacturing costs

```SQL
SELECT
    m.product_code,
    p.product,
    m.manufacturing_cost
FROM dim_product AS p
INNER JOIN fact_manufacturing_cost AS m
    ON m.product_code = p.product_code
WHERE
    manufacturing_cost = (SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost)
    OR manufacturing_cost = (SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost);
```

| product_code | product               | manufacturing_cost |
| ------------ | --------------------- | ------------------ |
| A2118150101  | AQ Master wired x1 Ms | 0.892              |
| A6120110206  | AQ HOME Allin1 Gen 2  | 240.5364           |

Knowing the manufacturing costs of products is important for businesses to determine the profitability of each product. By comparing the manufacturing cost to the selling price, businesses can determine the profit margin of each product and make informed decisions about pricing and production. We can see "AQ Master wired x1 Ms" product has the minimum manufacturing cost of Rs. 0.892. And "AQ HOME Allin1 Gen 2" product has the maximum cost of Rs. 240.53.

### Q6. Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market.

```sql
SELECT
    a.customer,
    b.customer_code,
    ROUND(AVG(B.pre_invoice_discount_pct) * 100, 2) AS average_discount_percentage
FROM dim_customer AS a
INNER JOIN fact_pre_invoice_deductions AS b
    ON a.customer_code = b.customer_code
WHERE
    b.fiscal_year = '2021'
    AND a.market = 'India'
GROUP BY a.customer_code, a.customer
ORDER BY average_discount_percentage DESC
LIMIT 5;
```

| customer | customer_code | average_discount_percentage |
| -------- | ------------- | --------------------------- |
| Flipkart | 90002009      | 30.83                       |
| Viveks   | 90002006      | 30.38                       |
| Ezone    | 90002003      | 30.28                       |
| Croma    | 90002002      | 30.25                       |
| Amazon   | 90002016      | 29.33                       |

Flipkart has the highest average discount percentage of 30.83% very closely followed by Viveks at 30.38%, and Ezone at 30.28%. Knowing the average discount percentage of customers can be useful for companies to understand their pricing strategies and competitiveness in the market. Companies may offer discounts as a way to attract and retain customers, but too high of a discount percentage could potentially hurt profitability.

### Q7. Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month

```SQL
SELECT
    EXTRACT(YEAR FROM B.date) AS Year,
    EXTRACT(MONTH FROM B.date) AS Month,
    ROUND(SUM(B.sold_quantity * C.gross_price), 2) AS 'Gross sales amount'
FROM dim_customer AS A
INNER JOIN fact_sales_monthly AS B
    ON A.customer_code = B.customer_code
INNER JOIN fact_gross_price AS C
    ON B.product_code = C.product_code
WHERE A.customer = 'Atliq Exclusive'
GROUP BY Year, Month
ORDER BY Year, Month;
```

| Year | Month | Gross sales amount |
| ---- | ----- | ------------------ |
| 2019 | 9     | 9092670.34         |
| 2019 | 10    | 10378637.6         |
| 2019 | 11    | 15231894.97        |
| 2019 | 12    | 9755795.06         |
| 2020 | 1     | 9584951.94         |
| 2020 | 2     | 8083995.55         |
| 2020 | 3     | 766976.45          |
| 2020 | 4     | 800071.95          |
| 2020 | 5     | 1586964.48         |
| 2020 | 6     | 3429736.57         |
| 2020 | 7     | 5151815.4          |
| 2020 | 8     | 5638281.83         |
| 2020 | 9     | 19530271.3         |
| 2020 | 10    | 21016218.21        |
| 2020 | 11    | 32247289.79        |
| 2020 | 12    | 20409063.18        |
| 2021 | 1     | 19570701.71        |
| 2021 | 2     | 15986603.89        |
| 2021 | 3     | 19149624.92        |
| 2021 | 4     | 11483530.3         |
| 2021 | 5     | 19204309.41        |
| 2021 | 6     | 15457579.66        |
| 2021 | 7     | 19044968.82        |
| 2021 | 8     | 11324548.34        |

The table shows that gross sales fluctuated significantly over time. Additionally, the table shows a general increasing trend in gross sales amount from September 2019 to November 2020, followed by a decrease in gross sales amount from December 2020 and February 2021.

Atliq Exclusive's best-performing months in terms of gross sales are October, November and December of 2020, with sales of Rs 21M, Rs 32M, and Rs 20M respectively. Lower-performing months are March and April 2020 with just Rs 766k and Rs 800k sales respectively. This may be due to the COVID-19 pandemic. There seems to be a seasonal trend in Atliq Exclusive's sales, with higher sales during the months of September to December, and lower sales during the months of January to April. These insights can help Atliq Exclusive make strategic decisions, such as focusing on marketing and promotions during the months of September to December, and planning for inventory and staffing needs based on the seasonal trends.

### Q8. Which quarter of 2020 got the maximum total_sold_quantity?

```SQL
SELECT
    CASE
	    WHEN monthname(date) IN ('September','October','November') THEN "Q1"
	    WHEN monthname(date) IN ('December','January','February') THEN "Q2"
	    WHEN monthname(date) IN ('March','April','May') THEN "Q3"
	    WHEN monthname(date) IN ('June','July','August') THEN "Q4"
    END AS Quarter,
    SUM(sold_quantity) AS total_sales
FROM fact_sales_monthly
WHERE fiscal_year = "2020"
GROUP BY Quarter
ORDER BY Quarter;
```

| Quarter | total_sales |
| ------- | ----------- |
| Q1      | 7005619     |
| Q2      | 6649642     |
| Q3      | 2075087     |
| Q4      | 5042541     |

The total quantity of products sold in Quarter 1 of 2020 was 7,005,619, which was the highest total quantity of products sold for any quarter in the time period specified. It went into a decline until Q4 probably due to Covid-19 pandemic but total sold quantity significantly increased in Q4 and this may be attributed to the increase in demand for hardware such as desktop, notebook as students shifted to online study and many jobs shifted to online work during this time.

### Q9. Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution?

```SQL
WITH channel_sales AS (
    SELECT
        A.channel,
        sum(B.sold_quantity * C.gross_price) AS gross_sales
    FROM dim_customer AS A
    INNER JOIN fact_sales_monthly AS B
        ON A.customer_code = B.customer_code
    INNER JOIN fact_gross_price AS C
        ON B.product_code = C.product_code
    WHERE B.fiscal_year='2021'
    GROUP BY A.channel)

SELECT
    channel,
    round(gross_sales, 2) AS total_sales,
    round(gross_sales * 100 / (SELECT SUM(gross_sales) FROM channel_sales), 2) AS percentage_contribution
FROM channel_sales
GROUP BY channel, gross_sales
ORDER BY gross_sales DESC;
```

| channel     | total_sales     | percentage_contribution |
| ----------- | --------------- | ----------------------- |
| Retailer    |   1924170397.91 |                   73.22 |
| Direct      |    406686873.90 |                   15.47 |
| Distributor |    297175879.72 |                   11.31 |


The data suggests that the company's sales are heavily dependent on the retailer channel, as it accounts for the majority of gross sales. The company may want to consider diversifying its sales channels to reduce its reliance on retailers and increase sales through other channels such as direct or distributor.s

### Q10. Get divisions, their segments and count of products in them

```SQL
SELECT
    division,
    segment,
	count(*) AS segment_count
FROM dim_product
GROUP BY division, segment
ORDER BY segment_count DESC;
```

| division | segment     | segment_count |
| -------- | ----------- | ------------- |
| PC       | Notebook    | 129           |
| P & A    | Accessories | 116           |
| P & A    | Peripherals | 84            |
| PC       | Desktop     | 32            |
| N & S    | Storage     | 27            |
| N & S    | Networking  | 9             |



### Q11. Answer the below questions:

A. List all platforms AtliQ Hardware has

```SQL
select
    platform
from dim_customer
group by platform;
```

| platform       |
| -------------- |
| Brick & Mortar |
| E-Commerce     |

B. List all type of channels AtliQ Hardware uses

```SQL
select
    channel
from dim_customer
group by channel;
```

| channel     |
| ----------- |
| Direct      |
| Distributor |
| Retailer    |

C. List regions where AtliQ Hardware has its customers

```SQL
select
    region
from dim_customer
group by region;
```

| region |
| ------ |
| APAC   |
| EU     |
| NA     |
| LATAM  |

### Q12. Get customers who are getting the highest pre-invoice discount from AtliQ

```SQL
select
    pre.customer_code,
    c.customer,
    max(pre_invoice_discount_pct) as max_discount
from fact_pre_invoice_deductions as pre
inner join dim_customer as c
    on pre.customer_code = c.customer_code
group by customer_code, customer
order by max_discount desc
limit 10;
```

| customer_code | customer      | max_discount |
| ------------- | ------------- | ------------ |
|      90001021 | Taobao             |       0.3095 |
|      90013122 | Radio Popular      |       0.3093 |
|      90021090 | Radio Popular      |       0.3091 |
|      80006155 | Novus              |       0.3091 |
|      90020099 | Integration Stores |       0.3091 |
|      90019203 | Amazon             |       0.3087 |
|      90002009 | Flipkart           |       0.3083 |
|      90023026 | Relief             |       0.3080 |
|      90015148 | Boulanger          |       0.3076 |
|      90014141 | Amazon             |       0.3074 |


Taobao customers are getting the highest pre deduction discount from AtliQ.