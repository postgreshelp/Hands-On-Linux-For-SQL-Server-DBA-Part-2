# SQL Server on Linux (Oracle Linux 9 / RHEL 9)
> 🌐 **labs.postgreshelp.com** | Hands-on Database Engineering Labs

## Learning Objectives

By the end of this chapter, you will be able to:

- Install SQL Server 2022 on Oracle Linux / RHEL 9
- Verify the SQL Server service is running
- Open the firewall for SQL Server connections
- Install `sqlcmd` and connect to the instance

------------------------------------------------------------------------

## Install Repository

``` bash
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/9/mssql-server-2022.repo
```

## Install

``` bash
sudo dnf install -y mssql-server
sudo /opt/mssql/bin/mssql-conf setup
sudo systemctl enable --now mssql-server
```

During `mssql-conf setup` you will be prompted for an edition and an SA
password. Use `Password123` for this lab.

## Verify

``` bash
systemctl status mssql-server --no-pager
```

Expected:

```
Active: active (running)
```

## Open Firewall

``` bash
sudo firewall-cmd --permanent --add-port=1433/tcp
sudo firewall-cmd --reload
ss -lntp | grep 1433
```

Expected:

```
LISTEN 0  511  0.0.0.0:1433  0.0.0.0:*  users:(("sqlservr",pid=...))
```

## Install sqlcmd

``` bash
sudo curl -o /etc/yum.repos.d/msprod.repo \
https://packages.microsoft.com/config/rhel/9/prod.repo
```

## Install

``` bash
dnf install epel-release -y
sudo dnf install -y mssql-tools18 unixODBC-devel
```

## Connect

``` bash
/opt/mssql-tools18/bin/sqlcmd \
-S localhost \
-U sa \
-C
```

Expected:

```
Password:
1>
```

------------------------------------------------------------------------

## Challenge

Change the SA password using `mssql-conf set-sa-password`, restart the
service, and reconnect with `sqlcmd` using the new password.
