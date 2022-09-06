# Snowflake-Real-Time-Data-Warehouse-Project

### Scope of the Project
Snowflake is on serverless cloud infrastructure.  The architecture of Snowflake is a mix of classic shared-disk and shared-nothing
database designs. Snowflake employs a central data repository for persistent data, like
shared-disk architectures, available from all compute nodes in the platform. Snowflake
executes queries utilizing MPP (massively parallel processing) compute clusters,
wherein each node in the cluster stores a part of the whole data set locally, like
shared-nothing architectures. This method combines the simplicity of a shared-disk
design with the speed and scale-out advantages of a shared-nothing architecture.

### Technologies used in project: 
AWS IAM, S3, SnowFlake, SnowSQL, QuickSight, Airflow (Amazon Managed Workflows for Apach Airflow(MWAA)), Kinesis FireHose

### Role created in IAM
SNOWFLAKE_ACCESS_ROLE
AmazonMWAA-SNOWFLAKEDATAPIPELINEGPRO-123

### List of Assets
Data folder
img folder
SnowSQL-Command.txt
customer_orders_raw_insert.py
requirements.txt
snowpipe_ETL_PRO_DB.txt
snowpipe_ETL_TEST_DB.txt

### Data Pipeline Architecture
Overall Architecture
![alt text](https://github.com/xerocopy/Snowflake-Real-Time-Data-Warehouse-Project/blob/03ca51e929bdf9a9512c5fd3adfbe3536faaee46/Snowflake_Airflow_DataPipeline_Architecture.jpg)

### General Steps Loading Bulk data from Cloud & Local Storage

1. Prepare the files into a format that is optimal for loading;

2. Stage the data - Make snowflake aware of the data. Internal or external staging area.

3. Excute Copy command - COPY the data into the table

4. Managing regular loads -Organize files & schedule loads

### Set Up S3 Buckets
![alt text|10x6](https://github.com/xerocopy/Snowflake-Realtime-Data-Warehouse-Project/blob/6702fd69ef52ea34756f7833532b48e8dbead9c3/img/s3.PNG)

### Set Up Kinesis FireHose
![alt text|10x8](https://github.com/xerocopy/Snowflake-Realtime-Data-Warehouse-Project/blob/bccefd6c7332857ecd241135e1a69e80bf47614e/img/kinesis.PNG)

### Snow SQL installation and setup

1. find the installation instructions at:
https://docs.snowflake.com/en/user-guide/snowsql-install-config.html#installing-snowsql-on-microsoft-windows-using-the-installer

2. download the latest version of snowsql in snowflake repository:
https://sfc-repo.snowflakecomputing.com/snowsql/bootstrap/1.2/windows_x86_64/index.html

3. change Configureation file under username/.snowsql.cnf
[connections.myconnection]
#Can be used in SnowSql as #connect example
accountname = ABC123456.us-east-1
username = XEROCOPY
password = *******

4. firt time setup with command:
snowsql -c myconnection

### load data using snowsql and run the SnowSQL-Command.txt
run the SQL queries in SnowSQL-Command.txt

### Data Load from aws S3 using snowflake web interface
run the SQL in the snowpile_ETL_TEST_DB.txt file

### Part 2 ETL 1 from S3 landing--> processing--> processed 

run the SQL in the snowpipe_ETL_PRO_DB.txt file

### Set Up Apache Airflow in AWS
<div style="width:60px ; height:60px">
  ![alt text](https://github.com/xerocopy/Snowflake-Realtime-Data-Warehouse-Project/blob/64819fabb6ed65a11016bfe33913922e49a44d74/img/awsMWAA_dags.PNG)

![alt text](https://github.com/xerocopy/Snowflake-Realtime-Data-Warehouse-Project/blob/64819fabb6ed65a11016bfe33913922e49a44d74/img/airflow_add_conn.PNG)

![alt text](https://github.com/xerocopy/Snowflake-Realtime-Data-Warehouse-Project/blob/64819fabb6ed65a11016bfe33913922e49a44d74/img/successful_dags_graph.PNG)

### Set Up QuickSight
![alt text](https://github.com/xerocopy/Snowflake-Realtime-Data-Warehouse-Project/blob/64819fabb6ed65a11016bfe33913922e49a44d74/img/setup_vis_QuickSight.PNG)

![alt text](https://github.com/xerocopy/Snowflake-Realtime-Data-Warehouse-Project/blob/64819fabb6ed65a11016bfe33913922e49a44d74/img/QuickSight_demo.PNG)

