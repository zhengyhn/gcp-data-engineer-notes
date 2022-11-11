# gcp-data-engineer-notes
My notes for the preparation of GCP data engineer cert

## Transfer appliance
- Request a device , google will send it to your place
- Connect to this device and move data inside, encrypt, compressed
- Google receive and perform data integrity check, then upload to GCS

## Transfer service
- Move data from other cloud provider to GCS
- Transferring from another cloud storage provider Use Storage Transfer Service.
- Transferring less than 1 TB from on-premises Use gsutil.
- Transferring more than 1 TB from on-premises Use Transfer service for on-premises data.
- Transferring less than 1 TB from another Cloud Storage region Use gsutil.
- Transferring more than 1 TB from another Cloud Storage region Use Storage Transfer Service.
## GCS
- Classes
    - Multi-Regional: highly available
    - Regional: mainly for data of compute engine, cheaper than BigQuery storage
    - Nearline: access once a month max, for data backup
    - Coldline: access once a year max, stored for legal or regulatory reason

## Cloud SQL
- Google managed relational DB
- can store up to 30 TB of data.
- cheaper than Spanner
- can automatically scale up for storeage, but not horizontal scaling

## Cloud Spanner
- Horizontal scaling, highly available, strong consistent
- is used to store more than 30 TB of data
- expensive
- With interleaving, Spanner physically co-locates child rows with parent rows in storage. Co-location can significantly improve performance
- if you insert records with a monotonically increasing integer as the key, you'll always insert at the end of your key space. This is undesirable because Spanner divides data among servers by key ranges, which means your inserts will be directed at a single server, creating a hotspot

## Datastore
- Like mongodb
- Cloud firestore, new version of Datastore
- Datastore is perfect for semi-structured data less than 1TB in size. Product catalogs are a recommended use case

## BigTable
- Like HBase
- Expensive
- When you create a Cloud Bigtable instance and cluster, your choice of SSD or HDD storage for the cluster is permanent
- Since you cannot change the disk type on an existing Bigtable instance, you will need to export/import your Bigtable data into a new instance with the different storage type. You will need to export to Cloud Storage then back to Bigtable again.
- You can configure access control only at the project level.
- storing very large amounts of single-keyed data with very high latency
- recommended minimum amount of stored data is 1TB
- A Cloud Bigtable table is sharded into blocks of contiguous rows, called tablets, to help balance the workload of queries. Tablets are stored on Colossus, Google's file system, in SSTable format. Each tablet is associated with a specific Cloud Bigtable node.
- Field promotion involves moving fields from the column data into the row key to make writes non-contiguous. For example, TIMESLIDERS could promote the USERID from a column to an element of the row key. This change would solve the hotspotting issue because user identifiers will provide a more uniform distribution of row keys
- Make sure you're reading and writing many different rows in your table. Bigtable performs best when reads and writes are evenly distributed throughout your table, which helps Bigtable distribute the workload across all of the nodes in your cluster. If reads and writes cannot be spread across all of your Bigtable nodes, performance will suffer. If you find that you're reading and writing only a small number of rows, you might need to redesign your schema so that reads and writes are more evenly distributed.
- When you use a single cluster to run a batch analytics job that performs numerous large reads alongside an application that performs a mix of reads and writes, the large batch job can slow things down for the application's users. With replication, you can use app profiles with single-cluster routing to route batch analytics jobs and application traffic to different clusters, so that batch jobs don't affect your applications' users

## BigQuery
- Like Hive
- Use the __PARTITIONTIME pseudo-column in the WHERE clause to query specific partitions in a BigQuery table
- Materialized views are suited to small datasets that are frequently queried. When underlying table data changes, the materialized view invalidates the affected portions and re-reads them
- Once table is created, you cannot change its partitioned
- Authorized views are used to provided restricted access. An authorized view lets you share query results with particular users and groups without giving them access to the underlying source data. You can also use the view's SQL query to restrict the columns (fields) the users are able to query
- Wildcards and comma-separated lists are not supported when you load files from a local data source. Files must be loaded individually.
- When using the Cloud Console, files loaded from a local data source cannot exceed 10 MB. For larger files, load the file from Cloud Storage
- Query language is not set at dataset level. It depends on from where we are trying to access Bigquery tables. Webs default is leagacy whereas Cloud console is Standard.
- A partitioned table is a special table that is divided into segments, called partitions, that make it easier to manage and query your data. By dividing a large table into smaller partitions, you can improve query performance, and you can control costs by reducing the number of bytes read by a query
- Giving a view access to a dataset is also known as creating an authorized view in BigQuery. An authorized view lets you share query results with particular users and groups without giving them access to the underlying tables. You can also use the view's SQL query to restrict the columns (fields) the users are able to query. In this tutorial, you create an authorized view.
- TABLE_DATE_RANGE: can select table name containing date
```
SELECT
  user_dim.geo_info.city,
  COUNT(user_dim.geo_info.city) as city_count 
FROM
TABLE_DATE_RANGE([firebase-analytics-sample-data:android_dataset.app_events_], DATE_ADD('2016-06-07', -7, 'DAY'), CURRENT_TIMESTAMP()),
TABLE_DATE_RANGE([firebase-analytics-sample-data:ios_dataset.app_events_], DATE_ADD('2016-06-07', -7, 'DAY'), CURRENT_TIMESTAMP())
GROUP BY
  user_dim.geo_info.city
ORDER BY
  city_count DESC
```
- Avro is an open source data format that bundles serialized data with the data's schema in the same file.
- Partition skew, sometimes called data skew, is when data is partitioned into very unequally sized partitions. This creates an imbalance in the amount of data sent between slots. You can't share partitions between slots, so if one partition is especially large, it can slow down, or even crash the slot that processes the oversized partition.
- Even since BigQuery Data Transfer Service initially supports Google application sources like Google Ads, Campaign Manager, Google Ad Manager and YouTube but it does not support destination anything other than bq data set
- aggregate functions: output result for a group, use Group by
- window functions: output result for each row, use PARTITION BY
    - LAG: return the value of xxxx on a preceding row
    - ROW_NUMBER: return the row number of a partition
- wildcard table, backticket ``
```
SELECT
  max
FROM
  `bigquery-public-data.noaa_gsod.gsod*`
WHERE
  max != 9999.9 # code for missing data
  AND _TABLE_SUFFIX = '1929'
ORDER BY
  max DESC
```
- When you export data from BigQuery, note the following:
    - You cannot export table data to a local file, to Google Sheets, or to Google Drive. The only supported export location is Cloud Storage. 
    - You can export up to 1 GB of table data to a single file. If you are exporting more than 1 GB of data, use a wildcard to export the data into multiple files. When you export data to multiple files, the size of the files will vary.
    - You cannot export nested and repeated data in CSV format. Nested and repeated data is supported for Avro and JSON exports.
    - When you export data in JSON format, INT64 (integer) data types are encoded as JSON strings to preserve 64-bit precision when the data is read by other systems.
    - You cannot export data from multiple tables in a single export job.
    - You cannot choose a compression type other than GZIP when you export data using the Cloud Console or the classic BigQuery web UI.
- All are charged, no charges for exporting and loading data in same region
- Cache
    - By default, a query's results are cached
    - BigQuery caches query results for 24 hours
    - When a destination table is specified in the job configuration, the Cloud Console, the bq command-line tool, or the API, the query results are not cached
    - There is no charge for a query that retrieves its results from cache
- Only Cloud sql is not available to be used for loading data directly

## Dataflow
- Used for pipeline, serverless, it’s Apache Beam
- GroupByKey vs CoGroupByKey: Aggregates all input elements by their key and allows downstream processing to consume all values associated with the key. While GroupByKey performs this operation over a single input collection and thus a single type of input values, CoGroupByKey operates over multiple input collections.
- Dataflow pipelines can also run on alternate runtimes like Spark and Flink, as they are built using the Apache Beam SDKs
- Dataflow's default windowing behavior is to assign all elements of a PCollection to a single, global window, even for unbounded PCollections
- In Google Cloud, the Dataflow SDK provides a transform component. It is responsible for the data processing operation. You can use conditional, for loops, and other complex programming structure to create a branching pipeline.
- ParDo collects the zero or more output elements into an output PCollection. The ParDo transform processes elements independently and possibly in parallel
- ParDo is a Beam transform for generic parallel processing. The ParDo processing paradigm is similar to the 'Map' phase of a Map/Shuffle/Reduce-style algorithm: a ParDo transform considers each element in the input PCollection, performs some processing function (your user code) on that element, and emits zero, one, or multiple elements to an output PCollection
- DirectPipelineRunner allows you to execute operations in the pipeline directly, without any optimization. Useful for small local execution and tests
- Types of window 
    - fixed(tumbling): no overlapping
    - sliding(hopping): e.g., detect load of a machine in 30 mins, do it every minute，then the size of the window is 30 mins and interval is 1 minute. 
    - session: for example, a shopping cart
- If you are using unbounded PCollections, you must use either non-global windowing or an aggregation trigger in order to perform a GroupByKey or CoGroupByKey
- A watermark is a threshold that indicates when Dataflow expects all of the data in a window to have arrived. If new data arrives with a timestamp that's in the window but older than the watermark, the data is considered late data.
- Triggers control when the elements for a specific key and window are output. As elements arrive, they are put into one or more windows by a Window transform and its associated WindowFn, and then passed to the associated Trigger to determine if the Windows contents should be output.
- There are three major kinds of triggers that Dataflow supports: 
    - Time-based triggers
        - Processing time triggers. These triggers operate on the processing time the time when the data element is processed at any given stage in the pipeline.
        - Event time triggers. These triggers operate on the event time, as indicated by the timestamp on each data element. Beams default trigger is event time-based.
    - Data-driven triggers. You can set a trigger to emit results from a window when that window has received a certain number of data elements.
    - Composite triggers. These triggers combine multiple time-based or data-driven triggers in some logical way
## Dataproc
- Used to manage Hadoop/Spark/Flink/Presto/Pig/Hive etc. clusters
- minute-by-minute billing 
- The YARN ResourceManager and the HDFS NameNode interfaces are available on a Cloud Dataproc cluster master node
- When using Cloud Dataproc clusters, configure your browser to use the SOCKS proxy. The SOCKS proxy routes data intended for the Cloud Dataproc cluster through an SSH tunnel.
- Used if migrating existing on-premise Hadoop or Spark infrastructure to GCP without redevelopment effort.
- The following rules will apply when you use preemptible workers with a Cloud Dataproc cluster:
    - Processing onlySince preemptibles can be reclaimed at any time, preemptible workers do not store data. Preemptibles added to a Cloud Dataproc cluster only function as processing nodes.
    - No preemptible-only clustersTo ensure clusters do not lose all workers, Cloud Dataproc cannot create preemptible-only clusters.
    - Persistent disk sizeAs a default, all preemptible workers are created with the smaller of 100GB or the primary worker boot disk size. This disk space is used for local caching of data and is not available through HDFS.
    - The managed group automatically re-adds workers lost due to reclamation as capacity permits.
- Use Dataflow for streaming instead. This is better for batch.
- Can use on disk (HDFS) or GCS
- Preemptible workers are the default secondary worker type. They are reclaimed and removed from the cluster if they are required by Google Cloud for other tasks. Although the potential removal of preemptible workers can affect job stability, you may decide to use preemptible instances to lower per-hour compute costs for non-critical data processing or to create very large clusters at a lower total cost
- Parquet files are composed of row groups, header and footer. Each row group contains data from the same columns. The same columns are stored together in each row group: This structure is well-optimized both for fast query performance, as well as low I/O (minimizing the amount of data scanned).
- Cloud Storage connector
    - The Cloud Storage connector is an open source Java library that lets you run Apache Hadoop or Apache Spark jobs directly on data in Cloud Storage,
    - Store your data in Cloud Storage and access it directly. You do not need to transfer it into HDFS first
- Local HDFS storage is a good option if:
    - Your jobs require a lot of metadata operations—for example, you have thousands of partitions and directories, and each file size is relatively small.
    - You modify the HDFS data frequently or you rename directories. (Cloud Storage objects are immutable, so renaming a directory is an expensive operation because it consists of copying all objects to a new key and deleting them afterwards.)
    - You heavily use the append operation on HDFS files.
    - You have workloads that involve heavy I/O. For example, you have a lot of partitioned writes, such as the following: spark.read().write.partitionBy(...).parquet("gs://")
    - You have I/O workloads that are especially sensitive to latency. For example, you require single-digit millisecond latency per storage operation.
## Pub/Sub
- if messages exceed the 10MB maximum, they cannot be published
- json payload base64 encoded
- undeliverd message are deleted after message retention units, by default 7 days
- Subscriptions have a lifecycle, expire after 31 days of inactivity (can change it up to 365 days)
- Messages are not received in order, for a lot of cases that is okay. More important there is a message. To order these messages you can attach a timestamp then order later. IF order is a problem consider other systems like kafka

## Datalab
- Managed Jupyter notebooks

## operations(stackdriver)
- like cloudwatch
- logging, monitoring

##  ML
- Vocabulary list is used when column values are known(incremental values are added as categorical columns starting from 0) and hash_bucket is used when you don’t know the values.
- Cloud machine learning engine only support Tensorflow
- To run a TensorFlow training job on your own computer using Cloud Machine Learning Engine
    - gcloud ml-engine local train
- TPU support Models with no custom TensorFlow operations inside the main training loop
- BigQuery ML requires a lot of data to build an accurate model
- AutoML uses transfer learning based on other similar data

## Others
- Pig is scripting language which can be used for checkpointing and splitting pipelines
- Cloud Deployment Manager: is an infrastructure deployment service that automates the creation and management of Google Cloud resources. Write flexible template and configuration files and use them to create deployments that have a variety of Google Cloud services,
- Cloud composer, A fully managed workflow orchestration service built on Apache Airflow.Ease your transition to the cloud or maintain a hybrid data environment by orchestrating workflows that cross between on-premises and the public cloud. Create workflows that connect data, processing, and services across clouds to give you a unified data environment.
- Aggregated log sink will create a single sink for all projects, the destination can be a google cloud storage, pub/sub topic, bigquery table or a cloud logging bucket
- Cloud data loss prevention: Fully managed service designed to help you discover, classify, and protect your most sensitive data
- Cloud Trace is used to debug latency in applications
- Titan Security Key: Two-factor authentication (2FA) device
- Event Threat Detection: Scans for suspicious activity

## Resources:
- Notes from a guy: https://luminous-manuscript-743.notion.site/Google-Services-and-APIs-for-Data-Engineering-91781dca9a91435fa7b3256eb2b87c71
- Mock exam in passexam.com: https://passnexam.com/google/google-data-engineer/1
- Exam topic: https://www.examtopics.com/exams/google/professional-data-engineer/view/
