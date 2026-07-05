# Schemas and Migration

## Learning Objectives

By the end of this chapter, you will be able to:

- List all schemas and tables in a SQL Server database
- Install and use `pgloader` to migrate SQL Server to PostgreSQL
- Verify a migration by comparing objects on both sides

------------------------------------------------------------------------

## Install pgloader

``` bash
dnf install freetds freetds-libs
dnf install pgloader
```

Expected:

```
Installed:
  pgloader-x.x.x
```

## List Schemas and Tables (SQL Server side)

``` sql
SELECT s.name AS SchemaName,t.name AS TableName
FROM sys.tables t JOIN sys.schemas s ON t.schema_id=s.schema_id
ORDER BY 1,2;
GO
```

Expected:

```
dbo    Orders
sales  Customers
```

## Migrate with pgloader

``` bash
pgloader mssql://sa:Password123@localhost/LinuxDemo postgresql:///linuxdemo

(or, specifying both ends explicitly)

pgloader \
mssql://sa:Password123@localhost/LinuxDemo \
postgresql://postgres:postgres@localhost/linuxdemo
```

Expected: a summary table at the end showing rows read, rows
imported, and any warnings/errors per table.

## Verify on the PostgreSQL side

``` sql
\dt *.*
SELECT table_schema,table_name FROM information_schema.tables WHERE table_schema NOT IN ('pg_catalog','information_schema');
```

Expected:

```
public  orders
sales   customers
```

------------------------------------------------------------------------

## Challenge

Add a new table in SQL Server with a `DATETIME` and a `BIT` column,
re-run pgloader, and confirm how those two data types were converted
on the PostgreSQL side.
