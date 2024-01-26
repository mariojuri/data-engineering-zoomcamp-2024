# Week 01 - Docker and SQL

## Commands:


Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash. Using  
```bash
winpty docker run -it --entrypoint=bash python:3.9
```

Run docker with postgres
```bash
winpty docker run -it \
-e POSTGRES_USER="root" \
-e POSTGRES_PASSWORD="root" \
-e POSTGRES_DB="ny_taxi" \
-v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
-p 5437:5432 \
 --network=pg-network \
 --name pg-database \
postgres:13
```

### Data ingestion

Build image
```bash
docker build -t taxi_ingest:v001 .
```

Run the script with Docker
```bash
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz"

winpty docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database2 \
    --port=5432 \
    --db=ny_taxi \
    --table_name=green_taxi_trips \
    --url=${URL}
```

Question 3. How many taxi trips were totally made on September 18th 2019?
```sql
	Select count(*) 
	from green_taxi_trips t
		where DATE(t.lpep_pickup_datetime)= '2019-09-18'
		and DATE(t.lpep_dropoff_datetime)= '2019-09-18'
```

Question 4. Largest trip for each day

```sql
select
	Date(lpep_pickup_datetime),
	sum(trip_distance)
from green_taxi_trips t
group by Date(lpep_pickup_datetime)
order by 2 desc
```




Run the script with Docker to import taxi zones
```bash
URL="https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv"

winpty docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database2 \
    --port=5432 \
    --db=ny_taxi \
    --table_name=taxi_zones \
    --url=${URL}
```
Question 5. Three biggest pick up Boroughs
```sql
select
	tz."Borough" ,
	sum(total_amount)
from green_taxi_trips gtt
	join taxi_zones tz
		on gtt."PULocationID" = tz."LocationID" 
where Date(lpep_pickup_datetime) = '2019-09-18'
group by tz."Borough" 
order by 2 desc

```

Question 6. Largest tip
```sql

select
	doz."Zone" ,
	tip_amount
from green_taxi_trips gtt
	join taxi_zones puz
		on gtt."PULocationID" = puz."LocationID" 
	join taxi_zones doz
		on gtt."DOLocationID" = doz."LocationID" 
where Date(lpep_pickup_datetime) between '2019-09-01' and '2019-09-30'
	and puz."Zone" = 'Astoria'
order by 2 desc
```