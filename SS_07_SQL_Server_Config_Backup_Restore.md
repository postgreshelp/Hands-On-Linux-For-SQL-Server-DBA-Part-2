# Lab 17 -- Edit SQL Server Configuration, Backup and Restore

## Learning Objectives

By the end of this chapter, you will be able to:

- Back up and edit SQL Server's configuration file
- Configure a default backup location
- Take an online backup and restore it
- Verify a restore both from SQL Server and from Linux

------------------------------------------------------------------------

# Step 1 -- Create a Backup Directory

``` bash
mkdir -p /backup/sqlserver

chown mssql:mssql /backup/sqlserver

chmod 755 /backup/sqlserver

ls -ld /backup/sqlserver
```

Expected:

```
drwxr-xr-x. 2 mssql mssql 6 ... /backup/sqlserver
```

------------------------------------------------------------------------

# Step 2 -- Backup the Configuration File

``` bash
cp -p /var/opt/mssql/mssql.conf \
      /var/opt/mssql/mssql.conf.bkp
```

Verify

``` bash
ls -lh /var/opt/mssql/mssql.conf*
```

Expected:

```
mssql.conf
mssql.conf.bkp
```

------------------------------------------------------------------------

# Step 3 -- Edit Configuration File

Open the configuration file.

``` bash
vi /var/opt/mssql/mssql.conf
```

Add the following section.

``` ini
[filelocation]
defaultbackupdir = /backup/sqlserver
```

Example:

``` ini
[sqlagent]
enabled = false

[licensing]
azurebilling = false

[EULA]
accepteula = Y

[language]
lcid = 1033

[filelocation]
defaultbackupdir = /backup/sqlserver
```

Save and Exit

    Esc
    :wq

------------------------------------------------------------------------

# Step 4 -- Compare Changes

``` bash
diff \
/var/opt/mssql/mssql.conf.bkp \
/var/opt/mssql/mssql.conf
```

Expected:

```
> [filelocation]
> defaultbackupdir = /backup/sqlserver
```

------------------------------------------------------------------------

# Step 5 -- Restart SQL Server

``` bash
systemctl restart mssql-server

systemctl status mssql-server
```

Expected:

```
Active: active (running)
```

------------------------------------------------------------------------

# Step 6 -- Take an Online Backup

Connect:

``` bash
/opt/mssql-tools18/bin/sqlcmd \
-S localhost \
-U sa \
-C
```

Take the backup:

``` sql
BACKUP DATABASE LinuxDemo
TO DISK='LinuxDemo.bak'
WITH
INIT,
FORMAT,
STATS=10;
GO
```

Expected:

```
10 percent processed.
...
100 percent processed.
BACKUP DATABASE successfully processed ... pages ...
```

------------------------------------------------------------------------

# Step 7 -- Verify Backup from Linux

``` bash
ls -lh /backup/sqlserver

find /backup/sqlserver -name "*.bak"

du -sh /backup/sqlserver/LinuxDemo.bak
```

Expected: `LinuxDemo.bak` listed with a non-zero size.

------------------------------------------------------------------------

# Step 8 -- Inspect Backup Contents

``` sql
RESTORE FILELISTONLY FROM DISK='/backup/sqlserver/LinuxDemo.bak';
GO
```

Expected logical files:

-   LinuxDemo
-   LinuxDemo_log

------------------------------------------------------------------------

# Step 9 -- Restore Database

If the database already exists:

``` sql
USE master;
GO

ALTER DATABASE LinuxDemo SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
GO

DROP DATABASE LinuxDemo;
GO
```

Restore:

``` sql
RESTORE DATABASE LinuxDemo
FROM DISK='/backup/sqlserver/LinuxDemo.bak'
WITH
MOVE 'LinuxDemo'
TO '/var/opt/mssql/data/LinuxDemo.mdf',

MOVE 'LinuxDemo_log'
TO '/var/opt/mssql/data/LinuxDemo_log.ldf',

RECOVERY,
STATS=10;
GO
```

Expected:

```
RESTORE DATABASE successfully processed ... pages ...
```

------------------------------------------------------------------------

# Step 10 -- Verify Restore

``` sql
USE LinuxDemo;
GO

SELECT COUNT(*) AS TotalOrders
FROM dbo.Orders;
GO

SELECT *
FROM sales.Customers;
GO
```

Expected:

```
TotalOrders
-----------
5
```

Verify files:

``` bash
ls -lh /var/opt/mssql/data/LinuxDemo*
```

------------------------------------------------------------------------

## Challenge

Take a second backup of `LinuxDemo` after inserting a new row, restore
it to a **new** database called `LinuxDemoCopy` (using `MOVE` to
different file paths), and confirm the row count matches.

------------------------------------------------------------------------

# What You Learned

-   vi editor
-   cp
-   diff
-   mkdir
-   chown
-   chmod
-   systemctl
-   SQL Server configuration
-   Online backup
-   RESTORE FILELISTONLY
-   Database restore
-   Linux verification commands
