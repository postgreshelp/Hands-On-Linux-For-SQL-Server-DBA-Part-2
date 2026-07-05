# Linux Health Checks
> 🌐 **labs.postgreshelp.com** | Hands-on Database Engineering Labs

## Learning Objectives

By the end of this chapter, you will be able to:

- Check whether the SQL Server service is healthy from the OS side
- Identify SQL Server's process and threads
- Confirm SQL Server is listening on its port
- Check disk, memory, and CPU usage relevant to SQL Server

------------------------------------------------------------------------

## Service and Process Checks

``` bash
systemctl status mssql-server --no-pager
```

Expected:

```
Active: active (running)
```

``` bash
journalctl -u mssql-server
```

Expected: startup log lines ending with something like
`SQL Server is now ready for client connections`.

``` bash
ps -ef | grep sqlservr
```

Expected: a running `sqlservr` process owned by user `mssql`.

``` bash
pstree -p | grep sqlservr
```

Expected: the `sqlservr` process tree with its worker threads.

``` bash
ps -T -p <PID>
```

Expected: a list of threads belonging to the SQL Server PID.

## Network Check

``` bash
ss -lntp | grep 1433
```

Expected:

```
LISTEN 0  511  0.0.0.0:1433  0.0.0.0:*  users:(("sqlservr",pid=...))
```

## Resource Checks

``` bash
df -h
du -sh /var/opt/mssql
free -h
top
```

Expected: enough free disk space under `/var/opt/mssql`, and
`sqlservr` visible near the top of `top`'s CPU/memory usage.

------------------------------------------------------------------------

## Challenge

Write a one-line shell command that finds the SQL Server PID
automatically (without you typing it in) and passes it straight into
`ps -T -p`.
