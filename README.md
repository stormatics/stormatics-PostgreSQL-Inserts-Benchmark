Get the dataset from here:

wget https://timescale-insert-dataset.s3.amazonaws.com/data.zip
unzip data.zip

Each folder contains 3 files as per records:

readings.csv 
readings_singleinserts.sql 
readings_batchedinserts.sql

Now in order to populate dataset for benchmarking, run the following command:

**CSV for COPY**

psql "postgres://tsdbadmin:redacted@redacted.tsdb.cloud.timescale.com:33098/tsdb?sslmode=require"  -c '\timing' -c "\\copy readings FROM '/mnt/data/readings.csv' DELIMITER ',' CSV"

**Batched Inserts**

psql "postgres://tsdbadmin:redacted@redacted.tsdb.cloud.timescale.com:33098/tsdb?sslmode=require"  -c '\timing' -f /mnt/data/readings_batchedinserts.sql 

**Single Inserts**

psql "postgres://tsdbadmin:redacted@redacted.tsdb.cloud.timescale.com:33098/tsdb?sslmode=require"  -c '\timing' -f /mnt/data/readings_singleinserts.sql


