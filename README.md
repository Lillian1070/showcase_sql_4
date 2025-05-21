# [SQL] Calculating Retention Rate Ratio

_This SQL practice is based on a problem from [StrataScratch](https://platform.stratascratch.com/coding/2053-retention-rate?code_type=3), used here for personal learning and educational purposes._


- **Objective**: Calculate the ratio of the retention rate in January 2021 to the one in December 2020 for each account ID.
- **Practice Purpose**: Self-learning and reinforcement of SQL data cleaning, aggregation, joins, subqueries, and window functions.
- **Outline**:
    - [**Practice**](#section-1) (practice problem and query output)
    - [**Solution**](#section-2) (step-by-step explanation)
    - [**Future Enhancements**](#section-3)




## <a name="section-1"></a>🧪 Practice

The dataset tracks user activity. It includes information about the date of user activity, the `account_id` associated with the activity, and the `user_id` of the user performing the activity. Each row in the dataset represents a user’s activity on a specific date for a particular `account_id`.


### Task 

Calculate the monthly retention rate for users for each `account_id` for December 2020 and January 2021. The retention rate is defined as the percentage of users active in a given month who have activity in any future month.


For instance, a user is considered retained for December 2020 if they have activity in December 2020 and any subsequent month (e.g., January 2021 or later). Similarly, a user is retained for January 2021 if they have activity in January 2021 and any later month (e.g., February 2021 or later).


The final output should include the `account_id` and the ratio of the retention rate in January 2021 to the retention rate in December 2020 for each `account_id`. If there are no users retained in December 2020, the retention rate ratio should be set to 0.


- Table: `sf_events`

| Column Name   | Type    |
|---------------|---------|
| record_date   | date    |
| account_id    | varchar |
| user_id       | varchar |


### Expected Output

| account_id | retention  |
|------------|------------|
| A1         | 1          |
| A2         | 1          |
| A3         | 0          |


## <a name="section-2"></a>🧠 Solution

*This section outlines my thought process for solving the problem.*

### Step 1: Identify Required Data

- Extract unique users (`user_id`) per `account_id` who were active in December 2020 and January 2021.
- Determine if each user had activity in any future month.


### Step 2a: Identify Active Users in December 2020

- Create a temp table `T1` to store users active in December 2020 and January 2021.
- Mark the latest activity date for each user per account ID.

```sql
WITH 
    T AS (
        SELECT 
            account_id,
            user_id, 
            (
                SELECT COUNT(sub.record_date) 
                FROM sf_events sub
                WHERE sub.record_date BETWEEN '2020-12-01' AND '2020-12-31'
            ) AS Dec20_act,
            (
                SELECT COUNT(sub.record_date) 
                FROM sf_events sub
                WHERE sub.record_date BETWEEN '2021-01-01' AND '2021-01-31'
            ) AS Jan21_act,
            MAX(record_date) AS max_date
        FROM sf_events
        GROUP BY account_id, user_id
        ORDER BY account_id, user_id
    ), 
```


The temporary table `T1` should be similar to what we have below. 

| account_id | user_id    | Dec20_act  | Jan21_act  | max_date   |
|------------|------------|------------|------------|------------|
| A1	     | U1	      | 10         | 9	        | 2021-02-07 |
| A1	     | U2	      | 10         | 9	        | 2021-02-10 |
| A1	     | U3	      | 10         | 9	        | 2021-01-06 |
| A1	     | U8	      | 10         | 9	        | 2020-12-05 |
| A2	     | U4	      | 10         | 9	        | 2021-02-01 |
| A2	     | U5	      | 10         | 9	        | 2021-02-01 |
| A3	     | U6	      | 10         | 9	        | 2021-01-14 |
| A3	     | U7	      | 10         | 9	        | 2020-12-06 |



### Step 2b: Identify Active Users in January 2021

- Create another temp table `T2` to check if users were retained in the future months of December 2020 and January 2021.

```sql
    T2 AS (
        SELECT 
            account_id,
            (CASE WHEN Dec20_act > 0 AND max_date > '2020-12-31' THEN COUNT(user_id) END) AS Dec20_ret,
            (CASE WHEN Jan21_act > 0 AND max_date > '2021-01-31' THEN COUNT(user_id) END) AS Jan21_ret
        FROM T
        GROUP BY account_id
        ORDER BY account_id
    )

```


The temporary table `T2` should be similar to what we have below. 

| account_id | Dec20_ret  | Jan21_ret  |
|------------|------------|------------|
| A1	     | 4	      | 4          |
| A2	     | 2	      | 2          |
| A3         | 2          |            |


### Step 3: Calculate Retention Rate Ratio

- Compute the retention rate for both months per `account_id`.
- Take the ratio of January's retention rate to December's.
- If December has zero retained users, return `0` to avoid division by zero using `IFNULL()` function.

```sql
SELECT 
    account_id, 
    IFNULL(Jan21_ret/Dec20_ret, 0) AS retention
FROM T2;
```


#### Final Syntax and Output using MySQL

##### * Syntax

```sql
WITH 
    T AS (
        SELECT 
            account_id,
            user_id, 
            (
                SELECT COUNT(sub.record_date) 
                FROM sf_events sub
                WHERE sub.record_date BETWEEN '2020-12-01' AND '2020-12-31'
            ) AS Dec20_act,
            (
                SELECT COUNT(sub.record_date) 
                FROM sf_events sub
                WHERE sub.record_date BETWEEN '2021-01-01' AND '2021-01-31'
            ) AS Jan21_act,
            MAX(record_date) AS max_date
        FROM sf_events
        GROUP BY account_id, user_id
        ORDER BY account_id, user_id
    ), 
    T2 AS (
        SELECT 
            account_id,
            (CASE WHEN Dec20_act > 0 AND max_date > '2020-12-31' THEN COUNT(user_id) END) AS Dec20_ret,
            (CASE WHEN Jan21_act > 0 AND max_date > '2021-01-31' THEN COUNT(user_id) END) AS Jan21_ret
        FROM T
        GROUP BY account_id
        ORDER BY account_id
    )

SELECT 
    account_id, 
    IFNULL(Jan21_ret/Dec20_ret, 0) AS retention
FROM T2;
```

##### * Output

| account_id | retention  |
|------------|------------|
| A1         | 1          |
| A2         | 1          |
| A3         | 0          |



## <a name="section-3"></a>🚀 Future Enhancements

- Expanding solutions with additional SQL databases.

- Adding performance benchmarking for queries.

- Incorporating visual explanations where applicable.




_💬 I’d love to hear your thoughts! If you have any suggestions or questions, feel free to connect with me._
