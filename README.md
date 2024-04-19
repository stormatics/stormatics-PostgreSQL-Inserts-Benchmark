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

Guide for 1 million rows. The dataset is defined inside **data_nested** folder.

Create Staging table and populate the dataset

`CREATE TABLE staging (
 tags_id integer[],         		 
 time timestamp with time zone[],
 latitude double precision[],		 
 longitude double precision[],		 
 elevation double precision[],		 
 velocity double precision[],		 
 heading double precision[],		 
 grade double precision[],		 
 fuel_consumption double precision[]);`

`psql "postgres://tsdbadmin:redacted@redacted.tsdb.cloud.timescale.com:33098/tsdb?sslmode=require"  -c '\timing' -f nested_1mil.sql`

Now insert into the reading tables like this:

`Insert into readings select unnest(time),
                        	 unnest(tags_id),
                        	 unnest(latitude),
                        	 unnest(longitude),
                        	 unnest(elevation),
                        	 unnest(velocity),
                        	 unnest(heading),
                        	 unnest(grade),
                        	 unnest(fuel_consumption) from staging;`
