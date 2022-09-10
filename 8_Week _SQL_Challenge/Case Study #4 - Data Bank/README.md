# [Case Study #4 - Data Bank](https://8weeksqlchallenge.com/case-study-4/)
<img src="https://github.com/tony1523/SQL/blob/main/8_Week%20_SQL_Challenge/img/case_study_4_pic.png"  width="600" height="450" style="display: block;margin-left: auto;margin-right: auto;width: 50%">


--------------------------------------------------------------------------------

## Table Of Contents
* [Problem Statement](#Problem-Statement)
* [Entity Relationship Diagram](#Entity-Relationship-Diagram)
* [Dataset](#Dataset)
* [Case Study Questions](#Case-Study-Questions)
* [Solutions](#Solutions)

--------------------------------------------------------------------------------


## Entity Relationship Diagram 
<img src="https://github.com/tony1523/SQL/blob/main/8_Week%20_SQL_Challenge/img/entity_case_4.jpg"  style="display: block;margin-left: auto;margin-right: auto;width: 50%">
--------------------------------------------------------------------------------


## Problem Statement 
There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches. Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank! This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!


--------------------------------------------------------------------------------

## Dataset

### REGIONS
This regions table contains the region_id and their respective region_name values.
| region_id | region_name |
|-----------|-------------|
| 1         | Africa      |
| 2         | America     |
| 3         | Asia        |
| 4         | Europe      |
| 5         | Oceania     |

### CUSTOMER NODES
Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data.
Below is a sample of the top 10 rows of the data_bank.customer_nodes.
| customer_id | region_id | node_id | start_date | end_date   |
|-------------|-----------|---------|------------|------------|
| 1           | 3         | 4       | 2020-01-02 | 2020-01-03 |
| 2           | 3         | 5       | 2020-01-03 | 2020-01-17 |
| 3           | 5         | 4       | 2020-01-27 | 2020-02-18 |
| 4           | 5         | 4       | 2020-01-07 | 2020-01-19 |
| 5           | 3         | 3       | 2020-01-15 | 2020-01-23 |
| 6           | 1         | 1       | 2020-01-11 | 2020-02-06 |
| 7           | 2         | 5       | 2020-01-20 | 2020-02-04 |
| 8           | 1         | 2       | 2020-01-15 | 2020-01-28 |
| 9           | 4         | 5       | 2020-01-21 | 2020-01-25 |
| 10          | 3         | 4       | 2020-01-13 | 2020-01-14 |

### CUSTOMER TRANSACTIONS
This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.
| customer_id | txn_date   | txn_type | txn_amount |
|-------------|------------|----------|------------|
| 429         | 2020-01-21 | deposit  | 82         |
| 155         | 2020-01-10 | deposit  | 712        |
| 398         | 2020-01-01 | deposit  | 196        |
| 255         | 2020-01-14 | deposit  | 563        |
| 185         | 2020-01-29 | deposit  | 626        |
| 309         | 2020-01-13 | deposit  | 995        |
| 312         | 2020-01-20 | deposit  | 485        |
| 376         | 2020-01-03 | deposit  | 706        |
| 188         | 2020-01-13 | deposit  | 601        |
| 138         | 2020-01-11 | deposit  | 520        |


----------------------------------------------------------------------------

## Case Study Questions
### A. Customer Nodes Exploration
1. How many unique nodes are there on the Data Bank system?
2. What is the number of nodes per region?
3. How many customers are allocated to each region?
4. How many days on average are customers reallocated to a different node?
5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
### B. Customer Transactions
1. What is the unique count and total amount for each transaction type?
2. What is the average total historical deposit counts and amounts for all customers?
3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
4. What is the closing balance for each customer at the end of the month?
5. What is the percentage of customers who increase their closing balance by more than 5%?

---------------------------------------------------------------------------

## Solutions
## A. Customer Nodes Exploration
### 1. How many unique nodes are there on the Data Bank system?
```sql
WITH cte_num_nodes as (
SELECT DISTINCT cn.node_id
FROM data_bank.customer_nodes AS cn)
SELECT COUNT(node_id) AS number_unique_nodes
FROM cte_num_nodes;
```
| number_unique_nodes |
|---------------------|
| 5                   |

### 2. What is the number of nodes per region?
```sql
SELECT r.region_name,COUNT(cn.node_id) AS number_of_nodes
FROM data_bank.customer_nodes AS cn
JOIN data_bank.regions r on r.region_id=cn.region_id
GROUP BY 1
ORDER BY 1;
```
| region_name | number_of_nodes |
|-------------|-----------------|
| Africa      | 714             |
| America     | 735             |
| Asia        | 665             |
| Australia   | 770             |
| Europe      | 616             |

### 3. How many customers are allocated to each region?
```sql
SELECT r.region_name,COUNT(DISTINCT cn.customer_id) AS number_of_customers
FROM data_bank.customer_nodes AS cn
JOIN data_bank.regions r on r.region_id=cn.region_id
GROUP BY 1
ORDER BY 1;
```
| region_name | number_of_customers |
|-------------|---------------------|
| Africa      | 102                 |
| America     | 105                 |
| Asia        | 95                  |
| Australia   | 110                 |
| Europe      | 88                  |

### 4. How many days on average are customers reallocated to a different node?
```sql
SELECT ROUND(AVG(cn.end_date-cn.start_date),2) AS num_days
FROM data_bank.customer_nodes AS cn
WHERE cn.end_date != '9999-12-31';
```
| num_days |
|----------|
| 14.63    |

### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```sql
WITH cte_avg_days AS (
SELECT cn.end_date-cn.start_date AS num_days
FROM data_bank.customer_nodes AS cn
WHERE cn.end_date != '9999-12-31')
SELECT  PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY num_days) AS median,
		PERCENTILE_CONT(0.8) WITHIN GROUP(ORDER BY num_days) AS "80th_percentile",
        PERCENTILE_CONT(0.95) WITHIN GROUP(ORDER BY num_days) AS "95th_percentile"
FROM cte_avg_days;
```
| median | 80th_percentile | 95th_percentile |
|--------|-----------------|-----------------|
| 15     | 23              | 28              |

## B. Customer Transactions

### 1. What is the unique count and total amount for each transaction type?
```sql
SELECT ct.txn_type, COUNT(ct.txn_type), SUM(ct.txn_amount) AS total_amount
FROM data_bank.customer_transactions AS ct
GROUP BY 1;
```
| txn_type   | count | total_amount |
|------------|-------|--------------|
| purchase   | 1617  | 806537       |
| deposit    | 2671  | 1359168      |
| withdrawal | 1580  | 793003       |

### 2. What is the average total historical deposit counts and amounts for all customers?
```sql
WITH cte_avg_table as (
SELECT ct.customer_id, COUNT(ct.txn_type) AS countamount, SUM(ct.txn_amount) AS total_amount
FROM data_bank.customer_transactions AS ct
WHERE ct.txn_type='deposit'
GROUP BY 1)
SELECT ROUND(AVG(countamount),2) AS avg_deposit_count, ROUND(AVG(total_amount),2) AS avg_total_amount
FROM cte_avg_table;
```
| avg_deposit_count | avg_total_amount |
|-------------------|------------------|
| 5.34              | 2718.34          |

### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
WITH cte_table_count as(
SELECT ct.customer_id, TO_CHAR(ct.txn_date, 'Month')AS month,
SUM(CASE WHEN ct.txn_type='deposit' THEN 1 ELSE 0 END) AS deposit_count,
SUM(CASE WHEN ct.txn_type='purchase' THEN 1 ELSE 0 END)AS purchase_count,
SUM(CASE WHEN ct.txn_type='withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
FROM data_bank.customer_transactions AS ct
GROUP BY 1,2)
 
SELECT month,COUNT(customer_id)
FROM cte_table_count
WHERE deposit_count>1 AND (purchase_count>=1 OR withdrawal_count>=1)
GROUP BY 1;
```
| month    | count |
|----------|-------|
| January  | 168   |
| February | 181   |
| March    | 192   |
| April    | 70    |

### 4. What is the closing balance for each customer at the end of the month?
```sql
WITH cte_balance_table as(
SELECT ct.customer_id, TO_CHAR(ct.txn_date, 'Month')AS month,
SUM(CASE WHEN ct.txn_type='deposit' THEN ct.txn_amount ELSE -ct.txn_amount END) AS balance
FROM data_bank.customer_transactions AS ct
GROUP BY 1,2
ORDER BY 1,2)
 
SELECT customer_id,month,balance,
sum(balance) over(PARTITION BY customer_id
ORDER BY month ROWS BETWEEN UNBOUNDED preceding AND CURRENT ROW) AS closing_balance
FROM cte_balance_table
```
| customer_id | month    | balance | closing_balance |
|-------------|----------|---------|-----------------|
| 1           | January  | 312     | 312             |
| 1           | March    | -952    | -640            |
| 2           | January  | 549     | 549             |
| 2           | March    | 61      | 610             |
| 3           | April    | 493     | 493             |
| 3           | February | -965    | -472            |
| 3           | January  | 144     | -328            |
| 3           | March    | -401    | -729            |
| 4           | January  | 848     | 848             |
| 4           | March    | -193    | 655             |

### 5. What is the percentage of customers who increase their closing balance by more than 5%?
