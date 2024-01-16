[Go back to portfolio](https://arortega94.github.io/)
# Unveiling User Patterns Through Advanced SQL Analysis
## Data inspection and analyzing for a **Shared Bikes Company**
### Context
In this SQL-driven case study focused on a bike-sharing company, our primary goal is to delve deeper into the distinct user profiles of subscribers and casual customers. By analyzing the data, we aim to uncover valuable insights into user behaviors, usage patterns, and trends. The results will provide a comprehensive understanding of each user type, enabling strategic decision-making for optimizing services and tailoring marketing efforts to enhance the overall user experience.

The full analysis will be performed using BigQuery (Google), hence some slight syntax differences compared to MS Server, Oracle or any other tool.
### Steps
1. Inspecting the data
2. Grouping the data
3. Segregating info by usertype
4. Inspecting traffic patterns
5. Conclussions

### 1. Inspecting the data
The data to inspect in the study will be from the 1st quarter of 2019, from a shared bike company. The goal will be to understand the type of users on the app and how they "move" around the city, discovering their traffic patterns and usage trends.
First of all, let's check the first 10 rows of the file:
```sql
SELECT *
FROM `my-data-project-1-407722.cyclistic_trips.2019_Q1`
LIMIT 10

trip_id	start_time	end_time	bikeid	tripduration	from_station_id	from_station_name	to_station_id	to_station_name	usertype	gender	birthyear	ride_length
21758631	2019-01-04 12:50:24.000000 UTC	2019-01-04 12:51:28.000000 UTC	640	64.0	628	Walsh Park	628	Walsh Park	Subscriber		1980	
21764541	2019-01-05 11:26:35.000000 UTC	2019-01-05 12:48:04.000000 UTC	2328	4889.0	628	Walsh Park	628	Walsh Park	Customer			
21767295	2019-01-05 15:05:33.000000 UTC	2019-01-05 16:17:40.000000 UTC	4764	4327.0	628	Walsh Park	628	Walsh Park	Subscriber	Female	1983	
21768113	2019-01-05 16:01:44.000000 UTC	2019-01-05 16:04:04.000000 UTC	5385	140.0	628	Walsh Park	628	Walsh Park	Subscriber	Male	1991	
21768154	2019-01-05 16:05:35.000000 UTC	2019-01-05 17:05:32.000000 UTC	2659	3597.0	628	Walsh Park	628	Walsh Park	Customer			
21768162	2019-01-05 16:06:06.000000 UTC	2019-01-05 17:05:41.000000 UTC	2328	3575.0	628	Walsh Park	628	Walsh Park	Customer			
21818307	2019-01-15 10:56:09.000000 UTC	2019-01-15 10:57:25.000000 UTC	451	76.0	628	Walsh Park	628	Walsh Park	Subscriber		1980	
21954243	2019-02-22 12:55:45.000000 UTC	2019-02-22 13:25:49.000000 UTC	688	1804.0	628	Walsh Park	628	Walsh Park	Subscriber	Female	1989	
22067010	2019-03-16 18:10:56.000000 UTC	2019-03-16 18:23:05.000000 UTC	5258	729.0	628	Walsh Park	628	Walsh Park	Customer			
22114146	2019-03-23 10:26:37.000000 UTC	2019-03-23 11:05:26.000000 UTC	4255	2329.0	628	Walsh Park	628	Walsh Park	Customer			
```
Exploring the schema, we can see all kind of data types, such as:
- Integer
- Timestamp
- FLoat
- String

## 2. Grouping the data
To inspect the data more in depth, let's group it by user type using the Group By clause. The following query displays the usertype, average ride length in muntes, min and max ride lengths and month:
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
As we can see, the customer or casual user spends 3 times more in average per ride than the Subscriber.

### 3. Segregating info by usertype
#### Subscribers
Let's dig into the Subscribers to understand them better. The following query t displays the month, the usertype, number of rides per month and average time per ride of the user type "Subscriber", ordered by the number of rides in descending order:
```sql
SELECT CASE 
  WHEN extract(MONTH FROM start_time) = 1 THEN "JANUARY" 
  WHEN extract(MONTH FROM start_time) = 2 THEN "FEBRUARY"
  ELSE "MACRH"
  END AS month_start_ride,
  usertype, 
COUNT(trip_id) as rides_per_month,
ROUND(AVG(tripduration/60), 2) avg_time_month
FROM `my-data-project-1-407722.cyclistic_trips.2019_Q1`
WHERE usertype = "Subscriber"
group by month_start_ride, usertype
order by rides_per_month DESC

| Row | month_start_ride | usertype   | rides_per_month | avg_time_month |
|-----|------------------|------------|-----------------|----------------|
| 1   | MARCH            | Subscriber | 149688          | 13.04          |
| 2   | JANUARY          | Subscriber | 98670           | 15.65          |
| 3   | FEBRUARY         | Subscriber | 93548           | 13.4           |
```
As we can see, March was the month of the first quarter with more number of rides, and interestingly with the shortest avergage ride time.
#### Customers
Let's repeat the query for the Customers, but let's use the UNION ALL function to display all info together. We need to be carefull with the ORDER BY and GROUP BY clauses when unifying queries:
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
  FROM `my-data-project-1-407722.cyclistic_trips.2019_Q1`
  WHERE usertype = "Subscriber"
  GROUP BY month_start_ride, usertype
  
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
  FROM `my-data-project-1-407722.cyclistic_trips.2019_Q1`
  WHERE usertype = "Customer"
  GROUP BY month_start_ride, usertype
) AS combined_data
GROUP BY month_start_ride, usertype
ORDER BY rides_per_month DESC;

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

Let's check now the user type distribution using the window function OVER():
```sql
SELECT
  usertype AS `User type`,
  COUNT(trip_id) AS `Number of trips`,
  ROUND((COUNT(trip_id) / SUM(COUNT(trip_id)) OVER ()) * 100, 2) AS `Percentage`
FROM `my-data-project-1-407722.cyclistic_trips.2019_Q1`
GROUP BY usertype;

| Row | User type   | Number of trips | Percentage |
|-----|-------------|------------------|------------|
| 1   | Subscriber  | 341,906          | 93.66      |
| 2   | Customer    | 23,163           | 6.34       |
```
The subscribers represent the 93.66% of the rides.
### 4. Inspecting traffic patterns
Now let's inspect the different stations from where the users start or end up a ride, since this is interesting to know for distribution and stocking purposes. The following query displays the top 10 stations more popular to start a ride:
```sql
SELECT
  from_station_name, 
  from_station_id,
  COUNT(from_station_id) AS rides_started_from_station
FROM 
  `my-data-project-1-407722.cyclistic_trips.2019_Q1`
GROUP BY
  from_station_id, from_station_name
ORDER BY
  COUNT(from_station_id) DESC
LIMIT 10

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
As we can see, Clinton St & Washington Blvd is from where users usually start their journeys.
### 5. Conclussions
[Go back to portfolio](https://arortega94.github.io/)
