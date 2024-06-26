# Banking Customer Churn Analysis:
*01 May 2024*


![](https://images.stockcake.com/public/1/9/f/19fabc82-245f-4bda-9db7-de541d0912b7_large/banking-service-area-stockcake.jpg)


## Background
I obtained [this dataset](https://www.kaggle.com/datasets/radheshyamkollipara/bank-customer-churn) from Kaggle to hone my SQL skills. The subject matter intrigued me due to my interest in the banking industry, and the dataset offered ample opportunities for exploratory analysis. Churn analysis, like the one conducted here, holds significant importance for banks, enabling them to identify behavioral patterns among retained and churned customers. Understanding the factors surrounding customer churn is crucial, as recognizing trends can empower banks to take proactive measures, ultimately leading to enhanced retention rates. Insights derived from data-driven approaches, such as this project, have the potential to streamline management processes and enhance the overall banking experience for customers.

The primary goal was to scrutinize the dataset and unveil patterns or trends that could prompt bank management to intervene and mitigate churn rates. Given that acquiring new clients is considerably more expensive than retaining existing ones, the emphasis on data analysis in business becomes imperative.

## Data Integrity
The initial step involved ensuring the cleanliness and accuracy of the dataset to facilitate precise results. This entailed checking for any duplicates or null values within the dataset. Maintaining data integrity is paramount as it forms the foundation for delivering insights and any compromise in this aspect can undermine the credibility of the results.
```sql
-- I wish to analyse this dataset from a bank to try and predict the customer churn rate. 
-- First, I want to inspect the data and familiarise myself with the column headers:
SELECT *
FROM customers
LIMIT 5;

-- 1: Checking that the data contains no null values or duplicate customers:
SELECT COUNT(DISTINCT CustomerId) AS ids, Count(*) AS total_rows
FROM customers;
-- There are 10,000 rows filled with unique customer IDs and no null values
```


| RowNumber | CustomerId | Surname   | CreditScore | Geography | Gender | Age | Tenure | Balance    | NumOfProducts | HasCrCard | IsActiveMember | EstimatedSalary | Exited | Complain | Satisfaction Score | card_type | Point Earned |
|-----------|------------|-----------|-------------|-----------|--------|-----|--------|------------|---------------|-----------|----------------|-----------------|--------|----------|---------------------|-----------|---------------|
| 1         | 15634602   | Hargrave  | 619         | France    | Female | 42  | 2      | 0          | 1             | 1         | 1              | 101348.88       | 1      | 1        | 2                   | DIAMOND   | 464           |
| 2         | 15647311   | Hill      | 608         | Spain     | Female | 41  | 1      | 83807.86   | 1             | 0         | 1              | 112542.58       | 0      | 1        | 3                   | DIAMOND   | 456           |
| 3         | 15619304   | Onio      | 502         | France    | Female | 42  | 8      | 159660.8   | 3             | 1         | 0              | 113931.57       | 1      | 1        | 3                   | DIAMOND   | 377           |
| 4         | 15701354   | Boni      | 699         | France    | Female | 39  | 1      | 0          | 2             | 0         | 0              | 93826.63        | 0      | 0        | 5                   | GOLD      | 350           |
| 5         | 15737888   | Mitchell  | 850         | Spain     | Female | 43  | 2      | 125510.82  | 1             | 1         | 1              | 79084.1         | 0      | 0        | 5                   | GOLD      | 425           |




| ids   | total_rows |
|-------|------------|
| 10000 | 10000      |

![](https://images.stockcake.com/public/6/5/1/65172cc1-ecc1-494c-a5ca-f2fc52dd73f4_large/card-payment-processed-stockcake.jpg) 

## Exploratory Data Analysis
In order to identify potential correlations among the various variables, my initial approach was to gain a broad overview of the dataset. This allowed me to grasp the breadth of the data and begin the process of uncovering connections. This preliminary step was crucial in forming a cohesive understanding of the data and laying the groundwork for identifying potential relationships.

```sql
-- Calculating the total number of churned customers:
SELECT COUNT(*) AS total_churn
FROM customers
WHERE Exited = '1';
```
Before diving deeper into demographics, it was essential to figure out how many customers churned overall. This helped set the stage for later analyses, where I planned to break down specific groups as percentages of the total churned customers.


| total_churn |
|-------------|
| 2038        |


```sql

-- Looking at the breakdown of male and female customers and identifying the churn rate by gender:

WITH gender_table AS(
SELECT COUNT(*) AS gender_count, Gender
FROM customers AS c
GROUP BY Gender
) 

SELECT c.Gender, 
       gt.gender_count, 
       COUNT(Exited) AS churns,
       ROUND((COUNT(Exited)*100)/(SELECT Count(*) FROM customers WHERE Exited = '1'), 1) AS pct_gender
FROM customers AS c
LEFT JOIN gender_table gt ON c.Gender=gt.Gender
WHERE c.Exited = '1'
GROUP BY Gender, gt.gender_count;
```
In the initial part of this query, I utilized a Common Table Expression (CTE) to compute the total count of male and female customers, encompassing both churned and retained. Then, in the subsequent section, I constructed a join to analyze the count of male and female customers who churned, expressing these figures as percentages relative to the overall churned customer base.


| Gender | gender_count | churns | pct_gender |
|--------|--------------|--------|------------|
| Female | 4543         | 1139   | 55.9       |
| Male   | 5457         | 899    | 44.1       |


```sql
-- Exploring the link between age and churn rate:

SELECT 
	CASE 
    WHEN Age BETWEEN 10 AND 20  THEN '10-20'
    WHEN Age BETWEEN 20 AND 30 THEN '20-30'
    WHEN Age BETWEEN 30 AND 40 THEN '30-40'
	WHEN Age BETWEEN 40 AND 50 THEN '40-50'
    WHEN Age BETWEEN 50 AND 60 THEN '50-60'
    ELSE '60+'
    END AS age_group,
    COUNT(*) AS age_churns
FROM customers
WHERE Exited = '1'
GROUP BY age_group
ORDER BY age_group ASC;		
```

I opted to segment the age demographics into evenly spaced bins. This segmentation facilitates the bank in targeting specific demographics with tailored marketing and advertisements, thereby enhancing retention rates. This strategy not only boosts efficiency but also minimizes resource wastage and streamlines operational processes.


| age_group | age_churns |
|-----------|------------|
| 10-20     | 5          |
| 20-30     | 143        |
| 30-40     | 539        |
| 40-50     | 788        |
| 50-60     | 448        |
| 60+       | 115        |


```sql
-- Idenitfying the countries with the highest churn rates:
SELECT Geography, COUNT(*)
FROM customers
WHERE Exited = '1'
GROUP BY Geography;
```


| Geography | total_churned |
|-----------|---------------|
| France    | 811           |
| Spain     | 413           |
| Germany   | 814           |


```sql
-- Customer behaviours that affect churn rate.

-- A: CREDIT SCORE:
SELECT 
MAX(CreditScore),
MIN(CreditScore),
AVG(CreditScore)
FROM customers;

SELECT 
	CASE 
    WHEN CreditScore BETWEEN 350 AND 450 THEN '350-450'
	WHEN CreditScore BETWEEN 450 AND 550 THEN '450-550'
	WHEN CreditScore BETWEEN 550 AND 650 THEN '550-650'
	WHEN CreditScore BETWEEN 650 AND 750 THEN '650-750'
	WHEN CreditScore BETWEEN 750 AND 850 THEN '750-850'
ELSE 'N/A'
END AS credit_score_group,
COUNT(*) AS total
FROM customers
WHERE Exited = '1'
GROUP BY credit_score_group
ORDER BY credit_score_group ASC;
```


| credit_score_group | total |
|--------------------|-------|
| 350-450            | 61    |
| 450-550            | 307   |
| 550-650            | 689   |
| 650-750            | 667   |
| 750-850            | 314   |



```sql
-- B: Customer Complaints
SELECT COUNT(Complain) AS total_complaints, COUNT(Exited) as total_exits
FROM customers
WHERE Complain = '1'AND Exited = '1';

-- C: Type of bank card owned

ALTER TABLE customers
CHANGE COLUMN `Card Type` card_type VARCHAR(255);
SELECT COUNT(*) as total_exited, card_type
FROM customers
WHERE Exited = '1'
GROUP BY card_type;
```


| total_complaints | total_exits |
|------------------|-------------|
| 2034             | 2034        |


| total_exited | card_type |
|--------------|-----------|
| 546          | DIAMOND   |
| 502          | SILVER    |
| 482          | GOLD      |
| 508          | PLATINUM  |


```sql
-- D: Age and Gender churn rate
SELECT COALESCE(Gender, '') AS Gender, COALESCE(Geography, '') AS Country, Count(*) AS total_exits
FROM customers
WHERE Exited = '1'
GROUP BY Gender, Geography WITH ROLLUP;


-- Card Type churn rate percentage:
SELECT 
    card_type, 
    COUNT(*) AS total_exits, 
    ROUND((COUNT(*) * 100) / (SELECT COUNT(*) FROM customers WHERE Exited = '1'), 1) AS pct_card_type
FROM customers
WHERE Exited ='1'
GROUP BY card_type;
```

| Gender | Country | total_exits |
|--------|---------|-------------|
| Female | France  | 460         |
| Female | Germany | 448         |
| Female | Spain   | 231         |
| Female |         | 1139        |
| Male   | France  | 351         |
| Male   | Germany | 366         |
| Male   | Spain   | 182         |
| Male   |         | 899         |
|        |         | 2038        |


| card_type | total_exits | pct_card_type |
|-----------|-------------|---------------|
| DIAMOND   | 546         | 26.8          |
| SILVER    | 502         | 24.6          |
| GOLD      | 482         | 23.7          |
| PLATINUM  | 508         | 24.9          |


```sql
-- Top 3 salaries of those customers who exited the bank.
SELECT CustomerID, EstimatedSalary
FROM customers
WHERE Exited = '1'
ORDER BY EstimatedSalary DESC
LIMIT 3;

-- Balances of retained and churned customers:
SELECT 
    CASE 
        WHEN Exited = '1' THEN 'Churned'
        ELSE 'Retained' 
    END AS customer_status,
    MIN(Balance) AS min_balance,
    MAX(Balance) AS max_balance,
    ROUND(AVG(Balance), 2) AS avg_balance
FROM customers
GROUP BY customer_status;
```


| CustomerID | EstimatedSalary |
|------------|-----------------|
| 15815656   | 199808.1        |
| 15661670   | 199725.39       |
| 15672152   | 199693.84       |


| customer_status | min_balance | max_balance | avg_balance |
|-----------------|-------------|-------------|-------------|
| Churned         | 0           | 250898.09   | 91109.48    |
| Retained        | 0           | 221532.8    | 72742.75    |


```sql
-- Looking at custmer activity and the effect on churn rate:
SELECT
CASE
WHEN Exited = '1' THEN 'Churned'
ELSE 'Retained'
END AS cust_status,
COUNT(*),
CASE 
WHEN IsActiveMember = '1' THEN 'Active'
ELSE 'Inactive'
END AS cust_act
FROM customers
GROUP BY cust_status, cust_act;
```

| cust_status | cust_totals | cust_act  |
|-------------|-------------|-----------|
| Churned     | 735         | Active    |
| Retained    | 4416        | Active    |
| Churned     | 1303        | Inactive  |
| Retained    | 3546        | Inactive  |



## Findings & Recommnedations
- Female customers comprised approximately 56% of the churned customers. Based on this data, there isn't substantial evidence to suggest a significant gender disparity. Therefore, I recommend that the bank not prioritize addressing gender-related concerns at this time.
- Customers in the 40-50 age group exhibit the highest churn rate. To address this, I recommend the bank focus their marketing efforts and tailored services towards this demographic. This could involve offering personalized financial planning services, exclusive benefits suited to their life stage, or targeted promotions.
- France and Germany exhibited approximately twice the number of churned customers compared to Spain. In light of this, I recommend that the bank conduct a thorough review of its operations in these regions.
- It's noteworthy that all churned customers have a recorded complaint on file. This observation suggests a potential area for improvement within the bank's customer service department. Timely resolution of these issues could potentially lower churn rates. Prioritizing the needs and preferences of customers should be a focus. By addressing customer complaints promptly and effectively, the bank can enhance customer satisfaction and loyalty.
- Customer activity proves to be crucial, as inactive members who churned were twice the number of active members who churned. To address this, the bank should consider implementing engaging customer service initiatives and introducing more enticing programs such as loyalty rewards and bonuses. By enhancing the overall customer experience, the bank can encourage greater customer retention and loyalty.
