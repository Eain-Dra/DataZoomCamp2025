# DataZoomCamp2025
docker run -it \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=postgres \
-e POSTGRES_DB="ny_taxi" \
-v "$pwd/ny_taxi_postgres_data:/var/lib/postgresql/data" \
-p 5433:5432 \
--network=pg-network \
--name db \
postgres:17-alpine

docker run -it \
-e PGADMIN_DEFAULT_EMAIL=pgadmin@pgadmin.com \
-e PGADMIN_DEFAULT_PASSWORD=pgadmin \
-p 8080:80 \
--network=pg-network \
--name pgadmin \
dpage/pgadmin4:latest

URL=https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
python pipeline.py \
--user=postgres \
--password=postgres \
--host=localhost \ 
--port=5433 \
--db=ny_taxi \
--table_name=green_taxi_trips \
--url=$URL

URL=https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
python zone.py \
--user=postgres \
--password=postgres \
--host=localhost \ 
--port=5433 \
--db=ny_taxi \
--table_name=taxi_zone \
--url=$URL

Question 3. Trip Segmentation Count
```sql
SELECT 
    COUNT(*) 
FROM 
    green_taxi_trips
WHERE 
    lpep_pickup_datetime >= '2019-10-01' 
    AND lpep_pickup_datetime < '2019-11-01'
    AND trip_distance <= 1;
```

```sql
SELECT 
    COUNT(*) 
FROM 
    green_taxi_trips
WHERE 
    lpep_pickup_datetime >= '2019-10-01' 
    AND lpep_pickup_datetime < '2019-11-01'
    AND trip_distance > 1 
    AND trip_distance <= 3;
```

```sql
SELECT 
    COUNT(*) 
FROM 
    green_taxi_trips
WHERE 
    lpep_pickup_datetime >= '2019-10-01' 
    AND lpep_pickup_datetime < '2019-11-01'
    AND trip_distance > 3 
    AND trip_distance <= 7;
```

```sql
SELECT 
    COUNT(*) 
FROM 
    green_taxi_trips
WHERE 
    lpep_pickup_datetime >= '2019-10-01' 
    AND lpep_pickup_datetime < '2019-11-01'
    AND trip_distance > 7 
    AND trip_distance <= 10;
```

```sql
SELECT 
    COUNT(*) 
FROM 
    green_taxi_trips
WHERE 
    lpep_pickup_datetime >= '2019-10-01' 
    AND lpep_pickup_datetime < '2019-11-01'
    AND trip_distance > 10
```

Question 4: Longest trip for each day
```sql
SELECT 
    MAX(trip_distance)
FROM 
    public.green_taxi_trips

SELECT 
    DATE(lpep_pickup_datetime)
FROM 
    green_taxi_trips
WHERE 
    trip_distance = 515.89
```

Question 5: Three biggest pickup zones
```sql
SELECT 
    tz."Zone" AS pickup_location, 
    SUM(gtt.total_amount) AS total_amount
FROM 
    green_taxi_trips gtt
INNER JOIN 
    taxi_zone tz 
ON 
    gtt."PULocationID" = tz."LocationID"
WHERE 
    DATE(gtt.lpep_pickup_datetime) = '2019-10-18'
GROUP BY 
    tz."Zone"
HAVING 
    SUM(gtt.total_amount) > 13000
ORDER BY 
    total_amount DESC;
```
Question 6: Largest tip
```sql
SELECT
    tz_drop."Zone" AS dropoff_zone,
    MAX(gtt.tip_amount) AS largest_tip
FROM
    green_taxi_trips gtt
INNER JOIN
    taxi_zone tz_pickup
ON
    gtt."PULocationID" = tz_pickup."LocationID"
INNER JOIN
    taxi_zone tz_drop
ON
    gtt."DOLocationID" = tz_drop."LocationID"
WHERE
    tz_drop."Zone" = 'East Harlem North'
    AND gtt.lpep_pickup_datetime LIKE '2019-10%'
GROUP BY
    tz_drop."Zone"
ORDER BY
    largest_tip DESC
LIMIT 1;
```