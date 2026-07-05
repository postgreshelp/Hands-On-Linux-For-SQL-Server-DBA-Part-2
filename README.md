# Hands-On Linux for SQL Server DBAs — Part 2

> **Install • Administer • Automate • Migrate**
>
> A practical hands-on workshop for SQL Server DBAs transitioning to Linux and PostgreSQL.

Day 2 of the workshop series.

While **Part 1** focuses on Linux fundamentals, **Part 2** takes you through deploying a real SQL Server instance on Linux and performing day-to-day DBA tasks including installation, administration, backup & restore, automation, and migration to PostgreSQL.

---

## What You'll Learn

By the end of this workshop, you'll be able to:

- Install SQL Server 2022 on Oracle Linux
- Connect using `sqlcmd`
- Create databases, schemas, tables and stored procedures
- Perform SQL Server administration tasks
- Configure SQL Server on Linux
- Perform online Backup & Restore
- Explore SQL Server data files, transaction log files and error logs
- Compare SQL Server and PostgreSQL internals side by side
- Migrate SQL Server to PostgreSQL using `pgloader`
- Automate SQL Server using Shell Scripts and Python

---

## Prerequisites

- Oracle Linux 9 / RHEL 9 (or compatible)
- `sudo` access
- Basic Linux knowledge (Part 1 recommended)
- Internet connectivity
- Approximately **4–6 hours** to complete all labs

---

## How to Use This Repository

Complete the chapters **in numerical order**.

Each chapter follows a consistent structure:

- 🎯 Learning Objectives
- 📝 Step-by-step Hands-on Exercises
- ✅ Expected Output
- 💡 Explanation of every command
- 🎯 Challenge Exercise

---

## Chapter Index

| File | Topic | Description |
|------|-------|-------------|
| `SS_01_SQL_Server_on_Linux_Installation.md` | SQL Server Installation | Install SQL Server 2022, configure the firewall, install `sqlcmd`, and connect to the server |
| `SS_02_First_Database_and_Tables.md` | Database Fundamentals | Create databases, schemas, tables and verify using system catalog views |
| `SS_03_Linux_Health_Checks.md` | Linux Monitoring | Verify SQL Server services, processes, network ports and resource utilization |
| `SS_04_SQL_Server_Files.md` | SQL Server Internals | Explore MDF, LDF, error logs and `mssql.conf` |
| `SS_05_Schemas_and_Migration.md` | SQL Server → PostgreSQL | Install `pgloader` and migrate a SQL Server database to PostgreSQL |
| `SS_06_SQL_Server_vs_PostgreSQL.md` | Architecture Comparison | Compare SQL Server and PostgreSQL commands, files, configuration and architecture |
| `SS_07_SQL_Server_Config_Backup_Restore.md` | Administration | Configure SQL Server, perform online Backup & Restore |
| `SS_08_CheatSheet.md` | Cheat Sheet | Complete SQL Server ↔ PostgreSQL comparison reference |
| `SS_09_Scripting.md` | Python Setup | Install Python, ODBC Driver, `pyodbc` and `psycopg2` |
| `SS_10_Automation_With_Python.md` | Python Automation | Connect to SQL Server & PostgreSQL using Python and automate common DBA tasks |
| `SS_11_Shell_For_SQL_Server.md` | Shell Automation | Automate SQL Server administration using Bash Shell Scripts |
| `SS_12_Next_Steps.md` | Next Steps | Workshop recap and transition to Advanced PostgreSQL Administration |

---

## Lab Environment

The following lab configuration is used throughout the workshop.

### SQL Server

| Parameter | Value |
|-----------|-------|
| Host | localhost |
| Port | 1433 |
| User | sa |
| Password | Password123 |
| Database | LinuxDemo |

### PostgreSQL

| Parameter | Value |
|-----------|-------|
| Host | localhost |
| Port | 5432 |
| PgBouncer Port | 6432 |
| Database | linuxdemo |
| User | postgres |

### Backup Directory

```text
/backup/sqlserver
```

> **Note:** These credentials are provided for demonstration purposes only. Never use hard-coded credentials in production environments.

---

## Learning Outcome

After completing this workshop, you will understand:

- Linux fundamentals for DBAs
- SQL Server administration on Linux
- SQL Server storage architecture
- Backup & Restore
- SQL Server vs PostgreSQL architecture
- SQL Server → PostgreSQL migration
- Database automation using Shell Scripts
- Database automation using Python

---

## Continue Your Learning

The next workshop in this series is:

**Advanced PostgreSQL Administration**

See:

```
SS_12_Next_Steps.md
```

---

⭐ **If this repository helped you, please consider giving it a GitHub Star.**

Your support motivates me to continue creating free, hands-on database engineering content for the community.
