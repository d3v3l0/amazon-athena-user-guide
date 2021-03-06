# Table Location in Amazon S3<a name="tables-location-format"></a>

When you run a `CREATE TABLE` query in Athena, you register your table with the AWS Glue Data Catalog\. \(If you are using Athena's older internal catalog, we highly recommend that you [upgrade](glue-upgrade.md) to the AWS Glue Data Catalog\.\) 

To specify the path to your data in Amazon S3, use the `LOCATION` property, as shown in the following example:

```
CREATE EXTERNAL TABLE `test_table`(
...
)
ROW FORMAT ...
STORED AS INPUTFORMAT ...
OUTPUTFORMAT ...
LOCATION s3://bucketname/folder/
```
+ For information about naming buckets, see [Bucket Restrictions and Limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) in the *Amazon Simple Storage Service Developer Guide*\.
+ For information about using folders in Amazon S3, see [Using Folders](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/using-folders.html) in the *Amazon Simple Storage Service Console User Guide\.* 

The `LOCATION` in Amazon S3 specifies *all* of the files representing your table\. 

**Important**  
Athena reads *all* data stored in `s3://bucketname/folder/'`\. If you have data that you do *not* want Athena to read, do not store that data in the same Amazon S3 folder as the data you want Athena to read\. If you are leveraging partitioning, to ensure Athena scans data within a partition, your `WHERE` filter must include the partition\. For more information, see [Table Location and Partitions](#table-location-and-partitions)\.

When you specify the `LOCATION` in the `CREATE TABLE` statement, use the following guidelines:
+ Use a trailing slash\.

   **Use**:

  ```
  s3://bucketname/folder/
  ```
+ Do not use any of the following items for specifying the `LOCATION` for your data\.
  + Do not use filenames, underscores, wildcards, or glob patterns for specifying file locations\.
  + Do not add the full HTTP notation, such as `s3.amazon.com` to the Amazon S3 bucket path\.
  + Do not specify an Amazon S3 [ access point](https://aws.amazon.com/s3/features/access-points/) in the `LOCATION` clause\.
**Note**  
Although you cannot specify an access point in the LOCATION clause, Athena can query objects using a bucket's access point as long as the requestor has the right access permissions to the objects in the bucket\.
  + Do not use empty folders like `//` in the path, as follows: `S3://bucketname/folder//folder/`\. While this is a valid Amazon S3 path, Athena does not allow it and changes it to `s3://bucketname/folder/folder/`, removing the extra `/`\. 

     **Do not use**:

    ```
    s3://path_to_bucket
    s3://path_to_bucket/*
    s3://path_to_bucket/mySpecialFile.dat
    s3://bucketname/prefix/filename.csv
    s3://test-bucket.s3.amazon.com
    S3://bucket/prefix//prefix/
    arn:aws:s3:::bucketname/prefix
    s3://arn:aws:s3:<region>:<account_id>:accesspoint/<accesspointname>
    https://<accesspointname>-<number>.s3-accesspoint.<region>.amazonaws.com
    ```

## Table Location and Partitions<a name="table-location-and-partitions"></a>

 Your source data may be grouped into Amazon S3 folders called *partitions* based on a set of columns\. For example, these columns may represent the year, month, and day the particular record was created\. 

When you create a table, you can choose to make it partitioned\. When Athena executes an SQL query against a non\-partitioned table, it uses the `LOCATION` property from the table definition as the base path to list and then scan all available files\. However, before a partitioned table can be queried, you must update the AWS Glue Data Catalog with partition information\. This information represents the schema of files within the particular partition and the `LOCATION` of files in Amazon S3 for the partition\. 
+ To learn how the AWS Glue crawler adds partitions, see [How Does a Crawler Determine When to Create Partitions? ](https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html#crawler-s3-folder-table-partition) in the *AWS Glue Developer Guide*\. 
+ To learn how to configure the crawler so that it creates tables for data in existing partitions, see [Using Multiple Data Sources with Crawlers](glue-best-practices.md#schema-crawlers-data-sources)\. 
+ You can also create partitions in a table directly in Athena\. For more information, see [Partitioning Data](partitions.md)\.

When Athena executes a query on a partitioned table, it checks to see if any partitioned columns are used in the `WHERE` clause of the query\. If partitioned columns are used, Athena requests the AWS Glue Data Catalog to return the partition specification matching the specified partition columns\. The partition specification includes the `LOCATION` property that tells Athena which Amazon S3 prefix to use when reading data\. In this case, *only* data stored in this prefix is scanned\. If you do not use partitioned columns in the `WHERE` clause, Athena scans all the files that belong to the table's partitions\. 

For examples of using partitioning with Athena to improve query performance and reduce query costs, see [Top Performance Tuning Tips for Amazon Athena](http://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/)\.