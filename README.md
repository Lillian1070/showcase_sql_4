# ss_sql_2
Retention Rate

[Retention Rate](https://platform.stratascratch.com/coding/2053-retention-rate?code_type=3)


Last Updated: June 2024

Meta
Salesforce
Hard
ID 2053

82

Data Engineer
Data Scientist
BI Analyst
Data Analyst
ML Engineer
Software Engineer
You are given a dataset that tracks user activity. The dataset includes information about the date of user activity, the account_id associated with the activity, and the user_id of the user performing the activity. Each row in the dataset represents a user’s activity on a specific date for a particular account_id.


Your task is to calculate the monthly retention rate for users for each account_id for December 2020 and January 2021. The retention rate is defined as the percentage of users active in a given month who have activity in any future month.


For instance, a user is considered retained for December 2020 if they have activity in December 2020 and any subsequent month (e.g., January 2021 or later). Similarly, a user is retained for January 2021 if they have activity in January 2021 and any later month (e.g., February 2021 or later).


The final output should include the account_id and the ratio of the retention rate in January 2021 to the retention rate in December 2020 for each account_id. If there are no users retained in December 2020, the retention rate ratio should be set to 0.

```sql
WITH 
    T AS (
        SELECT 
            account_id,
            user_id, 
            (SELECT COUNT(sub.record_date) 
            FROM sf_events sub
            WHERE sub.record_date BETWEEN '2020-12-01' AND '2020-12-31') AS Dec20_act,
            (SELECT COUNT(sub.record_date) 
            FROM sf_events sub
            WHERE sub.record_date BETWEEN '2021-01-01' AND '2021-01-31') AS Jan21_act,
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
