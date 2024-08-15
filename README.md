Get the dataset from here:

`wget https://timescale-insert-dataset.s3.amazonaws.com/data.zip`

`unzip data.zip`

Each folder contains 3 files as per records:

**readings.csv**

**readings_singleinserts.sql** 

**readings_batchedinserts.sql**

Now in order to populate dataset for benchmarking, run the following command:

**CSV for COPY**

`psql "postgres://tsdbadmin:redacted@redacted.tsdb.cloud.timescale.com:33098/tsdb?sslmode=require"  -c '\timing' -c "\\copy readings FROM '/mnt/data/readings.csv' DELIMITER ',' CSV"`

**Batched Inserts**

`psql "postgres://tsdbadmin:redacted@redacted.tsdb.cloud.timescale.com:33098/tsdb?sslmode=require"  -c '\timing' -f /mnt/data/readings_batchedinserts.sql `

**Single Inserts**

`psql "postgres://tsdbadmin:redacted@redacted.tsdb.cloud.timescale.com:33098/tsdb?sslmode=require"  -c '\timing' -f /mnt/data/readings_singleinserts.sql`

**Nested Inserts**

Guide for 1 million rows. 

**Generate Dataset**

`tsbs_generate_data --use-case="iot" --seed=123 --scale=150 --timestamp-start="2024-01-01T00:00:00Z" --timestamp-end="2024-01-01T00:00:00Z" --log-interval="10s" --format="timescaledb" > data1m`

**Load dataset**

`tsbs_load_timescaledb --host=localhost --port=<port> --db-name=<dbname> --user=<dbuser> --do-create-db=false --use-hypertable=false --workers=4 --field-index-count=0 --time-index=false --batch-size=20000 --partition-index=false --file=data1m --pass=<dbpassword>`

`ALTER TABLE readings DROP COLUMN additional_tags;`

**Convert data to colunm arrays**

`CREATE TABLE aggregated_readings (
time TIMESTAMP WITH TIME ZONE[],
tags_id INTEGER[],
latitude DOUBLE PRECISION[],
longitude DOUBLE PRECISION[],
elevation DOUBLE PRECISION[],
velocity DOUBLE PRECISION[],
heading DOUBLE PRECISION[],
grade DOUBLE PRECISION[],
fuel_consumption DOUBLE PRECISION[]
);`

`WITH numbered_rows AS (
    SELECT
        *,
        row_number() OVER (ORDER BY time) AS rn
    FROM
        readings
),
grouped_rows AS (
    SELECT
        CEIL(rn / 50.0) AS batch,
        array_agg(time) AS time,
        array_agg(tags_id) AS tags_id,
        array_agg(latitude) AS latitude,
        array_agg(longitude) AS longitude,
        array_agg(elevation) AS elevation,
        array_agg(velocity) AS velocity,
        array_agg(heading) AS heading,
        array_agg(grade) AS grade,
        array_agg(fuel_consumption) AS fuel_consumption
    FROM
        numbered_rows
    GROUP BY
        batch
)
INSERT INTO aggregated_readings (
    time,
    tags_id,
    latitude,
    longitude,
    elevation,
    velocity,
    heading,
    grade,
    fuel_consumption
)
SELECT
    Time,
    tags_id,
    latitude,
    longitude,
    elevation,
    velocity,
    heading,
    grade,
    fuel_consumption
FROM
    Grouped_rows;
`

**Generate CSV for column arrays**

`COPY aggregated_readings TO '/tmp/readings_aggregated.csv' WITH (FORMAT CSV, HEADER);`

**Create extension**

`CREATE EXTENSION file_fdw;`

`CREATE FOREIGN TABLE my_file_table (
  time timestamp with time zone[],
  tags_id integer[],
  latitude double precision[],
  longitude double precision[],
  elevation double precision[],
  velocity double precision[],
  heading double precision[],
  grade double precision[],
  fuel_consumption double precision[]
) SERVER file_server
OPTIONS (
  filename '/tmp/readings_aggregated.csv', 
  format 'csv', 
  header 'true'
);`

`CREATE TABLE normalized_readings (        time timestamp with time zone,    tags_id integer, latitude double precision,    longitude double precision,    elevation double precision,    velocity double precision,    heading double precision,    grade double precision,    fuel_consumption double precision);`

`\timing`

`INSERT INTO normalized_readings (time, tags_id,  latitude, longitude, elevation, velocity, heading, grade, fuel_consumption)
SELECT
    unnest(time) AS time,
    unnest(tags_id) AS tags_id,
    unnest(latitude) AS latitude,
    unnest(longitude) AS longitude,
    unnest(elevation) AS elevation,
    unnest(velocity) AS velocity,
    unnest(heading) AS heading,
    unnest(grade) AS grade,
    unnest(fuel_consumption) AS fuel_consumption
FROM 
    My_file_table;`
