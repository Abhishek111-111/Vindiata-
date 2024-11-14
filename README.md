# Vindiata- Assignment
 ABC is a real-money online gaming company providing multiplayer games such as Ludo. An user can register as a player, 
deposit money in the platform and play games with other players on the platform. 

If he/she wins the game then they can withdraw the winning amount while the platform charges a nominal fee for the services.
 
To retain players on the platform, the company ABC gives loyalty points to their players based on their activity on the platform.
 
Loyalty points are calculated on the basis of the number of games played, deposits and withdrawal made on the platform by a particular player.
 
The criteria to convert number of games played, deposits and withdrawal into points is given as below :


![image](https://github.com/user-attachments/assets/44c3b146-7835-402a-8899-336bfb71a6b7)

### Final Loyalty Point Formula
Loyalty Point = (0.01 * deposit) + (0.005 * Withdrawal amount) + (0.001 * (maximum of (#deposit - #withdrawal) or 0)) + (0.2 * Number of games played)

At the end of each month total loyalty points are alloted to all the players. Out of which the top 50 players are provided cash benefits.

### Part A - Calculating loyalty points

On each day, there are 2 slots for each of which the loyalty points are to be calculated:
S1 from 12am to 12pm 
S2 from 12pm to 12am

### Dividing slots based on time

Part A - Calculating loyalty points

On each day, there are 2 slots for each of which the loyalty points are to be calculated:
S1 from 12am to 12pm 
S2 from 12pm to 12am

import pandas as pd
import numpy as np
```python
# Reading User gameplay data, withdrawal data and deposit data

gp = pd.read_csv("User_gameplay_data.csv")
wd = pd.read_csv("Withdrawal_data.csv")
dp = pd.read_csv("Deposit_data.csv")

gp.info()

### Splitting date time for creating slot 1 and slot 2 according to time


gp["Datetime"] = pd.to_datetime(gp["Datetime"], format="%d-%m-%Y %H:%M")
print(gp)

gp.info()


def slot(date_time):
    time = date_time.time()
    if (
        time >= pd.to_datetime("00:00:00").time()
        and time < pd.to_datetime("12:00:00").time()
    ):
        return "Slot1"
    elif (
        time > pd.to_datetime("12:00:00").time()
        and time <= pd.to_datetime("23:59:00").time()
    ):
        return "Slot2"
    else:

        "UK"


gp["Slot"] = gp["Datetime"].apply(slot)

print(gp)

wd.head()

wd["Datetime"] = pd.to_datetime(wd["Datetime"], format="%d-%m-%Y %H:%M")
print(wd)


def slot(date_time):
    time = date_time.time()
    if (
        time >= pd.to_datetime("00:00:00").time()
        and time < pd.to_datetime("12:00:00").time()
    ):
        return "Slot1"
    elif (
        time > pd.to_datetime("12:00:00").time()
        and time <= pd.to_datetime("23:59:00").time()
    ):
        return "Slot2"
    else:

        "UK"


wd["Slot"] = wd["Datetime"].apply(slot)

print(wd)

dp.head()

dp["Datetime"] = pd.to_datetime(dp["Datetime"], format="%d-%m-%Y %H:%M")
print(dp)


def slot(date_time):
    time = date_time.time()
    if (
        time >= pd.to_datetime("00:00:00").time()
        and time < pd.to_datetime("12:00:00").time()
    ):
        return "Slot1"
    elif (
        time > pd.to_datetime("12:00:00").time()
        and time <= pd.to_datetime("23:59:00").time()
    ):
        return "Slot2"
    else:

        "UK"


dp["Slot"] = dp["Datetime"].apply(slot)
```

### Based on the above information and the data provided answer the following questions:
### 1. Find Playerwise Loyalty points earned by Players in the following slots:-
    a. 2nd October Slot S1
    b. 16th October Slot S2
    b. 18th October Slot S1
    b. 26th October Slot S2

```sql
#    a. 2nd October Slot S1

WITH deposit_sum
AS (
	SELECT User_Id
		,sum(Amount) AS Total_deposit
		,Slot
	FROM deposit
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-01'
	GROUP BY User_Id
		,Slot
	)
	,withdrawal_sum
AS (
	SELECT User_id
		,sum(Amount) AS Total_withdrawal
		,Slot
	FROM withdrawal
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-01'
	GROUP BY User_Id
		,Slot
	)
	,games_played
AS (
	SELECT User_Id
		,sum(Games_Played) AS Total_games
		,Slot
	FROM gameplay
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-01'
	GROUP BY User_Id
		,Slot
	)
	,count_deposit
AS (
	SELECT User_Id
		,(count(Amount)) AS deposit_num
		,Slot
	FROM deposit
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-01'
	GROUP BY User_Id
		,Slot
	)
	,count_withdrawal
AS (
	SELECT User_Id
		,count(Amount) AS withdrawa_num
		,Slot
	FROM withdrawal
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-01'
	GROUP BY User_Id
		,Slot
	)
	,deposit_withdrawal_diff
AS (
	SELECT d.User_Id
		,d.Slot
		,max(COALESCE(d.deposit_num, 0) - COALESCE(w.withdrawa_num, 0)) AS dep_with_diff
	FROM count_deposit d
	INNER JOIN count_withdrawal w ON d.User_Id = w.User_Id
		AND d.Slot = w.Slot
	GROUP BY d.User_Id
		,d.Slot
	)
SELECT gp.User_Id
	,gp.Slot
	,((0.01 * COALESCE(ds.Total_deposit, 0)) + (0.005 * COALESCE(wd.Total_withdrawal, 0)) + (0.001 * COALESCE(dwd.dep_with_diff, 0)) + (0.2 * coalesce(gp.Total_games, 0))) AS Loyality_points
FROM games_played gp
LEFT JOIN deposit_sum ds ON gp.User_Id = ds.User_Id
	AND gp.Slot = ds.Slot
LEFT JOIN withdrawal_sum wd ON gp.User_Id = wd.User_Id
	AND gp.Slot = wd.Slot
LEFT JOIN deposit_withdrawal_diff dwd ON gp.User_Id = dwd.User_Id
	AND gp.Slot = dwd.Slot
GROUP BY gp.User_Id
	,gp.Slot
	,ds.Total_deposit
	,wd.Total_withdrawal
	,dwd.dep_with_diff
	,gp.Total_games;


#    c. 16th October Slot S2

WITH deposit_sum
AS (
	SELECT User_Id
		,sum(Amount) AS Total_deposit
		,Slot
	FROM deposit
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-16'
	GROUP BY User_Id
		,Slot
	)
	,withdrawal_sum
AS (
	SELECT User_id
		,sum(Amount) AS Total_withdrawal
		,Slot
	FROM withdrawal
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-16'
	GROUP BY User_Id
		,Slot
	)
	,games_played
AS (
	SELECT User_Id
		,sum(Games_Played) AS Total_games
		,Slot
	FROM gameplay
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-16'
	GROUP BY User_Id
		,Slot
	)
	,count_deposit
AS (
	SELECT User_Id
		,(count(Amount)) AS deposit_num
		,Slot
	FROM deposit
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-16'
	GROUP BY User_Id
		,Slot
	)
	,count_withdrawal
AS (
	SELECT User_Id
		,count(Amount) AS withdrawa_num
		,Slot
	FROM withdrawal
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-16'
	GROUP BY User_Id
		,Slot
	)
	,deposit_withdrawal_diff
AS (
	SELECT d.User_Id
		,d.Slot
		,max(COALESCE(d.deposit_num, 0) - COALESCE(w.withdrawa_num, 0)) AS dep_with_diff
	FROM count_deposit d
	INNER JOIN count_withdrawal w ON d.User_Id = w.User_Id
		AND d.Slot = w.Slot
	GROUP BY d.User_Id
		,d.Slot
	)
SELECT gp.User_Id
	,gp.Slot
	,((0.01 * COALESCE(ds.Total_deposit, 0)) + (0.005 * COALESCE(wd.Total_withdrawal, 0)) + (0.001 * COALESCE(dwd.dep_with_diff, 0)) + (0.2 * coalesce(gp.Total_games, 0))) AS Loyality_points
FROM games_played gp
LEFT JOIN deposit_sum ds ON gp.User_Id = ds.User_Id
	AND gp.Slot = ds.Slot
LEFT JOIN withdrawal_sum wd ON gp.User_Id = wd.User_Id
	AND gp.Slot = wd.Slot
LEFT JOIN deposit_withdrawal_diff dwd ON gp.User_Id = dwd.User_Id
	AND gp.Slot = dwd.Slot
GROUP BY gp.User_Id
	,gp.Slot
	,ds.Total_deposit
	,wd.Total_withdrawal
	,dwd.dep_with_diff
	,gp.Total_games;


#    b. 18th October Slot S1

WITH deposit_sum
AS (
	SELECT User_Id
		,sum(Amount) AS Total_deposit
		,Slot
	FROM deposit
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-18'
	GROUP BY User_Id
		,Slot
	)
	,withdrawal_sum
AS (
	SELECT User_id
		,sum(Amount) AS Total_withdrawal
		,Slot
	FROM withdrawal
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-18'
	GROUP BY User_Id
		,Slot
	)
	,games_played
AS (
	SELECT User_Id
		,sum(Games_Played) AS Total_games
		,Slot
	FROM gameplay
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-18'
	GROUP BY User_Id
		,Slot
	)
	,count_deposit
AS (
	SELECT User_Id
		,(count(Amount)) AS deposit_num
		,Slot
	FROM deposit
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-18'
	GROUP BY User_Id
		,Slot
	)
	,count_withdrawal
AS (
	SELECT User_Id
		,count(Amount) AS withdrawa_num
		,Slot
	FROM withdrawal
	WHERE Slot = 'Slot1'
		AND DATE (DATETIME) = '2022-10-18'
	GROUP BY User_Id
		,Slot
	)
	,deposit_withdrawal_diff
AS (
	SELECT d.User_Id
		,d.Slot
		,max(COALESCE(d.deposit_num, 0) - COALESCE(w.withdrawa_num, 0)) AS dep_with_diff
	FROM count_deposit d
	INNER JOIN count_withdrawal w ON d.User_Id = w.User_Id
		AND d.Slot = w.Slot
	GROUP BY d.User_Id
		,d.Slot
	)
SELECT gp.User_Id
	,gp.Slot
	,((0.01 * COALESCE(ds.Total_deposit, 0)) + (0.005 * COALESCE(wd.Total_withdrawal, 0)) + (0.001 * COALESCE(dwd.dep_with_diff, 0)) + (0.2 * coalesce(gp.Total_games, 0))) AS Loyality_points
FROM games_played gp
LEFT JOIN deposit_sum ds ON gp.User_Id = ds.User_Id
	AND gp.Slot = ds.Slot
LEFT JOIN withdrawal_sum wd ON gp.User_Id = wd.User_Id
	AND gp.Slot = wd.Slot
LEFT JOIN deposit_withdrawal_diff dwd ON gp.User_Id = dwd.User_Id
	AND gp.Slot = dwd.Slot
GROUP BY gp.User_Id
	,gp.Slot
	,ds.Total_deposit
	,wd.Total_withdrawal
	,dwd.dep_with_diff
	,gp.Total_games;


#    b. 26th October Slot S2

WITH deposit_sum
AS (
	SELECT User_Id
		,sum(Amount) AS Total_deposit
		,Slot
	FROM deposit
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-26'
	GROUP BY User_Id
		,Slot
	)
	,withdrawal_sum
AS (
	SELECT User_id
		,sum(Amount) AS Total_withdrawal
		,Slot
	FROM withdrawal
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-26'
	GROUP BY User_Id
		,Slot
	)
	,games_played
AS (
	SELECT User_Id
		,sum(Games_Played) AS Total_games
		,Slot
	FROM gameplay
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-26'
	GROUP BY User_Id
		,Slot
	)
	,count_deposit
AS (
	SELECT User_Id
		,(count(Amount)) AS deposit_num
		,Slot
	FROM deposit
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-26'
	GROUP BY User_Id
		,Slot
	)
	,count_withdrawal
AS (
	SELECT User_Id
		,count(Amount) AS withdrawa_num
		,Slot
	FROM withdrawal
	WHERE Slot = 'Slot2'
		AND DATE (DATETIME) = '2022-10-26'
	GROUP BY User_Id
		,Slot
	)
	,deposit_withdrawal_diff
AS (
	SELECT d.User_Id
		,d.Slot
		,max(COALESCE(d.deposit_num, 0) - COALESCE(w.withdrawa_num, 0)) AS dep_with_diff
	FROM count_deposit d
	INNER JOIN count_withdrawal w ON d.User_Id = w.User_Id
		AND d.Slot = w.Slot
	GROUP BY d.User_Id
		,d.Slot
	)
SELECT gp.User_Id
	,gp.Slot
	,((0.01 * COALESCE(ds.Total_deposit, 0)) + (0.005 * COALESCE(wd.Total_withdrawal, 0)) + (0.001 * COALESCE(dwd.dep_with_diff, 0)) + (0.2 * coalesce(gp.Total_games, 0))) AS Loyality_points
FROM games_played gp
LEFT JOIN deposit_sum ds ON gp.User_Id = ds.User_Id
	AND gp.Slot = ds.Slot
LEFT JOIN withdrawal_sum wd ON gp.User_Id = wd.User_Id
	AND gp.Slot = wd.Slot
LEFT JOIN deposit_withdrawal_diff dwd ON gp.User_Id = dwd.User_Id
	AND gp.Slot = dwd.Slot
GROUP BY gp.User_Id
	,gp.Slot
	,ds.Total_deposit
	,wd.Total_withdrawal
	,dwd.dep_with_diff
	,gp.Total_games;
```
### 2. Calculate overall loyalty points earned and rank players on the basis of loyalty points in the month of October. 
###     In case of tie, number of games played should be taken as the next criteria for ranking.

```sql
WITH deposit_sum
AS (
	SELECT User_Id
		,sum(Amount) AS Total_deposit
		,Slot
	FROM deposit
	GROUP BY User_Id
		,Slot
	)
	,withdrawal_sum
AS (
	SELECT User_id
		,sum(Amount) AS Total_withdrawal
		,Slot
	FROM withdrawal
	GROUP BY User_Id
		,Slot
	)
	,games_played
AS (
	SELECT User_Id
		,sum(Games_Played) AS Total_games
		,Slot
	FROM gameplay
	GROUP BY User_Id
		,Slot
	)
	,count_deposit
AS (
	SELECT User_Id
		,(count(Amount)) AS deposit_num
		,Slot
	FROM deposit
	GROUP BY User_Id
		,Slot
	)
	,count_withdrawal
AS (
	SELECT User_Id
		,count(Amount) AS withdrawa_num
		,Slot
	FROM withdrawal
	GROUP BY User_Id
		,Slot
	)
	,deposit_withdrawal_diff
AS (
	SELECT d.User_Id
		,d.Slot
		,max(COALESCE(d.deposit_num, 0) - COALESCE(w.withdrawa_num, 0)) AS dep_with_diff
	FROM count_deposit d
	INNER JOIN count_withdrawal w ON d.User_Id = w.User_Id
		AND d.Slot = w.Slot
	GROUP BY d.User_Id
		,d.Slot
	)
SELECT gp.User_Id
	,gp.Slot
	,((0.01 * COALESCE(ds.Total_deposit, 0)) + (0.005 * COALESCE(wd.Total_withdrawal, 0)) + (0.001 * COALESCE(dwd.dep_with_diff, 0)) + (0.2 * coalesce(gp.Total_games, 0))) AS Loyality_points
	,row_number() OVER (
		ORDER BY (0.01 * COALESCE(ds.Total_deposit, 0)) + (0.005 * COALESCE(wd.Total_withdrawal, 0)) + (0.001 * COALESCE(dwd.dep_with_diff, 0)) + (0.2 * coalesce(gp.Total_games, 0)) DESC
		) AS RK
FROM games_played gp
LEFT JOIN deposit_sum ds ON gp.User_Id = ds.User_Id
	AND gp.Slot = ds.Slot
LEFT JOIN withdrawal_sum wd ON gp.User_Id = wd.User_Id
	AND gp.Slot = wd.Slot
LEFT JOIN deposit_withdrawal_diff dwd ON gp.User_Id = dwd.User_Id
	AND gp.Slot = dwd.Slot
GROUP BY gp.User_Id
	,gp.Slot
	,ds.Total_deposit
	,wd.Total_withdrawal
	,dwd.dep_with_diff
	,gp.Total_games;
```

### 3. What is the average deposit amount?

```sql
SELECT round(avg(amount)) AS avg_deposit_amount
FROM deposit;
```

### 4. What is the average deposit amount per user in a month?

```sql
SELECT User_Id
	,month(DATETIME) AS mnth
	,round(avg(amount)) AS avg_deposit_amount
FROM deposit
GROUP BY User_Id
	,mnth;
 ```

### 5. What is the average number of games played per user?

```sql
SELECT User_Id
	,avg(Games_Played) AS games_played
FROM gameplay
GROUP BY User_Id;
```

### Part B - How much bonus should be allocated to leaderboard players?

After calculating the loyalty points for the whole month find out which 50 players are at the top of the leaderboard. The company has allocated a pool of Rs 50000 to be given away as bonus money to the loyal players.

Now the company needs to determine how much bonus money should be given to the players.

Should they base it on the amount of loyalty points? Should it be based on number of games? Or something else?

That’s for you to figure out.

Suggest a suitable way to divide the allocated money keeping in mind the following points:
1. Only top 50 ranked players are awarded bonus

```sql
WITH deposit_sum
AS (
	SELECT User_Id
		,sum(Amount) AS Total_deposit
		,Slot
	FROM deposit
	GROUP BY User_Id
		,Slot
	)
	,withdrawal_sum
AS (
	SELECT User_id
		,sum(Amount) AS Total_withdrawal
		,Slot
	FROM withdrawal
	GROUP BY User_Id
		,Slot
	)
	,games_played
AS (
	SELECT User_Id
		,Games_Played
		,sum(Games_Played) AS Total_games
		,Slot
	FROM gameplay
	GROUP BY User_Id
		,Games_Played
		,Slot
	)
	,count_deposit
AS (
	SELECT User_Id
		,(count(Amount)) AS deposit_num
		,Slot
	FROM deposit
	GROUP BY User_Id
		,Slot
	)
	,count_withdrawal
AS (
	SELECT User_Id
		,count(Amount) AS withdrawa_num
		,Slot
	FROM withdrawal
	GROUP BY User_Id
		,Slot
	)
	,deposit_withdrawal_diff
AS (
	SELECT d.User_Id
		,d.Slot
		,max(COALESCE(d.deposit_num, 0) - COALESCE(w.withdrawa_num, 0)) AS dep_with_diff
	FROM count_deposit d
	INNER JOIN count_withdrawal w ON d.User_Id = w.User_Id
		AND d.Slot = w.Slot
	GROUP BY d.User_Id
		,d.Slot
	)
SELECT gp.User_Id
	,gp.Games_Played
	,gp.Slot
	,((0.01 * COALESCE(ds.Total_deposit, 0)) + (0.005 * COALESCE(wd.Total_withdrawal, 0)) + (0.001 * COALESCE(dwd.dep_with_diff, 0)) + (0.2 * coalesce(gp.Total_games, 0))) AS Loyality_points
	,row_number() OVER (
		ORDER BY (0.01 * COALESCE(ds.Total_deposit, 0)) + (0.005 * COALESCE(wd.Total_withdrawal, 0)) + (0.001 * COALESCE(dwd.dep_with_diff, 0)) + (0.2 * coalesce(gp.Total_games, 0)) DESC
		) AS RK
FROM games_played gp
LEFT JOIN deposit_sum ds ON gp.User_Id = ds.User_Id
	AND gp.Slot = ds.Slot
LEFT JOIN withdrawal_sum wd ON gp.User_Id = wd.User_Id
	AND gp.Slot = wd.Slot
LEFT JOIN deposit_withdrawal_diff dwd ON gp.User_Id = dwd.User_Id
	AND gp.Slot = dwd.Slot
GROUP BY gp.User_Id
	,gp.Games_Played
	,gp.Slot
	,ds.Total_deposit
	,wd.Total_withdrawal
	,dwd.dep_with_diff
	,gp.Total_games limit 50;

```
### Q. Suggest a suitable way to divide the allocated money keeping in mind the following points:

Ans. The best suitable way according to me is to give reward based on rank like
     Rank (1 to 10) should be given Higher reward like 30% of money distributed equally
     Rank (11 to 30) should be given medium reward like 45% of remaining money
     Rank (31 to 50) should be given less reward like 25% of remaining money

## Part C

### Q.Would you say the loyalty point formula is fair or unfair? Can you suggest any way to make the loyalty point formula more robust?

Ans. I think loyality point formula is fair according to me more robust method can be
 if company can add a metrics which can tell how much time a user is being engaged in the app.
