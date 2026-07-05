# SQL Server on Linux vs PostgreSQL - Side-by-Side Comparison

## Learning Objectives

By the end of this chapter, you will be able to:

- Translate common SQL Server administration terms, paths, and
  commands into their PostgreSQL equivalents
- Use this sheet as a quick reference during live troubleshooting or
  migration work

This chapter is a reference, not a hands-on lab — there are no
commands to run here. Keep it open in a separate tab during the other
labs.

## Installation & Setup

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| Microsoft Repository | PGDG Repository | Software repository used to install the database packages. |
| `mssql-server-2022.repo` | `pgdg-redhat-repo` | Repository configuration file added to the OS. |
| `dnf install mssql-server` | `dnf install postgresql18-server` | Installs the database server binaries. |
| `mssql-conf setup` | `postgresql-18-setup initdb` | Initializes and configures the database instance for first use. |
| `systemctl start mssql-server` | `systemctl start postgresql-18` | Starts the database service. |
| `systemctl status mssql-server` | `systemctl status postgresql-18` | Displays the current service status. |
| TCP Port **1433** | TCP Port **5432** | Default network port used for client connections. |
| `mssql-tools18` | `psql` (included with PostgreSQL) | Command-line client used to connect and execute SQL commands. |
| Microsoft ODBC Driver | libpq (Native PostgreSQL Client Library) | Client libraries used by applications to connect to the database. |

---

## Default Databases

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| master | postgres | Default administrative database used for maintenance tasks. |
| model | template1 | Template database used when creating new databases. |
| Resource Database (Hidden) | template0 | Read-only template used to create a clean database. |
| msdb | N/A | Stores SQL Server Agent jobs, backup history, alerts, etc. PostgreSQL has no equivalent built-in database. |
| tempdb | Temporary Objects | SQL Server has a dedicated tempdb; PostgreSQL stores temporary objects inside the current database. |

---

## Schemas

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| dbo | public | Default schema for user objects. |
| Custom Schemas | Custom Schemas | Logical containers used to organize database objects. |

---

## Physical Storage

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| MDF | `base/` | Stores actual user table and index data. |
| NDF | Tablespaces (`pg_tblspc`) | Additional data storage locations. |
| LDF | `pg_wal/` | Stores transaction logs used for recovery. |
| Error Log | PostgreSQL Log Files | Stores startup messages, errors, and warnings. |
| `mssql.conf` | `postgresql.conf` | Main database configuration file. |
| `/var/opt/mssql/data` | `/var/lib/pgsql/<version>/data` | Default data directory. |
| `/opt/mssql` | `/usr/pgsql-18` | Installation directory containing database binaries. |

---

## SQL Objects

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| Database | Database | Highest-level logical container. |
| Schema | Schema | Groups related database objects. |
| Table | Table | Stores application data. |
| View | View | Virtual table based on a query. |
| Stored Procedure | Procedure / Function | Reusable server-side program. |
| Function | Function | Returns a value or result set. |
| Trigger | Trigger | Automatically executes when data changes. |
| Index | Index | Improves query performance. |

---

## Data Types

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| `INT IDENTITY(1,1)` | `GENERATED AS IDENTITY` / Sequence | Automatically generates unique numbers. |
| `VARCHAR` | `TEXT` / `VARCHAR` | Stores character data. |
| `NVARCHAR` | `TEXT` | Unicode text storage. |
| `DATETIME` | `TIMESTAMP WITH TIME ZONE` | Stores date and time values. |
| `DECIMAL` | `NUMERIC` | Stores exact numeric values. |
| `BIT` | `BOOLEAN` | Stores True/False values. |
| `UNIQUEIDENTIFIER` | `UUID` | Stores globally unique identifiers. |
| `VARBINARY(MAX)` | `BYTEA` | Stores binary data such as files or images. |

---

## Common Functions

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| `GETDATE()` | `CURRENT_TIMESTAMP` / `now()` | Returns the current date and time. |
| `DB_NAME()` | `current_database()` | Returns the current database name. |
| `ISNULL()` | `COALESCE()` | Returns an alternate value when NULL. |
| `LEN()` | `LENGTH()` | Returns the length of a string. |
| `NEWID()` | `gen_random_uuid()` | Generates a UUID. |
| `SUSER_NAME()` | `current_user` | Returns the current login/user. |

---

## Backup & Recovery

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| `BACKUP DATABASE` | `pg_dump` | Creates a logical backup of the database. |
| `RESTORE DATABASE` | `pg_restore` | Restores a logical backup. |
| Differential Backup | N/A | SQL Server supports differential backups; PostgreSQL uses WAL-based backup strategies instead. |
| Transaction Log Backup | WAL Archiving | Enables point-in-time recovery. |
| Full Backup | Full Backup | Creates a complete backup of the database. |

---

## Transaction Logging

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| LDF | WAL | Records every database modification before it reaches the data files. |
| Write-Ahead Logging | Write-Ahead Logging | Ensures durability and crash recovery. |
| Crash Recovery | Crash Recovery | Replays logs after an unexpected shutdown. |

---

## Administration

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| SQL Server Agent | pg_cron / cron | Schedules recurring database jobs. |
| SSMS | pgAdmin | Graphical administration tool. |
| sqlcmd | psql | Command-line interface. |
| SQL Profiler | pg_stat_statements / auto_explain | Performance analysis tools. |
| Activity Monitor | pg_stat_activity | Displays currently running sessions. |

---

## System Catalogs

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| `sys.tables` | `pg_tables` | Lists all tables. |
| `sys.schemas` | `pg_namespace` | Lists schemas. |
| `sys.databases` | `pg_database` | Lists databases. |
| `sys.objects` | `pg_class` | Lists database objects. |
| `INFORMATION_SCHEMA` | `INFORMATION_SCHEMA` | ANSI-standard metadata views. |

---

## SQL Syntax

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| `GO` | Not Required | `GO` is recognized by client tools like SSMS/sqlcmd, not by the database engine. |
| `USE LinuxDemo` | `\c linuxdemo` | Switches the current database connection. |
| `EXEC dbo.GetOrders` | `CALL dbo.GetOrders(...)` | Executes a stored procedure. |
| `SELECT * FROM Function()` | `SELECT * FROM Function()` | Executes a table-returning function. |

---

## Migration

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| T-SQL | PL/pgSQL | Procedural language used for stored programs. |
| SQL Server Agent Jobs | pg_cron Jobs | Database job scheduling. |
| pgloader Source | pgloader Target | pgloader migrates schema and data between the two databases. |
| Stored Procedures | Manual Conversion | pgloader does not convert procedural code automatically. |
| Triggers | Manual Conversion | Triggers must be recreated manually after migration. |
| CLR Objects | No Equivalent | .NET CLR objects require application redesign or replacement. |

---

## Challenge

Without scrolling back up, write the PostgreSQL equivalents of:
`sys.tables`, `GETDATE()`, and `SSMS`. Then check your answers
against the tables above.
