# Chapter 13 - Database Automation with Python

## Learning Objectives

By the end of this chapter, you will be able to:

- Connect to SQL Server and PostgreSQL from Python
- Read server metadata and insert data programmatically
- Connect to PostgreSQL through pgbouncer and observe connection
  pooling in action

------------------------------------------------------------------------

# SQL Server Automation

## Step 1 - Connect to SQL Server

```python
import pyodbc

conn = pyodbc.connect(
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=192.168.108.152;"
    "DATABASE=LinuxDemo;"
    "UID=sa;"
    "PWD=Password123;"
    "TrustServerCertificate=yes;"
)

print("Connected to SQL Server")

conn.close()
```

Run

```bash
python3 connect_sqlserver.py
```

Expected:

```
Connected to SQL Server
```

---

### Code Breakdown

| Code | Purpose |
|------|---------|
| `import pyodbc` | Imports the SQL Server Python driver. |
| `pyodbc.connect()` | Creates a connection to SQL Server. |
| `SERVER` | SQL Server hostname or IP. |
| `DATABASE` | Database to connect to. |
| `UID` | Login user. |
| `PWD` | Password. |
| `TrustServerCertificate=yes` | Accepts the server certificate without validation. |
| `conn.close()` | Closes the connection. |

---

## Step 2 - Read SQL Server Information

```python
import pyodbc

conn = pyodbc.connect(
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=192.168.108.152;"
    "DATABASE=LinuxDemo;"
    "UID=sa;"
    "PWD=Password123;"
    "TrustServerCertificate=yes;"
)

cursor = conn.cursor()

cursor.execute("""
SELECT
    @@SERVERNAME,
    DB_NAME(),
    @@VERSION;
""")

row = cursor.fetchone()

print(f"Server   : {row[0]}")
print(f"Database : {row[1]}")
print(f"Version  : {row[2]}")

cursor.close()
conn.close()
```

Expected:

```
Server   : ...
Database : LinuxDemo
Version  : Microsoft SQL Server 2022 ...
```

---

### Code Breakdown

| Code | Purpose |
|------|---------|
| `cursor()` | Creates a SQL cursor. |
| `execute()` | Executes SQL. |
| `fetchone()` | Reads one row. |
| `@@SERVERNAME` | SQL Server instance name. |
| `DB_NAME()` | Current database. |
| `@@VERSION` | SQL Server version. |

---

## Step 3 - Insert Data

```python
import pyodbc

conn = pyodbc.connect(
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=192.168.108.152;"
    "DATABASE=LinuxDemo;"
    "UID=sa;"
    "PWD=Password123;"
    "TrustServerCertificate=yes;"
)

cursor = conn.cursor()

cursor.execute("""
INSERT INTO dbo.Orders
(
    CustomerName,
    Amount
)
VALUES
(
    ?,
    ?
)
""", ("Python Demo",5000))

conn.commit()

print("1 row inserted.")

cursor.execute("""
SELECT COUNT(*)
FROM dbo.Orders
""")

print("Total Orders :", cursor.fetchone()[0])

cursor.close()
conn.close()
```

Expected:

```
1 row inserted.
Total Orders : 6
```

---

### Code Breakdown

| Code | Purpose |
|------|---------|
| `?` | Parameter placeholder used by pyodbc. |
| `commit()` | Saves the transaction permanently. |
| `COUNT(*)` | Verifies the inserted row. |

---

# PostgreSQL Automation (via pgbouncer)

## Step 1 - Connect to PostgreSQL through pgbouncer

pgbouncer listens on port `6432` and pools connections in front of
PostgreSQL's own port `5432`.

```python
import psycopg2

conn = psycopg2.connect(
    host="192.168.108.147",
    port=6432,
    database="postgres",
    user="postgres",
    password="postgres"
)

print("Connected to PostgreSQL")

conn.close()
```

Expected:

```
Connected to PostgreSQL
```

---

### Code Breakdown

| Code | Purpose |
|------|---------|
| `import psycopg2` | Imports PostgreSQL driver. |
| `connect()` | Opens PostgreSQL connection. |
| `host` | PostgreSQL server. |
| `port` | pgbouncer's listener port (6432), not PostgreSQL's own port. |
| `database` | Database name. |
| `user` | Login user. |
| `password` | Login password. |

---

## Step 2 - Observe pgbouncer Pooling

This script opens a handful of concurrent sessions through pgbouncer
and prints the backend PID each one is assigned. With connection
pooling enabled, you'll see PostgreSQL reuse a small number of backend
processes even though multiple client threads are running.

```python
import psycopg2
import threading

def run_session():

    conn = psycopg2.connect(
        host="192.168.108.147",
        port=6432,
        database="postgres",
        user="postgres",
        password="postgres"
    )

    cursor = conn.cursor()

    cursor.execute("SELECT pg_backend_pid()")
    pid = cursor.fetchone()

    print(pid[0])

    cursor.execute("SELECT pg_sleep(2)")

    cursor.close()
    conn.close()

threads = []

for _ in range(10):
    t = threading.Thread(target=run_session)
    threads.append(t)

for t in threads:
    t.start()

for t in threads:
    t.join()

print("All sessions completed.")
```

Expected: 10 backend PIDs printed, with far fewer *distinct* PIDs than
10 if pgbouncer is pooling connections in `transaction` mode, followed
by:

```
All sessions completed.
```

---

### Code Breakdown

| Code | Purpose |
|------|---------|
| `threading` | Runs multiple sessions concurrently. |
| `Thread()` | Creates one client session. |
| `start()` | Starts the thread. |
| `join()` | Waits for all threads to finish. |
| `pg_backend_pid()` | Returns PostgreSQL backend process ID. |
| `pg_sleep(2)` | Keeps the session alive for two seconds. |

---

# Python Drivers Comparison

| SQL Server | PostgreSQL | Explanation |
|------------|------------|-------------|
| `pyodbc` | `psycopg2` | Python driver used to connect to the database. |
| ODBC Driver 18 | Native libpq | Underlying client library. |
| `DRIVER={ODBC Driver 18 for SQL Server}` | Not Required | SQL Server uses ODBC; psycopg2 communicates directly with PostgreSQL. |
| `?` | `%s` | SQL parameter placeholders. |
| Port **1433** | Port **5432** (direct) / **6432** (pgbouncer) | Default listener ports. |
| `commit()` | `commit()` | Saves the transaction. |
| `cursor()` | `cursor()` | Executes SQL statements. |
| `fetchone()` | `fetchone()` | Reads one row. |
| `fetchall()` | `fetchall()` | Reads all rows. |

------------------------------------------------------------------------

## Challenge

Modify the SQL Server insert script to insert 5 records instead of 1,
using a loop, and print the running total after each insert.
