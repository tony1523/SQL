# [Case Study #3 - Foodie-Fi](https://8weeksqlchallenge.com/case-study-3/)
<img src="https://github.com/tony1523/SQL/blob/main/8_Week%20_SQL_Challenge/img/case_study_3_pic.png"  width="600" height="450" style="display: block;margin-left: auto;margin-right: auto;width: 50%">


--------------------------------------------------------------------------------

## Table Of Contents
* [Problem Statement](#Problem-Statement)
* [Entity Relationship Diagram](#Entity-Relationship-Diagram)
* [Dataset](#Dataset)
* [Case Study Questions](#Case-Study-Questions)
* [Solutions](#Solutions)

--------------------------------------------------------------------------------


## Entity Relationship Diagram 
<img src="https://github.com/tony1523/SQL/blob/main/8_Week%20_SQL_Challenge/img/entity_case_3.jpg"  style="display: block;margin-left: auto;margin-right: auto;width: 50%">
--------------------------------------------------------------------------------


## Problem Statement 
Subscription based businesses are super popular and Danny realised that there was a large gap in the market.
He wanted to create a new streaming service that only had food related content, something like Netflix but with only cooking shows!
Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions 
and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.


--------------------------------------------------------------------------------

## Dataset

### PLANS

Customers can choose which plans to join Foodie-Fi when they first sign up.
Basic plan customers have limited access and can only stream their videos and is only available monthly at $9.90
Pro plan customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.
Customers can sign up to an initial 7 day free trial will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
When customers cancel their Foodie-Fi service - they will have a churn plan record with a null price but their plan will continue until the end of the billing period.

| plan_id | plan_name     | price |
|---------|---------------|-------|
| 0       | trial         | 0     |
| 1       | basic monthly | 9.90  |
| 2       | pro monthly   | 19.90 |
| 3       | pro annual    | 199   |
| 4       | churn         | null  |

### SUBSCRIBTIONS
Customer subscriptions show the exact date where their specific plan_id starts.
If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the start_date in the subscriptions table will reflect the date that the actual plan changes.
When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.
When customers churn - they will keep their access until the end of their current billing period but the start_date will be technically the day they decided to cancel their service.

| customer_id | plan_id | start_date |
|-------------|---------|------------|
| 1           | 0       | 2020-08-01 |
| 1           | 1       | 2020-08-08 |
| 2           | 0       | 2020-09-20 |
| 2           | 3       | 2020-09-27 |
| 11          | 0       | 2020-11-19 |
| 11          | 4       | 2020-11-26 |
| 13          | 0       | 2020-12-15 |
| 13          | 1       | 2020-12-22 |
| 13          | 2       | 2021-03-29 |
| 15          | 0       | 2020-03-17 |
| 15          | 2       | 2020-03-24 |
| 15          | 4       | 2020-04-29 |
| 16          | 0       | 2020-05-31 |
| 16          | 1       | 2020-06-07 |
| 16          | 3       | 2020-10-21 |
| 18          | 0       | 2020-07-06 |
| 18          | 2       | 2020-07-13 |
| 19          | 0       | 2020-06-22 |
| 19          | 2       | 2020-06-29 |
| 19          | 3       | 2020-08-29 |

----------------------------------------------------------------------------

## Case Study Questions
## A. Customer Journey

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

## B. Data Analysis Questions
1. How many customers has Foodie-Fi ever had?
2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?
3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name?
4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
6. What is the number and percentage of customer plans after their initial free trial?
7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
8. How many customers have upgraded to an annual plan in 2020?
9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

---------------------------------------------------------------------------

## Solutions
### 1. How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT customer_id) AS number_of_customers
FROM foodie_fi.subscriptions;
```
| number_of_customers |
|---------------------|
| 1000                |

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?
```sql
SELECT TO_CHAR(s.start_date, 'Month')AS month,COUNT(s.customer_id)
FROM foodie_fi.subscriptions AS s
WHERE s.plan_id=0
GROUP BY 1;
```
| month     | count |
|-----------|-------|
| April     | 81    |
| August    | 88    |
| December  | 84    |
| February  | 68    |
| January   | 88    |
| July      | 89    |
| June      | 79    |
| March     | 94    |
| May       | 88    |
| November  | 75    |
| October   | 79    |
| September | 87    |

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name?
```sql
WITH cte_table AS
(SELECT customer_id,plan_name,CAST(TO_CHAR(s.start_date, 'yyyy') AS INTEGER)AS year_num
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p ON s.plan_id=p.plan_id)

SELECT plan_name,COUNT(customer_id)
FROM cte_table
WHERE year_num>=2021
GROUP BY 1
ORDER BY 1
```
| plan_name     | count |
|---------------|-------|
| basic monthly | 8     |
| churn         | 71    |
| pro annual    | 63    |
| pro monthly   | 60    |

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
DROP TABLE IF EXISTS total_count;
CREATE TEMP TABLE total_count AS (
    SELECT COUNT(DISTINCT s.customer_id) AS customer_count
  	FROM foodie_fi.subscriptions AS s
);

WITH cte_churn_table AS (
SELECT COUNT(s.customer_id) AS customer_churn_count
FROM foodie_fi.subscriptions AS s
WHERE plan_id=4)

SELECT cte.customer_churn_count, (cte.customer_churn_count::FLOAT/total_count.customer_count::FLOAT)*100 AS percentage_churn_customers
FROM cte_churn_table AS cte, total_count
```
| customer_churn_count | percentage_churn_customers |
|----------------------|----------------------------|
| 307                  | 30.7                       |

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
DROP TABLE IF EXISTS temp_table_next_plan ;
CREATE TEMP TABLE temp_table_next_plan AS(
SELECT *,LEAD(plan_id,1) OVER (
		PARTITION BY customer_id 
	) next_plan
  	FROM foodie_fi.subscriptions AS s);
    
WITH cte_table_churn as(
  SELECT COUNT(customer_id) AS num_churned_after_free_trial
  FROM temp_table_next_plan
  WHERE plan_id=0 AND next_plan=4
)
SELECT num_churned_after_free_trial,(num_churned_after_free_trial::float/customer_count::float)*100 AS perc_churned_after_free_trial
FROM cte_table_churn,total_count
```
| num_churned_after_free_trial | perc_churned_after_free_trial |
|------------------------------|-------------------------------|
| 92                           | 9.2                           |

### 6. What is the number and percentage of customer plans after their initial free trial?
```sql

```

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```sql

```

### 8. How many customers have upgraded to an annual plan in 2020?
```sql

```

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```sql

```

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

```sql

```


### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql

```
