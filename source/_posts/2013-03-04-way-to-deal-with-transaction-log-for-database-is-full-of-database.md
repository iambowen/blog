---
layout: post
title: way to deal with "transaction log for database is full" issue
tag: database
categories: ["database"]
---

Recently, the ci build has failed with the following error message:

``` bash
_ODBC::Error: 37000 (9002) [unixODBC][FreeTDS][SQL Server]The transaction log for
database 'Warehouse' is full. To find out why space in the log cannot be reused,
see the log_reuse_wait_desc column in sys.databases (Sequel::DatabaseError)_*%
```

The reason is, so far as I known, that firstly the database has the log recorded
and when the volume of log increasing, it would cause the "out of space" error
if it exceeds the threshold of log size limit.


The knowledge that we should know is:

   # Scheme of way to record transaction log is store in *log_reuse_wait_desc*
   # column of *sys.databases*, check "this":http://msdn.microsoft.com/en-us/library/ms178534.aspx, it gives all the choices of how to deal with the transaction log

A little bit log for this document, I will just cut the cliche and show how I
solve this issue with the knowledge I found from Internet.


*Given* our current build is using data warehouse "Warehouse" and such error happens

*First* thing is to connect to the data warehouse, and check whether the log is
full by using *DBCC* tools.

Check log spaces used:

``` bash
DBCC SQLPERF(LOGSPACE) //from the output information you can also see the log name of corresponding database
```

Then you will see the log size for *"Warehouse"* is 100%
After knowing it is the right reason as the error message shows, we have two steps to go:

# set the data warehouse into recovery mode and set the log_reuse_wait_desc using no backup for reserving logs
# truncate the currently logs and shrink its size

Way to change the database recovery mode and set backup scheme:

```sql
USE Warehouse
GO  
ALTER DATABASE Warehouse SET RECOVERY full  
GO
ALTER DATABASE Warehouse SET RECOVERY SIMPLE WITH NO_WAIT
```

way to shrink the log file:

``` sql
USE Warehouse
DBCC SHRINKFILE('Warehouse_log', 0, TRUNCATEONLY)
```

Then you will get new log scheme with no backup and a log size less than 1MB. Of
course, the most important thing is that the ci build won't fail....
