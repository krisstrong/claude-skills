# SQL Style Guide

Use this skill whenever writing, reviewing, or formatting SQL queries. All SQL must conform to [sqlstyle.guide](https://www.sqlstyle.guide/).

## Naming Conventions

- **Identifiers:** lowercase letters, numbers, and underscores only; begin with a letter; no trailing underscores; no consecutive underscores; max 30 characters
- **Tables:** collective or singular nouns (`staff`, not `employees`); no `tbl_` prefix; no object-oriented concatenation for join tables
- **Columns:** always singular; never repeat the table name; no generic `id` as the only primary key name
- **Aliases:** use `AS` explicitly; abbreviate meaningfully (first letter of each word: `staff` → `s`)
- **Stored procedures:** name must contain a verb; no `sp_` prefix
- **No camelCase anywhere; no Hungarian notation**

### Standard Suffixes

| Suffix | Use |
|--------|-----|
| `_id` | unique identifier / primary key |
| `_status` | flag or status value |
| `_total` | sum or aggregate |
| `_num` | numeric field |
| `_name` | name field |
| `_seq` | contiguous sequence |
| `_date` | date column |
| `_tally` | count |
| `_size` | dimension or file size |
| `_addr` | physical or digital address |

## Reserved Words

- **Always UPPERCASE** all reserved keywords (`SELECT`, `WHERE`, `FROM`, `JOIN`, `AND`, `OR`, `AS`, etc.)
- Use full keywords, not abbreviations
- Prefer ANSI SQL keywords over vendor-specific alternatives

## Whitespace & Indentation — The "River"

Align root keywords so their right edges meet a consistent column (the "river"), with SELECT-level clauses starting a new line:

```sql
SELECT f.average_height,
       f.average_diameter,
       f.planet_of_origin
  FROM flora AS f
 WHERE f.species_name = 'Trekkle'
   AND f.planet_of_origin != 'Earth';
```

Rules:
- Root keywords (`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `ORDER BY`, `HAVING`, `LIMIT`) right-align to the river
- Columns and expressions indent to the right of the river
- Commas go at the end of lines (not the start) for column lists
- Put `AND` / `OR` on a new line, aligned to the river
- One space before and after `=` and other operators
- Space after commas

## JOINs

Indent `JOIN` to sit at the river; group `ON` conditions beneath it:

```sql
SELECT r.last_name
  FROM riders AS r
       INNER JOIN bikes AS b
       ON r.bike_vin_num = b.vin_num
          AND b.engine_tally > 2
       INNER JOIN fuel AS f
       ON f.fuel_type = 'Diesel';
```

## Subqueries

Align opening parenthesis with the river; indent inner query consistently:

```sql
SELECT r.last_name,
       (SELECT MAX(YEAR(championship_date))
          FROM champions AS c
         WHERE c.last_name = r.last_name
           AND c.confirmed = 'Y') AS last_championship_year
  FROM riders AS r
 WHERE r.last_name IN (SELECT c.last_name
                         FROM champions AS c
                        WHERE YEAR(c.championship_date) > '2008'
                          AND c.confirmed = 'Y');
```

## Preferred Constructs

- Use `BETWEEN` instead of chained `AND` range conditions
- Use `IN()` instead of repeated `OR` clauses
- Use `CASE` expressions for conditional value interpretation
- Minimize `UNION` and avoid temporary tables where possible
- Store dates in ISO 8601 format: `YYYY-MM-DDTHH:MM:SS.SSSSS`

## CREATE TABLE

- Declare `PRIMARY KEY` immediately after `CREATE TABLE (...`
- Place column constraints beneath the column they apply to
- Order constraint clauses alphabetically (`ON DELETE` before `ON UPDATE`)
- Use `NUMERIC` / `DECIMAL` over `REAL` / `FLOAT`
- Avoid vendor-specific data types
- Default values must match the column's declared type and come before `NOT NULL`

```sql
CREATE TABLE staff (
    staff_id   INT            NOT NULL,
    first_name NVARCHAR(100)  NOT NULL,
    last_name  NVARCHAR(100)  NOT NULL,
    email      NVARCHAR(255)  NOT NULL,
    hire_date  DATE           NOT NULL DEFAULT CURRENT_DATE,

    CONSTRAINT pk_staff PRIMARY KEY (staff_id)
);
```

## Comments

- Use `--` for single-line comments; `/* */` for multi-line
- Start comment blocks with `/*` on its own line and end with `*/` on its own line
- Keep comments above the line they describe, not inline where possible

## Designs to Avoid

- `camelCase` or `PascalCase` identifiers
- `tbl_`, `sp_`, or any other Hungarian-notation prefix
- Plural table names (`employees` → use `staff`)
- Quoting identifiers unless strictly necessary
- Entity-Attribute-Value (EAV) schemas
- Splitting a single unit across multiple columns (e.g., amount and currency in separate columns when a standard type exists)
- Applying object-oriented patterns to relational schema design
