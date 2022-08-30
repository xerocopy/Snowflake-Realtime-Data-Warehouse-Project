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


Part-1 architecture

![alt text](https://github.com/xerocopy/Snowflake-Real-Time-Data-Warehouse-Project/blob/af2a3b3f02a3809eef11e01005816a186d1e88fe/pipeline_architecture.PNG)

VPC Framework
![alt text](https://github.com/xerocopy/Snowflake-Real-Time-Data-Warehouse-Project/blob/9b0579f1056d8fb8a0bd14ba7d33bed2801a7d55/pic0.PNG)

Data Flow in Snowflake
![alt text](https://github.com/xerocopy/Snowflake-Real-Time-Data-Warehouse-Project/blob/c22b480a6ed855dc44a5487cf40d77aaaebc3a58/pic1.PNG)

Snow Flake Pipeline Architecture
![alt text](https://github.com/xerocopy/Snowflake-Real-Time-Data-Warehouse-Project/blob/c22b480a6ed855dc44a5487cf40d77aaaebc3a58/pic2.PNG)


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
accountname = ASD123456.us-east-1
username = XEROCOPY
password = *******

4. firt time setup with command:
snowsql -c myconnection


### load data using snowsql 
1. snowsql -c myconnection  

2. use test_db;

3. SELECT * FROM CUSTOMER_DETAIL LIMIT 10;

4. Create PIPE FORMAT

create or replace file format PIPE_FORMAT_CLI
  type = 'CSV'  
  field_delimiter = '|'  
  skip_header = 1;
  
5. Create state for snowflake

create or replace stage PIPE_CLI_STAGE
file_format = PIPE_FORMAT_CLI;
  
6. Put customer_detail csv in stage. Snowflake will load the table from this stage

put
file://C:\Users\jingc\Downloads\Snowflake_Real-Time_Data_Warehouse_Project_forBeginners-1\Data\teslaData\customer_detail.csv
@PIPE_CLI_STAGE auto_compress=true;

7. List stage to see how many files are loaded in stage

list @PIPE_CLI_STAGE;


8. Resume warehouse. In our case warehouse is set to auto-resume so no need to run this command. 

alter warehouse test_wh resume;
alter warehouse compute_wh resume;

9. Finally copy command to load dat int table from stage

copy into customer_detail
  from @PIPE_CLI_STAGE
  file_format = (format_name = PIPE_FORMAT_CLI)
  on_error = 'skip_file';
 
select * from customer_detail;  # to check the table content
  
 
10. We can also give copy command with pattern if your stage contain multile files.
 
copy into mycsvtable
  from @my_csv_stage
  file_format = (format_name = mycsvformat)
  pattern = '.*contacts[1-5].csv.gz'
  on_error = 'skip_file';



### Data Load from aws S3 using snowflake web interface

&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&

--set up db

CREATE DATABASE TEST_DB;

use test_db;

--empty table

DROP TABLE IF EXISTS TESLA_DATA;

--Create Tesla_Data table
CREATE TABLE TESLA_DATA (
    Date            date,
    Open_value      double,
    High_value      double,
    Low_value       double,
    Close_value     double,
    Adj_Close       double,
    volume          bigint
    );

&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&

--Bulk_Copy Batch_load

select * from tesla_data;
    
CREATE OR REPLACE STAGE BULK_COPY_TESLA_STAGE URL="s3://snowflakecomputingpro-123/TSLA.csv"
CREDENTIALS=(AWS_KEY_ID='*****************' AWS_SECRET_KEY='******************************'); 

--List the content of the stage

LIST @BULK_COPY_TESLA_STAGE;

--Copy data from table

COPY INTO TESLA_DATA
FROM @BULK_COPY_TESLA_STAGE
FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = ',' SKIP_HEADER = 1);

SELECT * FROM TESLA_DATA LIMIT 10;

&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&

-- Set up Automation  Continous_Load

//use role accountadmin;

//GRANT CREATE INTEGRATION on account to role accountadmin;
//GRANT USAGE on S3_INTEGRATION to ROLE ACCOUNTADMIN;

--Create Storage Integration
CREATE or replace STORAGE INTEGRATION S3_INTEGRATION_SYSADMIN
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = S3
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::***************:role/Snowflake_Access_Role'
ENABLED =TRUE
STORAGE_ALLOWED_LOCATIONS = ('s3://snowflakecomputingpro-123/Input/');

--need to check the updated exteranalID after creating the new integration
desc integration S3_INTEGRATION_SYSADMIN;

CREATE or REPLACE STAGE TESLA_DATA_STAGE_SYSADMIN
URL = 's3://snowflakecomputingpro-123/Input/'
STORAGE_INTEGRATION = S3_INTEGRATION_SYSADMIN
FILE_FORMAT = CSV_FORMAT;

list @TESLA_DATA_STAGE_SYSADMIN;

SELECT * FROM TESLA_DATA order by date desc LIMIT 10;

COPY INTO TESLA_DATA
FROM @TESLA_DATA_STAGE_SYSADMIN
PATTERN = '.*.csv';

CREATE or REPLACE pipe TESLA_PIPE_TEST AUTO_INGEST = TRUE as 
COPY INTO TEST_DB.PUBLIC.TESLA_DATA
FROM @TESLA_DATA_STAGE_SYSADMIN
FILE_FORMAT = CSV_FORMAT;

SHOW PIPES;

&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&

--switch to sysadmin and set up the correct role in snowflake when necessary
use role accountadmin;
GRANT CREATE INTEGRATION on account to role sysadmin;

GRANT USAGE ON DATABASE TEST_DB TO SYSADMIN

GRANT ALL ON WAREHOUSE TEST_WH TO ROLE sysadmin

GRANT ALL ON ACCOUNT TO ROLE sysadmin

GRANT OWNERSHIP ON DATABASE test_db TO ROLE sysadmin COPY CURRENT GRANTS;    
GRANT OWNERSHIP ON ALL SCHEMAS IN DATABASE test_db TO ROLE sysadmin COPY CURRENT GRANTS;
GRANT OWNERSHIP ON ALL TABLES IN DATABASE test_db TO ROLE sysadmin COPY CURRENT GRANTS; 
GRANT OWNERSHIP ON ALL VIEWS IN DATABASE test_db TO ROLE sysadmin COPY CURRENT GRANTS; 

use role sysadmin

SHOW SCHEMAS IN DATABASE test_db;




## Part 2

CREATE DATABASE PRO_DB;

USE PRO_DB

CREATE SCHEMA PRO_SCHEMA;

CREATE WAREHOUSE PRO_CURATION WITH WAREHOUSE_SIZE = 'XSMALL' WAREHOUSE_TYPE = 'STANDARD' AUTO_SUSPEND = 600 AUTO_RESUME = TRUE;


DROP TABLE IF EXISTS CUSTOMER_RAW;
CREATE TABLE CUSTOMER_RAW (C_CUSTKEY NUMBER(38,0),
                           C_NAME VARCHAR(25),
                           C_ADDRESS VARCHAR(40),
                           C_NATIONKEY NUMBER(38,0),
                           C_PHONE VARCHAR(15),
                           C_ACCTBAL NUMBER(12,2),
                           C_MKTSEGMENT VARCHAR(10),
                           C_COMMENT VARCHAR(117),
                           BATCH_ID DOUBLE
                           );

DROP TABLE IF EXISTS ORDERS_RAW;
CREATE TABLE ORDERS_RAW (O_ORDERKEY NUMBER(38,0),
                         O_CUSTKEY NUMBER(38,0),
                         O_ORDERSTATUS VARCHAR(1),
                         O_TOTALPRICE NUMBER(12,2),
                         O_ORDERDATE DATE,
                         O_ORDERPRIORITY VARCHAR(15),
                         O_CLERK VARCHAR(15),
                         O_SHIPPRIORITY NUMBER(38,0),
                         O_COMMENT VARCHAR(79),
                         BATCH_ID DOUBLE
                         );

USE PRO_DB.PRO_SCHEMA;

SELECT * FROM ORDERS_RAW;

SELECT * FROM CUSTOMER_RAW;

CREATE OR REPLACE STORAGE INTEGRATION S3_INTEGRATION_PRO
    TYPE = EXTERNAL_STAGE
    STORAGE_PROVIDER = S3
    STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::516003265142:role/Snowflake_Access_Role'
    ENABLED = TRUE
    STORAGE_ALLOWED_LOCATIONS = ('s3://snowflakedatapipelinepro-123/firehose/');

DESC INTEGRATION S3_INTEGRATION_PRO

CREATE OR REPLACE STAGE CUSTOMER_RAW_STAGE
    URL = 's3://snowflakedatapipelinepro-123/firehose/customers/'
    STORAGE_INTEGRATION = S3_INTEGRATION_PRO
    FILE_FORMAT=CSV_FORMAT;
    
CREATE OR REPLACE STAGE ORDERS_RAW_STAGE
    URL = 's3://snowflakedatapipelinepro-123/firehose/orders/'
    STORAGE_INTEGRATION = S3_INTEGRATION_PRO
    FILE_FORMAT=CSV_FORMAT;

COPY INTO PRO_DB.PRO_SCHEMA.CUSTOMER_RAW
(C_CUSTKEY, C_NAME, C_ADDRESS, C_NATIONKEY, C_PHONE, C_ACCTBAL, C_MKTSEGMENT, C_COMMENT, BATCH_ID) 
FROM (SELECT t.$1, t.$2, t.$3, t.$4, t.$5, t.$6, t.$7, t.$8, '20220826160201' FROM @CUSTOMER_RAW_STAGE t);


COPY INTO PRO_DB.PRO_SCHEMA.ORDERS_RAW
(O_ORDERKEY , O_CUSTKEY, O_ORDERSTATUS, O_TOTALPRICE, O_ORDERDATE, O_ORDERPRIORITY, O_CLERK, O_SHIPPRIORITY, O_COMMENT, BATCH_ID)
FROM (SELECT t.$1, t.$2, t.$3, t.$4, t.$5, t.$6, t.$7, t.$8, t.$9, '20220826160201' FROM @ORDERS_RAW_STAGE t);










