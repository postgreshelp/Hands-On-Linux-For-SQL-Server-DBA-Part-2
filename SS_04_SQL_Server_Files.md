# SQL Server Files
> 🌐 **labs.postgreshelp.com** | Hands-on Database Engineering Labs

## Learning Objectives

By the end of this chapter, you will be able to:

- Locate SQL Server's data (`.mdf`) and log (`.ldf`) files on disk
- Tail the SQL Server error log for live troubleshooting
- Read SQL Server's configuration file
- Cross-check physical files against `sys.master_files`

------------------------------------------------------------------------

## Locate Files on Disk

``` bash
find /var/opt/mssql -name "*.mdf"
```

Expected:

```
/var/opt/mssql/data/LinuxDemo.mdf
```

``` bash
find /var/opt/mssql -name "*.ldf"
```

Expected:

```
/var/opt/mssql/data/LinuxDemo_log.ldf
```

## Tail the Error Log

``` bash
tail -f /var/opt/mssql/log/errorlog
```

Expected: live log lines as you run queries or restart the service.
Press `Ctrl+C` to stop.

## Read the Configuration File

``` bash
cat /var/opt/mssql/mssql.conf
```

Expected: an `.ini`-style file with sections like `[sqlagent]`,
`[EULA]`, and `[filelocation]`.

## Cross-check with SQL Server

``` sql
SELECT name,physical_name FROM sys.master_files ORDER BY database_id;
GO
```

Expected: the same `.mdf` / `.ldf` paths you found with `find`.

------------------------------------------------------------------------

## Challenge

Find the exact size in MB of `LinuxDemo.mdf` and `LinuxDemo_log.ldf`
using a single `du` command, and compare it to the size reported by
`sys.database_files` in T-SQL.
