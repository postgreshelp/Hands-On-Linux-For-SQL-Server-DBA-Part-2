# Python Setup for SQL Server & PostgreSQL

## Learning Objectives

By the end of this chapter, you will be able to:

- Install Python 3 and pip on Linux
- Install the ODBC driver stack needed to connect Python to SQL Server
- Install and verify the `pyodbc` and `psycopg2` drivers

------------------------------------------------------------------------

## Install Python

```bash
sudo dnf install -y python3 python3-pip python3-devel gcc gcc-c++
```

Verify

```bash
python3 --version
pip3 --version
```

Expected:

```
Python 3.x.x
pip 2x.x.x from ...
```

---

## SQL Server Driver (ODBC)

```bash
sudo curl -o /etc/yum.repos.d/msprod.repo \
https://packages.microsoft.com/config/rhel/9/prod.repo

sudo ACCEPT_EULA=Y dnf install -y \
msodbcsql18 \
mssql-tools18 \
unixODBC-devel
```

Verify

```bash
odbcinst -q -d
```

Expected

```
[ODBC Driver 18 for SQL Server]
```

---

## PostgreSQL Driver

```bash
pip3 install psycopg2-binary
```

Verify

```bash
python3 -c "import psycopg2; print(psycopg2.__version__)"
```

Expected:

```
2.9.x (dt dec pq3 ext lo64)
```

---

## SQL Server Python Driver

```bash
pip3 install pyodbc
```

Verify

```bash
python3 -c "import pyodbc; print(pyodbc.version)"
```

Expected:

```
5.x.x
```

---

## Installed Python Packages

```bash
pip3 list
```

Expected

```
pyodbc
psycopg2-binary
```

---

# Test SQL Server

```python
import pyodbc

conn = pyodbc.connect(
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=localhost;"
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
python3 test_sqlserver.py
```

Expected:

```
Connected to SQL Server
```

---

# Test PostgreSQL

```python
import psycopg2

conn = psycopg2.connect(
    host="localhost",
    database="linuxdemo",
    user="postgres"
)

print("Connected to PostgreSQL")

conn.close()
```

Run

```bash
python3 test_postgres.py
```

Expected:

```
Connected to PostgreSQL
```

---

# Python Drivers Comparison

| SQL Server | PostgreSQL |
|------------|------------|
| pyodbc | psycopg2 |
| ODBC Driver 18 | libpq |
| Driver={ODBC Driver 18 for SQL Server} | Native PostgreSQL Client |
| Port 1433 | Port 5432 |

------------------------------------------------------------------------

## Challenge

Write a single Python script that connects to **both** SQL Server and
PostgreSQL, prints a success message for each, and closes both
connections cleanly.
