# Database and SQL Basics

**Path:** Pre-Security → Software Basics
**Room:** [Database SQL Basics](https://tryhackme.com/room/databasesqlbasics)
**Completed:** 2026-06-15

## What the room covered

What a database is, how data is organised into tables / rows / columns, and how SQL queries pull information back out. The hands-on used a Café SQL browser lab with two tables — `Orders(id, drink, price, time)` and `Menu(drink, price)` — to practise `SELECT`, `WHERE`, and `ORDER BY` against real data.

## Key concepts

**Database** — organised storage for data, structured into tables with rows and columns. Tables relate to each other via keys.

**Table structure:**
- Columns (fields) — categories of data (e.g., username, email, password_hash)
- Rows (records) — individual entries (one user's data)
- Primary key — unique identifier per row
- Foreign key — reference to another table's primary key

**SQL (Structured Query Language)** — language for talking to relational databases. Standard syntax across most database systems (MySQL, PostgreSQL, SQLite, MSSQL).

**Core SQL statements:**
- `SELECT column FROM table` — read data
- `WHERE condition` — filter results
- `ORDER BY column [DESC]` — sort results (ascending by default; `DESC` flips it)
- `INSERT INTO table VALUES (...)` — add data
- `UPDATE table SET column = value WHERE condition` — modify data
- `DELETE FROM table WHERE condition` — remove data

**Common database systems:**
- MySQL / MariaDB — most common in web apps
- PostgreSQL — feature-rich, used in many enterprise systems
- SQLite — lightweight, embedded in mobile apps and small tools
- MSSQL — Microsoft's enterprise database

## Mental model — why databases matter

Most modern systems are built around databases. Websites store user accounts, e-commerce stores products and orders, banks store transactions, social media stores posts and connections. Every login form, every search, every "my account" page is talking to a database in the background.

For cyber security: if you can read SQL, you can understand what attackers are trying to do when they attack databases — and what's at risk when a database is compromised. The user table is usually the highest-value target because it contains credentials.

## Offensive angle

| Concept | Attacker use |
| --- | --- |
| SELECT queries | Read data the attacker shouldn't see — user passwords, credit cards, private messages. The goal of most database attacks. |
| WHERE clauses | SQL injection target — manipulate the WHERE condition to bypass authentication (`WHERE username='admin' AND password='whatever' OR 1=1`) |
| UNION operator | Combine results from a legitimate query with an attacker's query to extract data from other tables |
| INSERT statements | Add attacker-controlled records — fake admin accounts, malicious content, persistence mechanisms |
| UPDATE statements | Modify existing data — change passwords, escalate privileges, alter financial records |
| DELETE statements | Destroy data for cover-up or sabotage |
| Information schema | Database metadata about itself — attackers query this to discover what tables/columns exist before exploiting |
| Dumped database files | Post-breach: attackers leak entire database contents online. Reading the SQL tells you what was exposed |

The classic SQL injection example: login form expects `SELECT * FROM users WHERE username='X' AND password='Y'`. Attacker enters `' OR '1'='1` as the username. Query becomes `SELECT * FROM users WHERE username='' OR '1'='1' AND password='...'`. The `OR '1'='1'` is always true, so the WHERE filter passes regardless of password. Attacker logs in.

## Practical — Café SQL lab

The lab gave two tables and a query editor. Walked through:

```sql
-- everything
SELECT * FROM Orders;

-- specific columns
SELECT drink, price FROM Orders;

-- filter
SELECT * FROM Orders WHERE drink = 'Coffee';

-- sort ascending (cheapest first, default)
SELECT * FROM Orders ORDER BY price;

-- sort descending (most expensive first)
SELECT * FROM Orders ORDER BY price DESC;

-- combine: filter then sort
SELECT * FROM Orders WHERE drink = 'Coffee' ORDER BY price DESC;
```

Four clauses cover most everyday queries: `SELECT` (what columns), `FROM` (which table), `WHERE` (which rows), `ORDER BY` (sort).

## What clicked

How readable SQL is once you see the structure. `SELECT drink, price FROM Orders WHERE drink = 'Coffee' ORDER BY price DESC` reads almost like English — "give me the drink and price columns from the Orders table, only rows where drink is Coffee, sorted by price highest first." Each clause answers a clear question, and they stack in a fixed order. It stopped looking like code and started looking like a way to phrase a question.

## What to revisit

- Practise basic SQL queries against a sample database (SQLZoo is free and good)
- SQL injection specifically — PortSwigger Web Academy has the best free labs
- Common database paths and config files on Linux/Windows for forensics work
