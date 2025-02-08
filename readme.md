
# DEZ_w3_homework

## w3-Data warehouse

This project utilizes Google Cloud CLI and BigQuery to build an ETL pipeline for processing NYC taxi data from 2024-01 to 2024-06.

## Step1 : Install Google Cloud CLI in Ubuntu:

1. Update apt packages

 ```bash
 sudo apt-get update
 ```

2. Import the Google Cloud public key.

 ```bash
 curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg
 ```

3. Add the gcloud CLI distribution URI as a package source.

 ```bash
 echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
 ```

4. Update and install the gcloud CLI

 ```bash
 $ sudo apt-get update && sudo apt-get install google-cloud-cli
 ```

5. Run gcloud init to get started:

 ```bash
 $ gcloud init
 ```

6. Using gcloud interactive shell

 ```bash
 $ gcloud beta interactive
 ```

## Step2: Download NY_taxi 2024-01 to 2024-06 dataset from website and upload to Google Cloud Storage
 
 ```bash
 $ python3 load_to_gcs.py
 ```

## Step3: Create BigQuery Dataset and external table. Note: Data in GCS must be in the same region with dataset region.(Biglake table can process cross region query)

    ```bash
    $ bq mk --dataset --location=australia-southeast1 dez-jimmyh:dez_hw3_tripdata
    ```

    ```bash
    $ bq mk --table --external_table_definition=/home/vice/DEZ/week3/hw3_ext_schema_def.json --location=australia-
    southeast1 dez_hw3_tripdata.yellow_tripdata_ext
    ```

## Step4: Create Main table in BigQuery

    ```bash
    $ bq query --use_legacy_sql=false --location=australia-southeast1 \
    'CREATE OR REPLACE TABLE dez-jimmyh.dez_hw3_tripdata.yellow_trip_main AS
    SELECT 
        MD5(CONCAT(
                COALESCE(CAST(VendorID AS STRING), ""),
                COALESCE(CAST(tpep_pickup_datetime AS STRING), ""),
                COALESCE(CAST(tpep_dropoff_datetime AS STRING), ""),
                COALESCE(CAST(PULocationID AS STRING), ""),
                COALESCE(CAST(DOLocationID AS STRING), "")
                )) AS unique_row_id,
        "yellow_trip_2024_01" AS filename, 
        * 
    FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_2024-01_ext`
    UNION ALL
    SELECT 
        MD5(CONCAT(
                COALESCE(CAST(VendorID AS STRING), ""),
                COALESCE(CAST(tpep_pickup_datetime AS STRING), ""),
                COALESCE(CAST(tpep_dropoff_datetime AS STRING), ""),
                COALESCE(CAST(PULocationID AS STRING), ""),
                COALESCE(CAST(DOLocationID AS STRING), "")
                )) AS unique_row_id,
        "yellow_trip_2024_02" AS filename, 
        * 
    FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_2024-02_ext`
    UNION ALL
    SELECT 
        MD5(CONCAT(
                COALESCE(CAST(VendorID AS STRING), ""),
                COALESCE(CAST(tpep_pickup_datetime AS STRING), ""),
                COALESCE(CAST(tpep_dropoff_datetime AS STRING), ""),
                COALESCE(CAST(PULocationID AS STRING), ""),
                COALESCE(CAST(DOLocationID AS STRING), "")
                )) AS unique_row_id,
        "yellow_trip_2024_03" AS filename, 
        * 
    FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_2024-03_ext`
    UNION ALL
    SELECT 
        MD5(CONCAT(
                COALESCE(CAST(VendorID AS STRING), ""),
                COALESCE(CAST(tpep_pickup_datetime AS STRING), ""),
                COALESCE(CAST(tpep_dropoff_datetime AS STRING), ""),
                COALESCE(CAST(PULocationID AS STRING), ""),
                COALESCE(CAST(DOLocationID AS STRING), "")
                )) AS unique_row_id,
        "yellow_trip_2024_04" AS filename, 
        * 
    FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_2024-04_ext`
    UNION ALL
    SELECT 
        MD5(CONCAT(
                COALESCE(CAST(VendorID AS STRING), ""),
                COALESCE(CAST(tpep_pickup_datetime AS STRING), ""),
                COALESCE(CAST(tpep_dropoff_datetime AS STRING), ""),
                COALESCE(CAST(PULocationID AS STRING), ""),
                COALESCE(CAST(DOLocationID AS STRING), "")
                )) AS unique_row_id,
        "yellow_trip_2024_05" AS filename, 
        * 
    FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_2024-05_ext`
    UNION ALL
    SELECT 
        MD5(CONCAT(
                COALESCE(CAST(VendorID AS STRING), ""),
                COALESCE(CAST(tpep_pickup_datetime AS STRING), ""),
                COALESCE(CAST(tpep_dropoff_datetime AS STRING), ""),
                COALESCE(CAST(PULocationID AS STRING), ""),
                COALESCE(CAST(DOLocationID AS STRING), "")
                )) AS unique_row_id,
        "yellow_trip_2024_06" AS filename, 
        * 
    FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_2024-06_ext`;'
    ```

## HW3-Quizs

Q1: What is count of records for the 2024 Yellow Taxi Data?

**Ans: 20332093**

```bash
$ bq query --use_legacy_sql=false 'SELECT COUNT(unique_row_id) FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_m
ain`'
+----------+
|   f0_    |
+----------+
| 20332093 |
+----------+
```

Q2: Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?

**Ans: 0 MB for the External Table and 155.12 MB for the Materialized Table**

```bash
$ bq query --dry_run --use_legacy_sql=false '
SELECT COUNT(DISTINCT PULocationID) FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_main`' 
# Query successfully validated. Assuming the tables are not modified, running this query will process 162656744 bytes of data.

$ bq query --dry_run --use_legacy_sql=false '
SELECT COUNT(DISTINCT PULocationID) FROM `dez-jimmyh.dez_hw3_tripdata.yellow_tripdata_ext`' 
# Query successfully validated. Assuming the tables are not modified, running this query will process lower bound of 0 bytes of data.
```

Q3: Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?

**Ans: BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.**

```bash
$ bq query --dry_run --use_legacy_sql=false '
  SELECT PULocationID FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_main`'
# Query successfully validated. Assuming the tables are not modified, running this query will process 162656744 bytes of data.
$ bq query --dry_run --use_legacy_sql=false '
  SELECT PULocationID,DOLocationID FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_main`'
# Query successfully validated. Assuming the tables are not modified, running this query will process 325313488 bytes of data.
```

Q4: How many records have a fare_amount of 0?

**Ans: 8333**

```bash
$ bq query --use_legacy_sql=false 'SELECT COUNT(unique_row_id) FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_m
ain` WHERE fare_amount=0'
+------+
| f0_  |
+------+
| 8333 |
+------+
```

Q5: What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)

**Ans: Partition by tpep_dropoff_datetime and Cluster on VendorID**

```bash
$ bq query --use_legacy_sql=false --location=australia-southeast1 \
'CREATE TABLE `dez-jimmyh.dez_hw3_tripdata.yellow_trip_partition_cluster`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_main`;'
```

Q6: Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive)
Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?

**Ans: 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table**

```bash
$ bq query --dry_run --use_legacy_sql=false 'SELECT COUNT(DISTINCT VendorID) FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_main` WHERE tpep_dropoff_datetime BETWEEN "2024-03-01" and "2024-03-15";'
# Query successfully validated. Assuming the tables are not modified, running this query will process 325313488 bytes of data.

$ bq query --dry_run --use_legacy_sql=false 'SELECT COUNT(DISTINCT VendorID) FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_partition_cluster` WHERE tpep_dropoff_datetime BETWEEN "2024-03-01" and "2024-03-15";'
# Query successfully validated. Assuming the tables are not modified, running this query will process upper bound of 28141776 bytes of data.
```

Q7: Where is the data stored in the External Table you created?

**Ans: GCP Bucket**

Q8: It is best practice in Big Query to always cluster your data:

**Ans: False**
It depends on your query needs and the characteristics of your data. Clustering is a way of sorting data based on certain columns, which can help accelerate the performance of specific types of queries, especially when queries frequently filter or sort based on clustering columns. If queries often use certain columns for filtering or sorting, BigQuery can locate the required data faster, reducing the amount of data that needs to be scanned. This is especially effective for large datasets, particularly when the clustering columns align with the filter conditions. Clustering is highly effective for filtering data based on range queries, such as BETWEEN, >, or <.

However, when data is frequently inserted or updated, using clustering might increase the cost of write operations, as BigQuery needs to sort the data based on the clustering keys every time new data is inserted. Additionally, too many clustering columns can degrade performance. BigQuery will need to sort data based on these columns every time a query runs, which may reduce query efficiency, especially when the clustering columns don't match the filter conditions in the query. Furthermore, if the query range is very small, or if the query doesn't frequently filter based on specific columns, clustering might not provide significant performance benefits and could instead incur additional storage and write costs.

Q9: Write a SELECT count(*) query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

**Ans: 0 Bytes. This query is getting answered from metadata tables, hence no cost. If we use some filter condition or use some actual column in the group by, it will incur cost.**

```bash
$ bq query --dry_run --use_legacy_sql=false 'SELECT COUNT(*) FROM `dez-jimmyh.dez_hw3_tripdata.yellow_trip_main`'
# Query successfully validated. Assuming the tables are not modified, running this query will process 0 bytes of data.
```
