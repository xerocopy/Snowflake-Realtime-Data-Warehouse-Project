# Snowflake-Real-Time-Data-Warehouse-Project

### Scope of the Project
Snowflake is on serverless cloud infrastructure.  The architecture of Snowflake is a mix of classic shared-disk and shared-nothing
database designs. Snowflake employs a central data repository for persistent data, like
shared-disk architectures, available from all compute nodes in the platform. Snowflake
executes queries utilizing MPP (massively parallel processing) compute clusters,
wherein each node in the cluster stores a part of the whole data set locally, like
shared-nothing architectures. This method combines the simplicity of a shared-disk
design with the speed and scale-out advantages of a shared-nothing architecture.

Technology in project: AWS S3, SnowFlake, SnowSQL, QuickSight


### Data Pipeline Architecture

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





