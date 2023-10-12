---
title: "Converting Managed to External Tables in Databricks"
date: 2023-10-12T12:30:05+05:00
draft: false
tags: 
    - Databricks
    - Spark
    - External Tables
    - Managed Tables
---

## Introduction:
Are you managing data in Databricks and looking to optimize your table storage strategy? Converting managed tables to external tables can be a smart move. In this guide, we'll walk you through the process step by step, so you can take advantage of the benefits that external tables offer.

## What Are Managed and External Tables?
Before we dive into the conversion process, let's quickly clarify what managed and external tables are and why you might want to switch.

* **Managed Tables**: 

These are tables in Databricks where both the metadata and data are stored within Databricks. They are easy to set up but might not be the best choice for large datasets or when you need to store data in external data sources.

* **External Tables**: 

External tables in Databricks store only the metadata, while the data resides in an external data source, such as AWS S3 or Azure Data Lake Store. They offer better flexibility, scalability, and control over data storage.

## Converting Tables

There are many ways to convert managed tables to external in databricks.

* **Method 1: Reading and writing using Spark**

This method can be as simple as reading the table and then writing it in your target location. After that you can create external table on top of that data.

**Example:**
1. Convert the Managed Table to Delta Lake Format:
```python
df_managed = spark.table("your_managed_table_name")
df_managed.write.format("delta").mode("overwrite").save("dbfs:/your/delta/location")
```

2. Create an External Table Using Delta Lake:
```sql
CREATE TABLE your_external_table_name
USING delta
LOCATION 'dbfs:/your/delta/location'
```

This approach works fine. But the main drawback will be if you have many tables and all have different partitions and stuff, it may take quite some time to convert all tables.

* **Method 2: Using cp command**

Another approach will be to use the `cp` command, which allows you to easily copy data from one source to another location.

**Example:**
1. Copy the data using `cp` command:
```python
dbutils.fs.cp("dbfs:/user/hive/warehouse/your_db_name.db/managed_table_name", "dbfs:/your/delta/location", recurse=True)
```

2. Create an External Table Using Delta Lake:
```sql
CREATE TABLE your_external_table_name
USING delta
LOCATION 'dbfs:/your/delta/location'
```

However, this approach is quite slow. It may take quite some time to copy data from source to destination.

* **Method 3: Using CLONE Command**

This approach is by far the best and easy one. Also it is quite fast and also you dont need to provide partition columns or running `CREATE TABLE` query. Clones can be either deep or shallow.

**Example:**

```sql
CREATE OR REPLACE TABLE your_external_table_name
    DEEP CLONE your_managed_table_name location 'dbfs:/your/delta/location'
```

This will create an external table from your managed table along with all the data and metadata in your specified external location.

## Conclusion:
Converting managed tables to external tables in Databricks can lead to better data management, scalability, and flexibility. It's a useful technique when you want to keep the table's metadata while storing the data in external storage. Just remember to handle data carefully and back it up before making changes.

With this guide, you can confidently convert your tables and harness the full power of Databricks for your data management needs.