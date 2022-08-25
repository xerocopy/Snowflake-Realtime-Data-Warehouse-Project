# Snowflake-Real-Time-Data-Warehouse-Project



#### a few steps missing (update later) 

#### a fewo steps missing (update later)



#### snowsql installation and setup

1. find the installation instructions at:
https://docs.snowflake.com/en/user-guide/snowsql-install-config.html#installing-snowsql-on-microsoft-windows-using-the-installer

2. download the latest version of snowsql in snowflake repository:
https://sfc-repo.snowflakecomputing.com/snowsql/bootstrap/1.2/windows_x86_64/index.html

3. change Configureation file under username/.snowsql.cnf
[connections.myconnection]
#Can be used in SnowSql as #connect example
accountname = JDB80709.us-east-1
username = XEROCOPY
password = *******

4. firt time setup with command:
snowsql -c myconnection


#### load data using snowsql 
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



#### In snowflake web interface/worksheet

Use test_db;

CREATE TABLE TESLA_DATA (
    Date            date,
    Open_value      double,
    High_value      double,
    Low_value       double,
    Close_value     double,
    Adj_Close       double,
    volume          bigint
    );

--Creat External S3 storage
CREATE OR REPLACE STAGE BULK_COPY_TESLA_STAGE URL="s3://snowflakecomputingpro-123/TSLA.csv"

CREDENTIALS=(AWS_KEY_ID='************' AWS_SECRET_KEY='**********'); 


--List the content of the stage

LIST @BULK_COPY_TESLA_STAGE;

--Copy data from table

COPY INTO TESLA_DATA
FROM @BULK_COPY_TESLA_STAGE
FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = ',' SKIP_HEADER = 1);


select * from TESLA_DATA limit 10;


use role accountadmin;

GRANT CREATE INTEGRATION on account to role accountadmin;
GRANT USAGE on S3_INTEGRATION to ROLE ACCOUNTADMIN;


--Create Storage Integration
CREATE or replace STORAGE INTEGRATION S3_INTEGRATION_SYSADMIN
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = S3
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::516003265142:role/Snowflake_Access_Role'
ENABLED =TRUE
STORAGE_ALLOWED_LOCATIONS = ('s3://snowflakecomputingpro-123/Input/');

desc integration S3_INTEGRATION_SYSADMIN;


--update STORAGE_AWS_IAM_USER_ARN & STORAGE_AWS_EXTERNAL_ID in aws role trust relationship policy

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::079828672386:user/jfa10000-s"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "sts:ExternalId": "JDB80709_SFCRole=2_Ikdsa02Hha74XLCVqHiDiLKmgfI="
                }
            }
        }
    ]
}


--switch to sysadmin
use role accountadmin;
GRANT CREATE INTEGRATION on account to role sysadmin;

GRANT ALL ON ACCOUNT TO ROLE sysadmin

use role sysadmin;


CREATE or REPLACE STAGE TESLA_DATA_STAGE_SYSADMIN
URL = 's3://snowflakecomputingpro-123/Input/TSLA.csv'
STORAGE_INTEGRATION = S3_INTEGRATION_SYSADMIN
FILE_FORMAT = CSV_FORMAT;

list @TESLA_DATA_STAGE;

SELECT * FROM TESLA_DATA;

COPY INTO TESLA_DATA
FROM @TESLA_DATA_STAGE_SYSADMIN
PATTERN = '.*.csv';

