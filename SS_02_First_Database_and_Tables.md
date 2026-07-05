# First Database and Tables

## Learning Objectives

By the end of this chapter, you will be able to:

- Create a database, schema, and tables in SQL Server
- Insert sample data
- Verify tables and schemas using system catalog views
- Locate the physical data files behind a database

------------------------------------------------------------------------

## Load Data

``` SQL
CREATE DATABASE LinuxDemo;
GO

USE LinuxDemo;
GO

CREATE SCHEMA sales;
GO

CREATE TABLE dbo.Orders
(
    OrderID INT IDENTITY(1,1) PRIMARY KEY,
    CustomerName VARCHAR(100) NOT NULL,
    OrderDate DATETIME DEFAULT GETDATE(),
    Amount DECIMAL(10,2) NOT NULL
);
GO

INSERT INTO dbo.Orders(CustomerName,Amount)
VALUES
('Amazon',1200),
('Google',2500),
('Microsoft',3000),
('Oracle',1700),
('IBM',900);
GO

CREATE TABLE sales.Customers
(
    CustomerID INT PRIMARY KEY,
    Name VARCHAR(100) NOT NULL
);
GO

INSERT INTO sales.Customers
VALUES
(1,'Alice'),
(2,'Bob');
GO
```

Expected:

```
(5 rows affected)
(2 rows affected)
```

------------------------------------------------------------------------

## Verification

``` SQL
SELECT DB_NAME();
GO

SELECT
s.name,
t.name
FROM sys.tables t
JOIN sys.schemas s
ON t.schema_id=s.schema_id;
GO
```

Expected:

```
LinuxDemo

dbo    Orders
sales  Customers
```

## Locate physical files

``` SQL
SELECT
name,
physical_name
FROM sys.master_files;
GO
```

Expected: rows showing `LinuxDemo` and `LinuxDemo_log` pointing to
`.mdf` and `.ldf` files under `/var/opt/mssql/data/`.

------------------------------------------------------------------------

## Challenge

Create a third table, `sales.Products`, with columns `ProductID`,
`ProductName`, and `Price`. Insert three sample rows and confirm it
appears under the `sales` schema using the verification query above.
