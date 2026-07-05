# Chapter 12 - Shell Scripting for DBAs

## Learning Objectives

By the end of this chapter, you will be able to:

- Write and run a basic Bash script
- Use a shell script to query SQL Server with `sqlcmd`
- Use a shell script to insert data into SQL Server

------------------------------------------------------------------------

# Script 1 - Add Two Numbers

Create a file.

```bash
vi add.sh
```

```bash
#!/bin/bash

num1=10
num2=20

sum=$((num1 + num2))

echo "Number 1 : $num1"
echo "Number 2 : $num2"
echo "Sum      : $sum"
```

Make it executable.

```bash
chmod +x add.sh
```

Run

```bash
./add.sh
```

Expected / Output

```
Number 1 : 10
Number 2 : 20
Sum      : 30
```

---

### Code Breakdown

| Code | Explanation |
|------|-------------|
| `#!/bin/bash` | Executes the script using Bash. |
| `num1=10` | Assigns a value to a variable. |
| `$(( ))` | Performs arithmetic operations. |
| `echo` | Prints output to the terminal. |
| `chmod +x` | Makes the script executable. |

---

# Script 2 - Fetch Records from SQL Server

Create

```bash
vi fetch_orders.sh
```

```bash
#!/bin/bash

SERVER="localhost"
DATABASE="LinuxDemo"
USER="sa"
PASSWORD="Password123"

SQLCMD="/opt/mssql-tools18/bin/sqlcmd"

$SQLCMD \
-S $SERVER \
-U $USER \
-P $PASSWORD \
-C \
-d $DATABASE \
-Q "SELECT * FROM dbo.Orders"
```

Run

```bash
chmod +x fetch_orders.sh

./fetch_orders.sh
```

Expected:

```
OrderID  CustomerName  OrderDate            Amount
-------  ------------  -------------------  -------
1        Amazon        2026-07-05 10:00:00  1200.00
...
```

---

### Code Breakdown

| Code | Explanation |
|------|-------------|
| `SERVER=` | SQL Server hostname. |
| `DATABASE=` | Database name. |
| `USER=` | SQL Server login. |
| `PASSWORD=` | Password stored in a variable. |
| `sqlcmd` | SQL Server command-line utility. |
| `-S` | SQL Server hostname. |
| `-U` | Login user. |
| `-P` | Password. |
| `-d` | Database name. |
| `-Q` | Executes a SQL query and exits. |
| `-C` | Trust the SQL Server certificate. |

---

# Script 3 - Insert Data into SQL Server

Create

```bash
vi insert_order.sh
```

```bash
#!/bin/bash

SERVER="localhost"
DATABASE="LinuxDemo"
USER="sa"
PASSWORD="Password123"

CUSTOMER="Linux Script"
AMOUNT=7500

SQLCMD="/opt/mssql-tools18/bin/sqlcmd"

$SQLCMD \
-S $SERVER \
-U $USER \
-P $PASSWORD \
-C \
-d $DATABASE \
-Q "
INSERT INTO dbo.Orders
(
    CustomerName,
    Amount
)
VALUES
(
    '$CUSTOMER',
    $AMOUNT
);
"

echo "1 row inserted successfully."
```

Run

```bash
chmod +x insert_order.sh

./insert_order.sh
```

Expected:

```
1 row inserted successfully.
```

---

### Verify

```bash
./fetch_orders.sh
```

or

```sql
SELECT COUNT(*)
FROM dbo.Orders;
```

---

### Code Breakdown

| Code | Explanation |
|------|-------------|
| `CUSTOMER=` | Stores the customer name in a shell variable. |
| `AMOUNT=` | Stores the amount in a shell variable. |
| `INSERT INTO` | Inserts a new record. |
| `echo` | Displays a success message. |

---

## Challenge

Write a shell script, `count_orders.sh`, that connects with `sqlcmd`
and prints only the total number of rows in `dbo.Orders`.

------------------------------------------------------------------------

# Summary

| Script | Purpose |
|---------|---------|
| `add.sh` | Learn variables and arithmetic. |
| `fetch_orders.sh` | Read data from SQL Server. |
| `insert_order.sh` | Insert data into SQL Server using Shell Script. |
