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

put file://C:\Users\jingc\Downloads\Snowflake_Real-Time_Data_Warehouse_Project_forBeginners-1\Data\teslaData\customer_detail.csv
@PIPE_CLI_STAGE auto_compress=true;

7. List stage to see how many files are loaded in stage

list @PIPE_CLI_STAGE;

