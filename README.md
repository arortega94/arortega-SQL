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
### 4. Inspecting traffic patterns
### 5. Conclussions
[Go back to portfolio](https://arortega94.github.io/)
