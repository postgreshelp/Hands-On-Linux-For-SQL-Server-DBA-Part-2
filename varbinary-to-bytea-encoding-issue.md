# SQL Server → PostgreSQL Migration: VARBINARY/BYTEA Encoding Issue

## Problem Statement

After migrating a column from SQL Server `VARBINARY` to PostgreSQL `BYTEA`, the column's
values display as raw binary/hex instead of readable text — both directly in Postgres
queries and when read by the .NET application.

## Root Cause

The original SQL Server column was `NVARCHAR` (not `VARCHAR`). SQL Server stores
`NVARCHAR` data internally as **UTF-16LE**. When that column was cast to `VARBINARY`
and migrated to Postgres as raw bytes, the UTF-16LE bytes were copied over **as-is** —
with no conversion to UTF-8.

PostgreSQL's `BYTEA` column has no idea what encoding the bytes represent — it just
stores them. So any attempt to read them as UTF-8 (the default assumption) fails or
produces garbled/spaced-out output, because every character is padded with a `0x00`
byte (the hallmark of UTF-16LE for ASCII-range text).

**Important finding:** PostgreSQL has **no native UTF-16 support** in `convert_from`/
`convert_to` — its encoding list covers UTF8, LATIN1, WIN1252, SJIS, etc., but not
UTF16LE. This means the UTF-16 → UTF-8 conversion **cannot be done in pure SQL** and
must happen in application code (or a one-off script).

## Diagnostic Steps (Reproduced in Lab)

### 1. Recreated the source data in SQL Server
Two rows were created to compare a `VARCHAR` source vs an `NVARCHAR` source:

```sql
CREATE TABLE dbo.TestVarbinary (
    id INT IDENTITY PRIMARY KEY,
    data_col VARBINARY(MAX)
);

INSERT INTO dbo.TestVarbinary (data_col)
VALUES (CONVERT(VARBINARY(MAX), CAST('Hello Postgres Migration' AS VARCHAR(MAX))));

INSERT INTO dbo.TestVarbinary (data_col)
VALUES (CONVERT(VARBINARY(MAX), CAST(N'Hello Postgres Migration' AS NVARCHAR(MAX))));
```

### 2. Extracted the bytes as base64 (to avoid sqlcmd column truncation)
```sql
SELECT id, data_col FROM dbo.TestVarbinary FOR XML PATH('');
```
Result:
- Row 1 (`VARCHAR` source): `SGVsbG8gUG9zdGdyZXMgTWlncmF0aW9u`
- Row 2 (`NVARCHAR` source): `SABlAGwAbABvACAAUABvAHMAdABnAHIAZQBzACAATQBpAGcAcgBhAHQAaQBvAG4A`

### 3. Loaded into PostgreSQL and inspected raw bytes
```sql
CREATE TABLE test_bytea (
    id SERIAL PRIMARY KEY,
    label TEXT,
    data_col BYTEA
);

INSERT INTO test_bytea (label, data_col) VALUES
('varchar_source', decode('SGVsbG8gUG9zdGdyZXMgTWlncmF0aW9u', 'base64')),
('nvarchar_source', decode('SABlAGwAbABvACAAUABvAHMAdABnAHIAZQBzACAATQBpAGcAcgBhAHQAaQBvAG4A', 'base64'));
```

Raw byte comparison confirmed the theory:
- `varchar_source`: `\x48656c6c6f20506f737467726573204d6967726174696f6e` — no padding
- `nvarchar_source`: `\x480065006c006c006f00200050006f00...` — `00` after every character (UTF-16LE signature)

### 4. Confirmed the failure mode
```sql
SELECT id, label, convert_from(data_col, 'UTF8') FROM test_bytea;
-- ERROR: invalid byte sequence for encoding "UTF8": 0x00
```

Attempting to decode as UTF-16LE directly in Postgres also failed, confirming Postgres
has no built-in support for it:
```sql
SELECT id, label, convert_from(data_col, 'UTF16LE') FROM test_bytea;
-- ERROR: invalid source encoding name "UTF16LE"
```

## Two Fix Options (Both Tested)

### Option A — Fix at the data level (one-time Python script)
Re-encode the existing bad rows from UTF-16LE to UTF-8, so every future consumer
(any app, any report, plain SQL) reads the column correctly without special handling.

```python
import psycopg2

conn = psycopg2.connect(dbname="postgres", user="postgres", host="localhost")
cur = conn.cursor()

cur.execute("SELECT id, data_col FROM fix_python")
rows = cur.fetchall()

for row_id, raw_bytes in rows:
    text = bytes(raw_bytes).decode('utf-16-le')
    utf8_bytes = text.encode('utf-8')
    cur.execute("UPDATE fix_python SET data_col = %s WHERE id = %s", (utf8_bytes, row_id))

conn.commit()
print(f"Fixed {len(rows)} rows in fix_python")
```

**Result after running:**
```sql
SELECT id, convert_from(data_col, 'UTF8') FROM fix_python;
--  id |       convert_from
-- ----+--------------------------
--   1 | Hello Postgres Migration
```
Works cleanly — no more UTF8 decode errors.

### Option B — Fix at the code level (.NET read-time decode)
Leave the stored bytes untouched; decode correctly wherever the app reads them.

```csharp
using Npgsql;
using System.Text;

var connString = "Host=localhost;Username=postgres;Database=postgres";

await using var conn = new NpgsqlConnection(connString);
await conn.OpenAsync();

await using var cmd = new NpgsqlCommand("SELECT id, data_col FROM fix_codelevel", conn);
await using var reader = await cmd.ExecuteReaderAsync();

while (await reader.ReadAsync())
{
    byte[] binaryData = (byte[])reader["data_col"];
    string readableText = Encoding.Unicode.GetString(binaryData); // Unicode = UTF-16LE in .NET
    Console.WriteLine($"id={reader["id"]}: {readableText}");
}
```

**Result:**
```
id=1: Hello Postgres Migration
```
Works cleanly — the app now decodes with the correct encoding.

## Recommendation

| Option | Pros | Cons |
|---|---|---|
| **A — Re-encode data in Postgres (Python/script)** | Fixes it for *every* consumer (reports, other apps, raw SQL) permanently. One-time cost. | Requires a migration script + careful rollout if the table is large or live. |
| **B — Decode at read time in .NET** | Fast, no data changes, low risk. | Only patches this one app. Every other consumer (SQL reports, other services) still breaks. Must be applied everywhere the column is read. |

**Suggested path:** Use Option B as an immediate patch to unblock the .NET application,
then schedule Option A (or an equivalent proper re-migration with UTF-8 re-encoding) as
the permanent fix so the data is correct for all consumers going forward.

## Key Gotcha to Flag to the Team

Confirm whether the original SQL Server column was `VARCHAR`/`CHAR` (single-byte) or
`NVARCHAR`/`NCHAR` (UTF-16) **before** assuming all rows need the same fix. If the table
has a mix of sources, a blanket UTF-16 decode will break the already-correct rows —
each row (or each known source system) may need to be handled according to its actual
original encoding.

## Environment Notes (Lab Setup)

- SQL Server accessed via `sqlcmd`/PowerShell on Windows.
- PostgreSQL 19beta3 running on a separate Linux (RHEL/CentOS-family) box, accessed via
  `/usr/local/pgsql/bin/psql`.
- .NET 8 SDK installed on Linux via `dotnet-install.sh`; required manually adding
  `~/.dotnet` to `PATH` and `DOTNET_ROOT` since the installer only sets it for the
  invoking shell session.
- `dotnet run` requires an actual project (`dotnet new console` + `.csproj`) — it will
  not run a loose `.cs` file directly.
- Python fix used `psycopg2-binary`, installed via `pip install psycopg2-binary`.
