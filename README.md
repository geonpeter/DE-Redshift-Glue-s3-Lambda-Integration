# DE-Redshift-Glue-s3-Lambda-Integration
 This project demonstrates how to integrate Amazon Redshift with Amazon S3 using an event-driven architecture via AWS Lambda. The Lambda function is triggered whenever a new file is uploaded to S3, and it automatically loads the data into a Redshift table. Additionally, this project covers the use of internal tables, external tables (using Redshift Spectrum), materialized views, and the Redshift Data API.

![Diagram](![image](https://github.com/user-attachments/assets/3fea6d08-b557-48c5-a7dc-e28d5c0356b8)


## Architecture Overview
The solution consists of the following components:

1 Amazon S3: Used to store the data files (e.g., CSV files).
2 Amazon Redshift: The data warehouse where data is loaded from S3 into both internal and external tables.
3 AWS Lambda: An event-driven serverless function that triggers when a file is uploaded to S3 and automatically loads the data into Redshift.
4 Redshift Data API: Used by Lambda to execute SQL queries on Redshift without needing a persistent connection.
5 AWS Glue: Serves as the metadata catalog for the external tables.

## Features
- Internal Table: Data is loaded from S3 into Redshift using the COPY command.
- External Table (Redshift Spectrum): Query data directly from S3 without loading it into Redshift using Spectrum.
- Materialized View: Created to speed up queries by aggregating data from internal and external tables.
- Manifest File: Generated during the UNLOAD process to track files exported from Redshift to S3.
- AWS Lambda Trigger: Automatically triggers a data load into Redshift when a new file is uploaded to S3 using the Redshift Data API.

## Setup Instructions
- Prerequisites
- AWS Account
- Amazon Redshift Cluster (with Data API enabled)
- Amazon S3 Bucket for storing CSV data
- IAM Role with permissions for S3, Redshift, Lambda, and Redshift Data API
- Python 3.8+ for the Lambda function


1. Set Up Amazon Redshift
- Create an Internal Table in Redshift for loading data:
```sql
CREATE TABLE sales_internal (
    sale_id INT,
    customer_id INT,
    product_id INT,
    sale_amount DECIMAL(10, 2),
    sale_date DATE
);
```
- Create External Schema for Redshift Spectrum:
```sql
CREATE EXTERNAL SCHEMA spectrum_schema
FROM DATA CATALOG
DATABASE 'my_glue_database'
IAM_ROLE ***';
```
- Create External Table
```sql
CREATE EXTERNAL TABLE spectrum_schema.sales_external (
    sale_id INT,
    customer_id INT,
    product_id INT,
    sale_amount DECIMAL(10, 2),
    sale_date DATE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION 's3://*****'
TABLE PROPERTIES ('skip.header.line.count'='1');
```
- Create Materialized View
```sql
CREATE MATERIALIZED VIEW sales_summary AS
SELECT sale_date, SUM(sale_amount) AS total_sales
FROM (
    SELECT sale_date, sale_amount FROM sales_internal
    UNION ALL
    SELECT sale_date, sale_amount FROM spectrum_schema.sales_external
)
GROUP BY sale_date;
```
2 Setup s3
- Create an S3 Bucket: Upload your sample data files (e.g., sales_data.csv) to S3.
- Set up an event trigger for object uploads to notify the Lambda function.

3 Set Up AWS Lambda

- Create a Lambda Function: Use the following code to load data into Redshift when a new file is uploaded to S3.
- Add an S3 Trigger to invoke the Lambda function when a new object is uploaded.
```sql
PENDING
```

4  Set Up IAM Roles
- Create an IAM Role for Lambda that allows it to access S3, Redshift, and execute SQL queries via the Data API.
- Attach this role to the Lambda function.




