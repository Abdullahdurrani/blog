---
title: "Working With MinIO and Spark"
date: 2023-03-24T22:04:36+05:00
---

In this post, we'll explore how to use Minio and Spark together. Before jumping into Spark and MinIO let's first get a brief introduction to Spark and MinIO.

## Spark
Apache Spark is a fast and flexible open-source data processing engine that's used to process large datasets in parallel across a cluster of computers. 

Some of the benefits of Spark include its ability to handle large datasets quickly, its support for multiple programming languages, its ability to run on a variety of hardware and cloud platforms.

### How to install Spark
[Ways to Install Pyspark for Python](https://sparkbyexamples.com/pyspark/install-pyspark-for-python/#:~:text=Ways%20to%20Install%20Pyspark%20for%20Python%201%201%3A,...%204%204.%20Test%20PySpark%20Install%20from%20Shell)

## MinIO
MinIO is a powerful, open-source object storage system that is designed for storing and accessing large amounts of unstructured data. 

One of the key benefits of MinIO is its versatility - it can be deployed on-premises, in the cloud, or even on a local machine as a data lake for developers. 

Other benefits of MinIO include its support for the Amazon S3 API, its compatibility with a wide range of applications and programming languages.

### How to install MinIO
https://min.io/download#/windows

Now you have MinIO and Spark installed. Let's start working with MinIO and Spark.

First create `access_key`, `secret_key` from MinIO console. They are used to identify the user or application that is accessing the MinIO server.

### Working with Spark
Create a python file and copy the following code to read from MinIO bucket.

*Replace `minio_access_key`, `minio_secret_key` with the keys you generated earlier from MinIO console*.
```
# Import the SparkSession module
from pyspark.sql import SparkSession

# Create a SparkSession
spark = SparkSession.builder.getOrCreate()

# Get the SparkContext from the SparkSession
sc = spark.sparkContext

# Set the MinIO access key, secret key, endpoint, and other configurations
sc._jsc.hadoopConfiguration().set("fs.s3a.access.key", "minio_access_key")
sc._jsc.hadoopConfiguration().set("fs.s3a.secret.key", "minio_secret_key")
sc._jsc.hadoopConfiguration().set("fs.s3a.endpoint", "http://localhost:9000")
sc._jsc.hadoopConfiguration().set("fs.s3a.path.style.access", "true")
sc._jsc.hadoopConfiguration().set("fs.s3a.connection.ssl.enabled", "false")
sc._jsc.hadoopConfiguration().set("fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem")
sc._jsc.hadoopConfiguration().set("fs.s3a.connection.ssl.enabled", "false")

# Read a JSON file from an MinIO bucket using the access key, secret key, 
# and endpoint configured above
df = spark.read.option("header", "true") \
    .json(f"s3a://{minio_bucket}/file.json")

# show data
df.show()
```
Running this code you may encounter the following error:
```
Py4JJavaError: An error occurred while calling o123.json.
: java.lang.RuntimeException: java.lang.ClassNotFoundException: Class org.apache.hadoop.fs.s3a.S3AFileSystem not found
```

To solve this error we need to download the following dependencies:
* JAR file: `hadoop-aws`
* JAR file: `aws-java-sdk-bundle`

### Downloading `hadoop-aws`
We need to check which version to install for `hadoop-aws`. All Hadoop JARs must have the exact same version. So, if existing Hadoop JARs are `3.2.2`, then we should download `hadoop-aws:3.2.0`

One way to find out the exisitng Hadoop JARs versions is to check jars folder inside your pyspark folder. 
```
cd pyspark/jars && ls -l | grep hadoop
```

Once you know which `hadoop-aws` version to install, head over to this link: [hadoop-aws maven repo](https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-aws) and download required version.

### Downloading `aws-java-sdk-bundle`
On the `hadoop-aws` download page scroll down a little and you will see    
**Compile Dependencies** section. Download the `aws-java-sdk-bundle` under the **Version** column.

Once you have downloaded the JARs, copy them inside your `your-pyspark-folder/jars/` folder.

Now run the script again. This time maybe you will get the following error:
```
py4j.protocol.Py4JJavaError: An error occurred while calling o68.start.
: java.io.IOException: From option fs.s3a.aws.credentials.provider 
java.lang.ClassNotFoundException: Class 
org.apache.hadoop.fs.s3a.auth.IAMInstanceCredentialsProvider not found
```

To solve this error change your spark session to:
```
spark = SparkSession.builder \
    .config("spark.hadoop.fs.s3a.aws.credentials.provider", "org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider") \
    .getOrCreate()
```

Hopefully this will solve your error. If you still get this error try replacing your existing `guava-xx.x.jar` with `guava-23.0.jar` or `guava-30.0.jar`.

### Writing to MinIO
You can write to MinIO using following code. 
```
df.write.mode("overwrite").parquet(f"s3a://{minio_bucket}/file.parquet")
```
While writing you may encounter the following error:
```
An error occurred while calling o1152.parquet.
: org.apache.spark.SparkException: Job aborted.
	at org.apache.spark.sql.errors.QueryExecutionErrors$.jobAbortedError(QueryExecutionErrors.scala:638)
```
To solve this error download the `hadoop.dll` from [winutils hadoop](https://github.com/cdarlint/winutils/blob/master)
according to your hadoop version. Copy this dll to your hadoop folder in your windows.

Once you have all your errors sorted you will be able to interact with MinIO using your pyspark code.