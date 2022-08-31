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
AWS S3, SnowFlake, SnowSQL, QuickSight, Airflow (Amazon Managed Workflows for Apach Airflow(MWAA)), Kinesis

### Role created in IAM
SNOWFLAKE_ACCESS_ROLE

AmazonMWAA-SNOWFLAKEDATAPIPELINEGPRO-123

### List of Assets
Dags.py
requirements.txt
sql
data - test data for first dag


### Data Pipeline Architecture
Overall Architecture
![alt text](https://github.com/xerocopy/Snowflake-Real-Time-Data-Warehouse-Project/blob/03ca51e929bdf9a9512c5fd3adfbe3536faaee46/Snowflake_Airflow_DataPipeline_Architecture.jpg)


### Loading Bulk data from Cloud & Local Storage

1. Prepare the files into a format that is optimal for loading;

2. Stage the data - Make snowflake aware of the data. Internal or external staging area.

3. Excute Copy command - COPY the data into the table

4. Managing regular loads -Organize files & schedule loads


### snowsql installation and setup

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




