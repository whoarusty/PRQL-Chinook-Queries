# PRQL-Chinook-Queries


**Setup:**

[Chinook Database](https://github.com/lerocha/chinook-database)
(exported from SQLite and imported CSV files into DuckDB to work with
v0.8.0)

[DuckDB](https://github.com/duckdb/duckdb/) v0.8.0

[PRQL](https://github.com/PRQL/prql) v0.8.0 (via [DuckDB
Extension](https://github.com/ywelsch/duckdb-prql))

[PRQL-Query](https://github.com/prql/prql-query) v0.0.15

Windows 10

**Introduction:**

Demonstrating PRQL queries on the Chinook database.

Providing PRQL-Query and resulting SQL queries using the –no-exec option
since DuckDB already had the database open. Also used PRQL pipes to pass
to pq. Tested the SQL results in DuckDB to confirm results.


**Queries:**

1.  Provide a query showing Customers (just their full names, customer
    ID and country) who are not in the US.

PRQL:

``` elm
from customers
filter country != 'USA'
select [name = f"{first_name} {last_name}", country, customer_id]
```

DuckDB results:
```
┌───────────────────────┬────────────────┬─────────────┐
│         name          │    country     │ customer_id │
│        varchar        │    varchar     │    int32    │
├───────────────────────┼────────────────┼─────────────┤
│ Luís Gonçalves        │ Brazil         │           1 │
│ Leonie Köhler         │ Germany        │           2 │
│ François Tremblay     │ Canada         │           3 │
│ Bjorn Hansen          │ Norway         │           4 │
│ Frantisek Wichterlová │ Czech Republic │           5 │
│ Helena Holy           │ Czech Republic │           6 │
│ Astrid Gruber         │ Austria        │           7 │
│ Daan Peeters          │ Belgium        │           8 │
│ Kara Nielsen          │ Denmark        │           9 │
│ Eduardo Martins       │ Brazil         │          10 │
│ Alexandre Rocha       │ Brazil         │          11 │
│ Roberto Almeida       │ Brazil         │          12 │
│ Fernanda Ramos        │ Brazil         │          13 │
│ Mark Philips          │ Canada         │          14 │
│ Jennifer Peterson     │ Canada         │          15 │
│ Robert Brown          │ Canada         │          29 │
│ Edward Francis        │ Canada         │          30 │
│ Martha Silk           │ Canada         │          31 │
│ Aaron Mitchell        │ Canada         │          32 │
│ Ellie Sullivan        │ Canada         │          33 │
│       ·               │   ·            │           · │
│       ·               │   ·            │           · │
│       ·               │   ·            │           · │
│ Dominique Lefebvre    │ France         │          40 │
│ Marc Dubois           │ France         │          41 │
│ Wyatt Girard          │ France         │          42 │
│ Isabelle Mercier      │ France         │          43 │
│ Terhi Hämäläinen      │ Finland        │          44 │
│ Ladislav Kovács       │ Hungary        │          45 │
│ Hugh O'Reilly         │ Ireland        │          46 │
│ Lucas Mancini         │ Italy          │          47 │
│ Johannes Van der Berg │ Netherlands    │          48 │
│ Stanislaw Wójcik      │ Poland         │          49 │
│ Enrique Muñoz         │ Spain          │          50 │
│ Joakim Johansson      │ Sweden         │          51 │
│ Emma Jones            │ United Kingdom │          52 │
│ Phil Hughes           │ United Kingdom │          53 │
│ Steve Murray          │ United Kingdom │          54 │
│ Mark Taylor           │ Australia      │          55 │
│ Diego Gutiérrez       │ Argentina      │          56 │
│ Luis Rojas            │ Chile          │          57 │
│ Manoj Pareek          │ India          │          58 │
│ Puja Srivastava       │ India          │          59 │
├───────────────────────┴────────────────┴─────────────┤
│ 46 rows (40 shown)                         3 columns │
└──────────────────────────────────────────────────────┘
```
PRQL-Query:
pq --no-exec "from customers | filter country != 'USA' | select [first_name, last_name, country, customer_id]"

result:

``` sql
SELECT
  first_name,
  last_name,
  country,
  customer_id
FROM
  customers
WHERE
  country <> 'USA'
```

2.  Provide a query only showing the Customers from Brazil.

PRQL:

``` elm
from customers
filter country == 'Brazil'
select [customer_id, name = f"{first_name} {last_name}", country]
```

DuckDB results:

```
┌─────────────┬─────────────────┬─────────┐
│ customer_id │      name       │ country │
│    int32    │     varchar     │ varchar │
├─────────────┼─────────────────┼─────────┤
│           1 │ Luís Gonçalves  │ Brazil  │
│          10 │ Eduardo Martins │ Brazil  │
│          11 │ Alexandre Rocha │ Brazil  │
│          12 │ Roberto Almeida │ Brazil  │
│          13 │ Fernanda Ramos  │ Brazil  │
└─────────────┴─────────────────┴─────────┘
```

PRQL-Query:
pq --no-exec "from customers | filter country == 'Brazil' | select [customer_id, first_name, last_name, country]"

Notes: had to take out the concatenation of name for pq to work

results:

``` sql
SELECT
  customer_id,
  first_name,
  last_name,
  country
FROM
  customers
WHERE
  country = 'Brazil'
```

3.  Provide a query showing the Invoices of customers who are from
    Brazil. The resultant table should show the customer’s full name,
    Invoice ID, Date of the invoice and billing country.

PRQL:

``` elm
from c=customers
join i=invoices [c.customer_id == i.customer_id]
filter c.country == 'Brazil'
select [c.customer_id, Name = f"{c.first_name} {c.last_name}", i.invoice_id, i.invoice_date, i.billing_country]
take 5
```

DuckDB results:
```
┌─────────────┬─────────────────┬────────────┬──────────────┬─────────────────┐
│ customer_id │      Name       │ invoice_id │ invoice_date │ billing_country │
│    int32    │     varchar     │   int32    │     date     │     varchar     │
├─────────────┼─────────────────┼────────────┼──────────────┼─────────────────┤
│          10 │ Eduardo Martins │         25 │ 2009-04-09   │ Brazil          │
│          12 │ Roberto Almeida │         34 │ 2009-05-23   │ Brazil          │
│          13 │ Fernanda Ramos  │         35 │ 2009-06-05   │ Brazil          │
│          11 │ Alexandre Rocha │         57 │ 2009-09-06   │ Brazil          │
│          13 │ Fernanda Ramos  │         58 │ 2009-09-07   │ Brazil          │
└─────────────┴─────────────────┴────────────┴──────────────┴─────────────────┘
```

PRQL-Query:
pq --no-exec "from c=customers | join i=invoices [c.customer_id == i.customer_id] | filter c.country == 'Brazil' | select [c.customer_id, c.first_name, c.last_name, i.invoice_id, i.invoice_date, i.billing_country] | take 5"

results:

``` sql
SELECT
  c.customer_id,
  c.first_name,
  c.last_name,
  i.invoice_id,
  i.invoice_date,
  i.billing_country
FROM
  customers AS c
  JOIN invoices AS i ON c.customer_id = i.customer_id
WHERE
  c.country = 'Brazil'
LIMIT
  5
```

4.  Provide a query showing only the Employees who are Sales Agents.

PRQL:

``` elm
func like fld str -> s"{fld} like '%' || {str} || '%' " # from: https://github.com/PRQL/prql/issues/1123
from employees
filter (like title 'Agent')
select [employee_id, title]
```

Note: like work around is difficult to currently support two words – had to change from ‘Sales Agent’ to just ‘Agent’

DuckDB results:
```
┌─────────────┬─────────────────────┐
│ employee_id │        title        │
│    int32    │       varchar       │
├─────────────┼─────────────────────┤
│           3 │ Sales Support Agent │
│           4 │ Sales Support Agent │
│           5 │ Sales Support Agent │
└─────────────┴─────────────────────┘
```

PRQL-Query:
pq --no-exec "func like fld str -> s"{fld} like '%' || {str} || '%' " # from: https://github.com/PRQL/prql/issues/1123 | from employees | filter (like title 'Agent') | select [employee_id, title]"

result: error: unexpected argument ‘like’ found

5.  Provide a query showing a unique list of billing countries from the Invoice table.

PRQL:

``` elm
from invoices
select billing_country
group billing_country (take 1)
```

DuckDB results:
```
┌─────────────────┐
│ billing_country │
│     varchar     │
├─────────────────┤
│ Germany         │
│ Norway          │
│ Belgium         │
│ Canada          │
│ USA             │
│ France          │
│ Ireland         │
│ United Kingdom  │
│ Australia       │
│ Chile           │
│ India           │
│ Brazil          │
│ Portugal        │
│ Netherlands     │
│ Spain           │
│ Sweden          │
│ Czech Republic  │
│ Finland         │
│ Denmark         │
│ Italy           │
│ Poland          │
│ Austria         │
│ Hungary         │
│ Argentina       │
├─────────────────┤
│     24 rows     │
└─────────────────┘
```

PRQL-Query:
pq --no-exec "from invoices | select billing_country | group billing_country (take 1)"

result:

``` sql
SELECT
  DISTINCT billing_country
FROM
  invoices
```

6.  Provide a query that shows the invoices associated with each sales
    agent. The resultant table should include the Sales Agent’s full
    name.

PRQL:

``` elm
from c=customers
join side:left i=invoices [i.customer_id == c.customer_id]
join side:left e=employees [e.employee_id == c.support_rep_id]
select [i.invoice_id, employee_name = f"{e.first_name} {e.last_name}"]
```

DuckDB results:
```
┌────────────┬───────────────┐
│ invoice_id │ employee_name │
│   int32    │    varchar    │
├────────────┼───────────────┤
│          1 │ Steve Johnson │
│          2 │ Margaret Park │
│          3 │ Margaret Park │
│          4 │ Steve Johnson │
│          5 │ Margaret Park │
│          6 │ Jane Peacock  │
│          7 │ Jane Peacock  │
│          8 │ Margaret Park │
│          9 │ Jane Peacock  │
│         10 │ Jane Peacock  │
│         11 │ Jane Peacock  │
│         12 │ Steve Johnson │
│         13 │ Margaret Park │
│         14 │ Steve Johnson │
│         15 │ Jane Peacock  │
│         16 │ Steve Johnson │
│         17 │ Steve Johnson │
│         18 │ Steve Johnson │
│         19 │ Margaret Park │
│         20 │ Steve Johnson │
│          · │       ·       │
│          · │       ·       │
│          · │       ·       │
│        406 │ Steve Johnson │
│        407 │ Margaret Park │
│        408 │ Steve Johnson │
│        409 │ Jane Peacock  │
│        410 │ Margaret Park │
│        412 │ Jane Peacock  │
│         41 │ Steve Johnson │
│         53 │ Jane Peacock  │
│        162 │ Steve Johnson │
│        173 │ Steve Johnson │
│        182 │ Jane Peacock  │
│        205 │ Jane Peacock  │
│        227 │ Jane Peacock  │
│        228 │ Steve Johnson │
│        279 │ Jane Peacock  │
│        357 │ Steve Johnson │
│        380 │ Steve Johnson │
│        400 │ Jane Peacock  │
│        402 │ Steve Johnson │
│        411 │ Jane Peacock  │
├────────────┴───────────────┤
│    412 rows (40 shown)     │
└────────────────────────────┘
```

PRQL-Query:
pq --no-exec "from c=customers | join side:left i=invoices [i.customer_id == c.customer_id] | join side:left e=employees [e.employee_id == c.support_rep_id] | select [i.invoice_id, e.first_name, e.last_name]"

result:

``` sql
SELECT
  i.invoice_id,
  e.first_name,
  e.last_name
FROM
  customers AS c
  LEFT JOIN invoices AS i ON i.customer_id = c.customer_id
  LEFT JOIN employees AS e ON e.employee_id = c.support_rep_id
```

7.  Provide a query that shows the Invoice Total, Customer name, Country
    and Sale Agent name for all invoices and customers.

PRQL:

``` elm
from c=customers
join side:left i=invoices [i.customer_id == c.customer_id]
join side:left e=employees [e.employee_id == c.support_rep_id]
select [Customer_Name = f"{c.first_name} {c.last_name}", Employee_Name = f"{e.first_name} {e.last_name}", i.total, c.country, e.title]
```

DuckDB results:
```
┌────────────────────┬───────────────┬───────────────┬────────────────┬─────────────────────┐
│   Customer_Name    │ Employee_Name │     total     │    country     │        title        │
│      varchar       │    varchar    │ decimal(10,2) │    varchar     │       varchar       │
├────────────────────┼───────────────┼───────────────┼────────────────┼─────────────────────┤
│ Leonie Köhler      │ Steve Johnson │          1.98 │ Germany        │ Sales Support Agent │
│ Bjorn Hansen       │ Margaret Park │          3.96 │ Norway         │ Sales Support Agent │
│ Daan Peeters       │ Margaret Park │          5.94 │ Belgium        │ Sales Support Agent │
│ Mark Philips       │ Steve Johnson │          8.91 │ Canada         │ Sales Support Agent │
│ John Gordon        │ Margaret Park │         13.86 │ USA            │ Sales Support Agent │
│ Fynn Zimmermann    │ Jane Peacock  │          0.99 │ Germany        │ Sales Support Agent │
│ Niklas Schröder    │ Jane Peacock  │          1.98 │ Germany        │ Sales Support Agent │
│ Dominique Lefebvre │ Margaret Park │          1.98 │ France         │ Sales Support Agent │
│ Wyatt Girard       │ Jane Peacock  │          3.96 │ France         │ Sales Support Agent │
│ Hugh O'Reilly      │ Jane Peacock  │          5.94 │ Ireland        │ Sales Support Agent │
│ Emma Jones         │ Jane Peacock  │          8.91 │ United Kingdom │ Sales Support Agent │
│ Leonie Köhler      │ Steve Johnson │         13.86 │ Germany        │ Sales Support Agent │
│ Frank Harris       │ Margaret Park │          0.99 │ USA            │ Sales Support Agent │
│ Jack Smith         │ Steve Johnson │          1.98 │ USA            │ Sales Support Agent │
│ Tim Goyer          │ Jane Peacock  │          1.98 │ USA            │ Sales Support Agent │
│ Kathy Chase        │ Steve Johnson │          3.96 │ USA            │ Sales Support Agent │
│ Victor Stevens     │ Steve Johnson │          5.94 │ USA            │ Sales Support Agent │
│ Martha Silk        │ Steve Johnson │          8.91 │ Canada         │ Sales Support Agent │
│ Dominique Lefebvre │ Margaret Park │         13.86 │ France         │ Sales Support Agent │
│ Steve Murray       │ Steve Johnson │          0.99 │ United Kingdom │ Sales Support Agent │
│      ·             │       ·       │            ·  │  ·             │          ·          │
│      ·             │       ·       │            ·  │  ·             │          ·          │
│      ·             │       ·       │            ·  │  ·             │          ·          │
│ Kathy Chase        │ Steve Johnson │          1.98 │ USA            │ Sales Support Agent │
│ John Gordon        │ Margaret Park │          1.98 │ USA            │ Sales Support Agent │
│ Victor Stevens     │ Steve Johnson │          3.96 │ USA            │ Sales Support Agent │
│ Robert Brown       │ Jane Peacock  │          5.94 │ Canada         │ Sales Support Agent │
│ Madalena Sampaio   │ Margaret Park │          8.91 │ Portugal       │ Sales Support Agent │
│ Manoj Pareek       │ Jane Peacock  │          1.99 │ India          │ Sales Support Agent │
│ Enrique Muñoz      │ Steve Johnson │          0.99 │ Spain          │ Sales Support Agent │
│ Terhi Hämäläinen   │ Jane Peacock  │          8.91 │ Finland        │ Sales Support Agent │
│ Enrique Muñoz      │ Steve Johnson │          1.98 │ Spain          │ Sales Support Agent │
│ Enrique Muñoz      │ Steve Johnson │         13.86 │ Spain          │ Sales Support Agent │
│ Terhi Hämäläinen   │ Jane Peacock  │          1.98 │ Finland        │ Sales Support Agent │
│ Terhi Hämäläinen   │ Jane Peacock  │          7.96 │ Finland        │ Sales Support Agent │
│ Terhi Hämäläinen   │ Jane Peacock  │          5.94 │ Finland        │ Sales Support Agent │
│ Enrique Muñoz      │ Steve Johnson │          8.91 │ Spain          │ Sales Support Agent │
│ Terhi Hämäläinen   │ Jane Peacock  │          0.99 │ Finland        │ Sales Support Agent │
│ Enrique Muñoz      │ Steve Johnson │          1.98 │ Spain          │ Sales Support Agent │
│ Enrique Muñoz      │ Steve Johnson │          3.96 │ Spain          │ Sales Support Agent │
│ Terhi Hämäläinen   │ Jane Peacock  │          1.98 │ Finland        │ Sales Support Agent │
│ Enrique Muñoz      │ Steve Johnson │          5.94 │ Spain          │ Sales Support Agent │
│ Terhi Hämäläinen   │ Jane Peacock  │         13.86 │ Finland        │ Sales Support Agent │
├────────────────────┴───────────────┴───────────────┴────────────────┴─────────────────────┤
│ 412 rows (40 shown)                                                             5 columns │
└───────────────────────────────────────────────────────────────────────────────────────────┘
```

PRQL-Query:
pq --no-exec "from c=customers | join side:left i=invoices [i.customer_id == c.customer_id] | join side:left e=employees [e.employee_id == c.support_rep_id] | select [c.first_name, c.last_name, e.first_name, e.last_name, i.total, c.country, e.title]"

result:

``` sql
SELECT
  c.first_name AS _expr_0,
  c.last_name AS _expr_1,
  e.first_name,
  e.last_name,
  i.total,
  c.country,
  e.title
FROM
  customers AS c
  LEFT JOIN invoices AS i ON i.customer_id = c.customer_id
  LEFT JOIN employees AS e ON e.employee_id = c.support_rep_id
```

Note: removed employee name concatenation for pq to provide results

8.  How many Invoices were there in 2009 and 2011? What are the
    respective total sales for each of those years?

PRQL:

``` elm
from invoices
derive year = s"date_part('year', invoice_date)"
group year (
aggregate [count]
)
filter (year == 2009 || year == 2011)
```

DuckDB results:
```
┌───────┬──────────────┐
│ year  │ count_star() │
│ int64 │    int64     │
├───────┼──────────────┤
│  2009 │           83 │
│  2011 │           83 │
└───────┴──────────────┘
```

PRQL-Query:
pq --no-exec "from invoices | derive year = s"date_part('year', invoice_date)" | group year (aggregate [count])"

result: error: unexpected argument ‘invoice_date) \| group year
(aggregate \[count\])’ found

9.  Looking at the InvoiceLine table, provide a query that COUNTs the
    number of line items for Invoice ID 37.

PRQL:

``` elm
from invoice_items
filter invoice_id == 37
aggregate [line_items_ID_37 = count]
```

DuckDB results:
```
┌──────────────────┐
│ line_items_ID_37 │
│      int64       │
├──────────────────┤
│                4 │
└──────────────────┘
```

PRQL-Query:
pq --no-exec "from invoice_items | filter invoice_id == 37 | aggregate [line_items_ID_37 = count]"

result:

``` sql
SELECT
  COUNT(*) AS "line_items_ID_37"
FROM
  invoice_items
WHERE
  invoice_id = 37
```

10. Looking at the Invoices table, provide a query that COUNTs the
    number of line items for each Invoice.

PRQL:

``` elm
from invoices
group invoice_id(
aggregate[ct=count]
)
```

DuckDB results:
```
┌────────────┬───────┐
│ invoice_id │  ct   │
│   int32    │ int64 │
├────────────┼───────┤
│          1 │     1 │
│          2 │     1 │
│          3 │     1 │
│          4 │     1 │
│          5 │     1 │
│          6 │     1 │
│          7 │     1 │
│          8 │     1 │
│          9 │     1 │
│         10 │     1 │
│         11 │     1 │
│         12 │     1 │
│         13 │     1 │
│         14 │     1 │
│         15 │     1 │
│         16 │     1 │
│         17 │     1 │
│         18 │     1 │
│         19 │     1 │
│         20 │     1 │
│          · │     · │
│          · │     · │
│          · │     · │
│        393 │     1 │
│        394 │     1 │
│        395 │     1 │
│        396 │     1 │
│        397 │     1 │
│        398 │     1 │
│        399 │     1 │
│        400 │     1 │
│        401 │     1 │
│        402 │     1 │
│        403 │     1 │
│        404 │     1 │
│        405 │     1 │
│        406 │     1 │
│        407 │     1 │
│        408 │     1 │
│        409 │     1 │
│        410 │     1 │
│        411 │     1 │
│        412 │     1 │
├────────────┴───────┤
│      412 rows      │
│     (40 shown)     │
└────────────────────┘
```

PRQL-Query:
pq --no-exec "from invoices | group invoice_id(aggregate[ct=count])"

result:

``` sql
SELECT
  invoice_id,
  COUNT(*) AS ct
FROM
  invoices
GROUP BY
  invoice_id
```

**Credits:**

Queries adapted from from Alaa Sedeeq

Source:
https://www.kaggle.com/code/alaasedeeq/chinook-questions-with-sqlite/notebook
under the Apache 2.0 open source license.
