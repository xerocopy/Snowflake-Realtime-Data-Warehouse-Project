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
