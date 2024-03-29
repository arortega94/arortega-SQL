[Go back to portfolio](https://arortega94.github.io/)
# Unveiling User Patterns Through Advanced SQL Analysis
## Data inspection and analyzing for a **Shared Bikes Company**
### Context
In this SQL-driven case study focused on a bike-sharing company, our primary goal is to delve deeper into the distinct user profiles of subscribers and casual customers. By analyzing the data, we aim to uncover valuable insights into user behaviors, usage patterns, and trends. The results will provide a comprehensive understanding of each user type, enabling strategic decision-making for optimizing services and tailoring marketing efforts to enhance the overall user experience.

*The full analysis will be performed using BigQuery (Google), hence some slight syntax differences compared to MS Server, Oracle or any other tool.*
### Steps
1. Inspecting the data
2. Grouping the data
3. Segregating data by usertype
4. Inspecting traffic patterns
5. Conclusions

### 1. Inspecting the data
The data to inspect in the study will be from the 1st quarter of 2019, from a shared bike company. The goal will be to understand the type of users on the app and how they "move" around the city, discovering their traffic patterns and usage trends.

First of all, let's check the first 10 rows of the file:
```sql
SELECT *
FROM `my-data-project-1-407722.cyclistic_trips.2019_Q1`
LIMIT 10

| Row | trip_id | start_time                 | end_time                   | bikeid | tripduration | from_station_id | from_station_name | to_station_id | to_station_name | usertype   | gender | birthyear | ride_length |
|-----|---------|----------------------------|----------------------------|--------|--------------|-----------------|---------------------|---------------|------------------|------------|--------|-----------|-------------|
| 1   | 21758631| 2019-01-04 12:50:24 UTC    | 2019-01-04 12:51:28 UTC    | 640    | 64.0         | 628             | Walsh Park          | 628           | Walsh Park       | Subscriber | null   | 1980      | null        |
| 2   | 21764541| 2019-01-05 11:26:35 UTC    | 2019-01-05 12:48:04 UTC    | 2328   | 4889.0       | 628             | Walsh Park          | 628           | Walsh Park       | Customer   | null   | null      | null        |
| 3   | 21767295| 2019-01-05 15:05:33 UTC    | 2019-01-05 16:17:40 UTC    | 4764   | 4327.0       | 628             | Walsh Park          | 628           | Walsh Park       | Subscriber | Female | 1983      | null        |
| 4   | 21768113| 2019-01-05 16:01:44 UTC    | 2019-01-05 16:04:04 UTC    | 5385   | 140.0        | 628             | Walsh Park          | 628           | Walsh Park       | Subscriber | Male   | 1991      | null        |
| 5   | 21768154| 2019-01-05 16:05:35 UTC    | 2019-01-05 17:05:32 UTC    | 2659   | 3597.0       | 628             | Walsh Park          | 628           | Walsh Park       | Customer   | null   | null      | null        |
| 6   | 21768162| 2019-01-05 16:06:06 UTC    | 2019-01-05 17:05:41 UTC    | 2328   | 3575.0       | 628             | Walsh Park          | 628           | Walsh Park       | Customer   | null   | null      | null        |
| 7   | 21818307| 2019-01-15 10:56:09 UTC    | 2019-01-15 10:57:25 UTC    | 451    | 76.0         | 628             | Walsh Park          | 628           | Walsh Park       | Subscriber | null   | 1980      | null        |
| 8   | 21954243| 2019-02-22 12:55:45 UTC    | 2019-02-22 13:25:49 UTC    | 688    | 1804.0       | 628             | Walsh Park          | 628           | Walsh Park       | Subscriber | Female | 1989      | null        |
| 9   | 22067010| 2019-03-16 18:10:56 UTC    | 2019-03-16 18:23:05 UTC    | 5258   | 729.0        | 628             | Walsh Park          | 628           | Walsh Park       | Customer   | null   | null      | null        |
| 10  | 22114146| 2019-03-23 10:26:37 UTC    | 2019-03-23 11:05:26 UTC    | 4255   | 2329.0       | 628             | Walsh Park          | null          | null            | null       | null   | null      | null        |
			
```
Exploring the schema, we can see all kind of data types, such as:
- Integer
- Timestamp
- FLoat
- String

## 2. Grouping the data
To inspect the data more in depth, let's group it by user type using the **GROUP BY** clause. The following query displays the usertype, average ride length in muntes, min and max ride lengths and month:
```sql
SELECT 
    usertype,
    ROUND(AVG(tripduration/60), 2) as average_ride_length_minutes,
    ROUND(MAX(tripduration/60), 2) as max_ride_length,
    ROUND(MIN(tripduration/60), 2) as min_ride_length,
    EXTRACT(MONTH FROM start_time) AS month_start_time
FROM 
    `my-data-project-1-407722.cyclistic_trips.2019_Q1`
GROUP BY 
    usertype, month_start_time
ORDER BY 
    usertype, month_start_time;

| usertype   | average_ride_length_minutes | max_ride_length | min_ride_length | month_start_time |
|------------|-----------------------------|-----------------|-----------------|-------------------|
| Customer   | 47.32                       | 29414.0         | 1.02            | 1                 |
| Customer   | 145.56                      | 177140.0        | 1.02            | 2                 |
| Customer   | 52.29                       | 56782.67        | 1.02            | 3                 |
| Subscriber | 15.65                       | 94103.0         | 1.02            | 1                 |
| Subscriber | 13.4                        | 65447.5         | 1.02            | 2                 |
| Subscriber | 13.04                       | 101607.17       | 1.02            | 3                 |
```
As we can see, the Customer or casual user spends 3 times more in average per ride than the Subscriber.

### 3. Segregating data by usertype
#### Subscribers
Let's dig into the Subscribers to understand them better. The following query displays the **month, the usertype, number of rides per month and average time per ride** of the user type "Subscriber", ordered by the number of rides in descending order:
```sql
SELECT CASE 
  WHEN extract(MONTH FROM start_time) = 1 THEN "JANUARY" 
  WHEN extract(MONTH FROM start_time) = 2 THEN "FEBRUARY"
  ELSE "MACRH"
  END AS month_start_ride,
  usertype, 
COUNT(trip_id) as rides_per_month,
ROUND(AVG(tripduration/60), 2) as avg_ride_time
FROM
`my-data-project-1-407722.cyclistic_trips.2019_Q1`
WHERE
usertype = "Subscriber"
GROUP BY
month_start_ride, usertype
ORDER BY
rides_per_month DESC

| Row | month_start_ride | usertype   | rides_per_month | avg_ride_time  |
|-----|------------------|------------|-----------------|----------------|
| 1   | MARCH            | Subscriber | 149688          | 13.04          |
| 2   | JANUARY          | Subscriber | 98670           | 15.65          |
| 3   | FEBRUARY         | Subscriber | 93548           | 13.4           |
```
**March** was the month of the first quarter with more number of rides, and interestingly with the shortest avergage ride time.
#### Customers
Let's repeat the query for the Customers, but let's use the **UNION ALL** function to display all info together. We need to be carefull with the ORDER BY and GROUP BY clauses when unifying queries:
```sql
SELECT 
  month_start_ride,
  usertype, 
  SUM(rides_per_month) AS rides_per_month,
  AVG(avg_time_month) AS avg_time_month
FROM (
  SELECT 
    CASE 
      WHEN EXTRACT(MONTH FROM start_time) = 1 THEN "JANUARY" 
      WHEN EXTRACT(MONTH FROM start_time) = 2 THEN "FEBRUARY"
      ELSE "MARCH"
    END AS month_start_ride,
    usertype, 
    COUNT(trip_id) AS rides_per_month,
    ROUND(AVG(tripduration/60), 2) AS avg_time_month
  FROM
    `my-data-project-1-407722.cyclistic_trips.2019_Q1`
  WHERE
    usertype = "Subscriber"
  GROUP BY
    month_start_ride, usertype
  
  UNION ALL
  
  SELECT 
    CASE 
      WHEN EXTRACT(MONTH FROM start_time) = 1 THEN "JANUARY" 
      WHEN EXTRACT(MONTH FROM start_time) = 2 THEN "FEBRUARY"
      ELSE "MARCH"
    END AS month_start_ride,
    usertype, 
    COUNT(trip_id) AS rides_per_month,
    ROUND(AVG(tripduration/60), 2) AS avg_time_month
  FROM
    `my-data-project-1-407722.cyclistic_trips.2019_Q1`
  WHERE
    usertype = "Customer"
  GROUP BY
    month_start_ride, usertype
) AS combined_data
GROUP BY
    month_start_ride, usertype
ORDER BY
    rides_per_month DESC;

| Row | month_start_ride | usertype   | rides_per_month | avg_time_month |
|-----|------------------|------------|-----------------|----------------|
| 1   | MARCH            | Subscriber | 149688          | 13.04          |
| 2   | JANUARY          | Subscriber | 98670           | 15.65          |
| 3   | FEBRUARY         | Subscriber | 93548           | 13.4           |
| 4   | MARCH            | Customer   | 15923           | 52.29          |
| 5   | JANUARY          | Customer   | 4602            | 47.32          |
| 6   | FEBRUARY         | Customer   | 2638            | 145.56         |
```
We can check how differently both user types are leveraging the services. Customers take **longer rides** while Subscribers **take more rides**.

Let's check now the user type distribution using the window function **OVER()**:
```sql
SELECT
  usertype AS `User type`,
  COUNT(trip_id) AS `Number of trips`,
  ROUND((COUNT(trip_id) / SUM(COUNT(trip_id)) OVER ()) * 100, 2) AS `Percentage`
FROM
`my-data-project-1-407722.cyclistic_trips.2019_Q1`
GROUP BY
usertype;

| Row | User type   | Number of trips | Percentage |
|-----|-------------|------------------|------------|
| 1   | Subscriber  | 341,906          | 93.66      |
| 2   | Customer    | 23,163           | 6.34       |
```
The subscribers represent the **93.66%** of the rides.
### 4. Inspecting traffic patterns
Now let's inspect the different **stations** from where the users start or end up a ride, since this is interesting to know for distribution and stocking purposes. The following query displays the **top 10 stations more popular to start a ride**:
```sql
SELECT
  from_station_name AS `Station of origin`,
  from_station_id AS `Id of station`,
  COUNT(from_station_id) AS `Rides started from there`
FROM 
  `my-data-project-1-407722.cyclistic_trips.2019_Q1`
GROUP BY
  from_station_id, from_station_name
ORDER BY
  COUNT(from_station_id) DESC
LIMIT 10;

| Row | Station of origin                | Id of station | Rides started from there |
|-----|----------------------------------|---------------|---------------------------|
| 1   | Clinton St & Washington Blvd     | 91            | 7,699                     |
| 2   | Clinton St & Madison St          | 77            | 6,565                     |
| 3   | Canal St & Adams St              | 192           | 6,342                     |
| 4   | Columbus Dr & Randolph St        | 195           | 4,655                     |
| 5   | Canal St & Madison St            | 174           | 4,571                     |
| 6   | Kingsbury St & Kinzie St         | 133           | 4,395                     |
| 7   | Michigan Ave & Washington St     | 43            | 3,992                     |
| 8   | Franklin St & Monroe St          | 287           | 3,516                     |
| 9   | LaSalle St & Jackson Blvd        | 283           | 3,252                     |
| 10  | Dearborn St & Monroe St          | 49            | 3,246                     |
```
As we can see, **Clinton St & Washington Blvd** is from where users usually start their journeys. Let's check where user normally **end** their rides:
```sql
SELECT
  to_station_name AS `Station of end`,
  to_station_id AS `Id of station`,
  COUNT(to_station_id) AS `Rides finished here`
FROM 
  `my-data-project-1-407722.cyclistic_trips.2019_Q1`
GROUP BY
  to_station_id, to_station_name
ORDER BY
  COUNT(to_station_id) DESC
LIMIT 10;

| Row | Station of end                   | Id of station | Rides finished here       |
|-----|----------------------------------|---------------|---------------------------|
| 1   | Clinton St & Washington Blvd     | 91            | 7,699                     |
| 2   | Clinton St & Madison St          | 77            | 6,859                     |
| 3   | Canal St & Adams St              | 192           | 6,744                     |
| 4   | Canal St & Madison St            | 174           | 4,875                     |
| 5   | Michigan Ave & Washington St     | 43            | 4,412                     |
| 6   | Kingsbury St & Kinzie St         | 133           | 4,376                     |
| 7   | LaSalle St & Jackson Blvd        | 283           | 3,304                     |
| 8   | Clinton St & Lake St             | 66            | 3,297                     |
| 9   | Dearborn St & Monroe St          | 49            | 3,137                     |
```
Similarly, it's also **Clinton St & Washington Blvd** with 7.699 rides finished there.

Let's finally check the numbers of trips started and ended at the same station grouped by station:
```sql
SELECT 
  from_station_name as `Station name`,
  count(trip_id) AS `Round trips`
FROM 
  `my-data-project-1-407722.cyclistic_trips.2019_Q1`
WHERE 
  from_station_id = to_station_id
group by 
  from_station_name
ORDER BY 
  count(trip_id) DESC
LIMIT 10

| Row | Station name                          | Round trips |
|-----|---------------------------------------|-------------|
| 1   | Lake Shore Dr & Monroe St             | 302         |
| 2   | Streeter Dr & Grand Ave               | 230         |
| 3   | Millennium Park                       | 116         |
| 4   | Michigan Ave & Oak St                 | 99          |
| 5   | Central Park Ave & Bloomingdale Ave   | 78          |
| 6   | Wabash Ave & Grand Ave                | 75          |
| 7   | Shedd Aquarium                        | 74          |
| 8   | Columbus Dr & Randolph St             | 65          |
| 9   | Indiana Ave & Roosevelt Rd            | 60          |
| 10  | Montrose Harbor                       | 58          |
```
Surprisingly, it's not Clinton St & Washington Blvd, but **Lake Shore Dr & Monroe St** with 302 round trips.
### 5. Conclusions
In conclusion, we obtained the following insights from the inspection:
- The Customer user spends **3 times** more in average per ride than the Subscriber
- **March** was the month of the first quarter with more number of rides and with the shortest avergage ride time
- Customers take **longer rides** while Subscribers **take more rides**
- Subscribers represent the **93.66%** of the rides
- **Clinton St & Washington Blvd** is the most popular station to **start and end a ride**
- **Lake Shore Dr & Monroe St** is the most popular station to do round trips

[Go back to portfolio](https://arortega94.github.io/)
