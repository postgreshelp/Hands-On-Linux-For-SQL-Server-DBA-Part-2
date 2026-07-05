# Schemas and Migration
> 🌐 **labs.postgreshelp.com** | Hands-on Database Engineering Labs

## Learning Objectives

By the end of this chapter, you will be able to:

- List all schemas and tables in a SQL Server database
- Install and use `pgloader` to migrate SQL Server to PostgreSQL
- Verify a migration by comparing objects on both sides
- Migrate stored procedures using SQLines, since `pgloader` does not convert them

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

## Migrate Stored Procedures (SQLines)

`pgloader` only migrates schemas, tables, and data — it does not convert
stored procedures, functions, or triggers. For those, use the
[SQLines SQL Server to PostgreSQL converter](https://www.sqlines.com/sql-server-to-postgresql)
instead.

The general workflow:

1. Copy the SQL Server procedure definition into the SQLines converter.
2. Review the generated PL/pgSQL output — SQLines gets most of the
   syntax right, but result sets returned via implicit `SELECT`
   statements need to become an explicit `REFCURSOR` output parameter,
   since PostgreSQL procedures don't return result sets the way SQL
   Server ones do.
3. Apply the converted procedure on the PostgreSQL side with `psql`.
4. Call the procedure and fetch from the cursor to confirm it works.

### SQL Server source

``` sql
CREATE OR ALTER PROCEDURE dbo.GetOrders
AS
BEGIN
    SET NOCOUNT ON;
    SELECT *
    FROM dbo.Orders
    ORDER BY OrderID;
END;
GO
EXEC dbo.GetOrders;
```

### PostgreSQL target (converted with SQLines)

``` sql
CREATE OR REPLACE PROCEDURE GetOrders (IN OUT cur REFCURSOR)
AS $$
BEGIN
    OPEN cur FOR SELECT *
    FROM Orders
    ORDER BY OrderID;
END;
$$ LANGUAGE plpgsql;
```

### Call it and fetch the results

Because the procedure returns a cursor instead of a result set directly,
it must be called inside a transaction, then read with `FETCH`:

``` sql
linuxdemo=# BEGIN;
BEGIN
linuxdemo=*# CALL dbo.GetOrders('mycursor');
   cur
----------
 mycursor
(1 row)
linuxdemo=*# FETCH ALL FROM mycursor;
```

> **Production Gotcha ⚠️** Forgetting the `BEGIN` before `CALL` is the
> most common mistake here — a cursor-returning procedure only stays
> open for the life of the transaction that opened it. Call it outside
> a transaction and the cursor closes before you can `FETCH` from it.

------------------------------------------------------------------------

## Challenge

Add a new table in SQL Server with a `DATETIME` and a `BIT` column,
re-run pgloader, and confirm how those two data types were converted
on the PostgreSQL side.

Then write a second SQL Server stored procedure that accepts an input
parameter (for example, filtering `Orders` by `CustomerID`), convert it
with SQLines, and confirm the PostgreSQL version returns the correct
filtered cursor.
