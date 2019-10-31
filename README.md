Table of Contents
=================
   * [MySQL Master-Slave Architecture](#mysql-master-slave-architecture)
      * [1 Edit the setting file of MySQL](#1-edit-the-setting-file-of-mysql)
      * [2 Operations on a master server](#2-operations-on-a-master-server)
      * [3 Operation on a slave database](#3-operation-on-a-slave-database)
      * [4 Multi-master-slave architecture](#4-multi-master-slave-architecture)
      
# MySQL Master-Slave Architecture
[![LICENSE](https://img.shields.io/hexpm/l/plug)](https://github.com/eShopool/Android-Application/blob/master/LICENSE)
[![MySQL](https://img.shields.io/badge/MySQL-5.7.28-blueviolet)](https://dev.mysql.com/downloads/mysql/)

## 1 Edit the setting file of MySQL
Normally the setting file is located at: C:\ProgramData\MySQL\MySQL Server X.X. The file name is “my.ini”.  
Add following setting to the file for both master and slave databases:  
``` 
server_id = 1             #every database must own a unique id 
log-bin= mysql-bin        #switch on the binary log function
replicate-do-db= xxxx     #The database needs to be replicated  
```  
reboot MySQL 

## 2 Operations on a master server
Create an account:
```
mysql>create user 'mysql1'@'x.x.x.x' identified by 'xxxxxx';
```  
Give privilege to the account:  
```
mysql>grant replication slave on *.* to 'mysql1'@'x.x.x.x';
```  
Refresh:  
```
mysql>flush privileges;
```  

Check the status of the master database:  
```
mysql >show master status;
```  

The output can be something like:  
```
|      File        | Position    |Binlog_Do_DB | Binlog_Ignore_DB|
| mysql-bin.000010 |    460    |   balabala      | manual,mysql    |

```  

Record the value of “File” and “Position”, which can be used later.  

## 3 Operation on a slave database  
Connect to the master database: 
```
mysql> CHANGE MASTER TO        
MASTER_HOST='x.x.x.x',                  #Host IP      
MASTER_USER='mysql1',                 #The account created       
MASTER_PASSWORD='xxxxxx',            #password      
MASTER_LOG_FILE='mysql-bin.000010',	 #value of “File”
MASTER_LOG_POS=73;                  #value of “Position”
```  
Start the slave process:  
```
mysql>start slave;
```  
Check the status of the slave database:  
```
mysql>show slave status\G
```  
The output can be something like:  
```
*************************** 1. row ***************************              
       Slave_IO_State: Waiting for master to send event                  
          Master_Host: x.x.x.x                  
          Master_User: mysql1                  
          Master_Port: 3306                
        Connect_Retry: 60             
      Master_Log_File: mysql-bin.000010         
  Read_Master_Log_Pos:460              
       Relay_Log_File:mysqld-relay-bin.000022                
        Relay_Log_Pos: 10450       
Relay_Master_Log_File: mysql-bin.000010             
     Slave_IO_Running: Yes            
    Slave_SQL_Running: Yes              
      Replicate_Do_DB:           
  Replicate_Ignore_DB: 
```  
If both values of “Slave_IO_Running” and “Slave_SQL_Running” are “Yes”, The setting has been updated successful.  


## 4 Multi-master-slave architecture
Do the same thing once again but remember to add those setting to the “my.ini” file.  
For master database:  
```
auto-increment-offset = 1.    #Initial value for an auto-increment column 
auto-increment-increment=2.   #unit for each increment
```  
For slave database:  
```
auto-increment-offset=2
auto-increment-increment=2 
```  
