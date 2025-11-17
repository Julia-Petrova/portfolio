---
layout: page
title: ALTER TABLE ADD COLUMN (command reference sample)
permalink: /command-reference-sample/
---

{: .text-delta }
- TOC
{:toc}

This sample is a reference describing an SQL-like command for adding a column to a table, which is independent 
of other samples in the portfolio.
It is for a fictional system Datafuel but based on a real product documentation that I wrote.

* **Type:** Reference
* **Objective:** A description of how to use the `ALTER TABLE ADD COLUMN` command
* **Audience:** Developers, system analysts, system architects, and deployment teams
* **Tools Used:** Markdown, Jekyll, Hydejack theme, GitHub Pages

The sample begins below the line.

▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬

## Overview

The query adds a column to a specified logical or proxy table.
The new column is added after all the existed columns of the table.

## Execution results

When a query completes successfully, the system updates:
* the structure of the table in the logical schema
* the structure of the connected physical tables in the data storage

After changing the table's structure, some `SELECT` queries to logical and materialized views connected with the table 
can stop working.
If that happens, recreate affected views by deleting them and then creating them back.

## How a query works

A query goes through operation queue and is processed fon a first come, first served basis.
The system records every change of a table into the changelog, which is available for viewing with `GET_CHANGES`.

In the event of an error while processing a syntactically correct query, all next DDL queries are locked in the target logical database.
To reset a DDL lock, follow steps described in section <>.

## Syntax

```sql
ALTER TABLE [db_name.]table_name
ADD COLUMN [IF NOT EXISTS] column_name data_type
[LOGICAL_ONLY]
```

**Parameters:**

`db_name`

: A name of the logical database that includes the target table.
  Optional if a default logical database is set up.

`table_name`

: The name of the target table. 
  You can specify a logical table or a proxy table.

`column_name`

: The name of the added column.

`data_type`

: The logical data type of the added column.
  The list of the supported data types see in the section <>.

### The `IF NOT EXISTS` keyword

Enables a check for the column to exist before its adding. 
If specified, the system returns a successful response when the column is added via the query and has existed before the query.
If not specified, the system returns a successful response only when the column has been added via the query.

### The `LOGICAL_ONLY` keyword

Allows you to add a column on the logical level only (to the logical schema) 
without updating the physical schema of the table in the data storage.

If not specified, the column is added on both levels—logical and physical.

## Response

A response contains:
* an empty ResultSet object for a successfully executed query
* an exception for an unsuccessfully executed query

## Limitations

### Execution limitations

* A query for a logical table is not available if the table participates in a running write operation.
* A query cannot be executed if ddl changes are denied in the logical database.

### Name limitations

* The column name must be unique within the table and match the requirements to names described in Section <>.

### Partitioning limitations

* After adding a column to a partitioned table, the same column must be added to every partition of the table and vise versa.

### Other limitations

* The added column must be nullable.
* Changing a table may require recreating the connected logical and materialized views.
* The information schema is updated asynchronously, which is why the added column may appear in the information schema with a small delay.

## Examples

### Adding a column without additional keywords {#ex_no_keywords}

Adding a column to a logical table in a specified logical database:
```sql
ALTER TABLE marketing.clients
ADD COLUMN description VARCHAR;

ALTER TABLE marketing.stores
ADD COLUMN previous_month_sales BIGINT;
```

Adding a column to a proxy table in a specified logical database:
```sql
ALTER TABLE marketing.payments_proxy
ADD COLUMN manager_id BIGINT
```

Adding a column to a logical table in a previously selected logical database:
```sql
USE marketing_new;

ALTER TABLE sales
ADD COLUMN manager_id BIGINT;
```

### Adding a column with the IF NOT EXISTS keyword {#ex_if_not_exists}

```sql
ALTER TABLE marketing.clients
ADD COLUMN IF NOT EXISTS description VARCHAR
```

### Adding a column with the LOGICAL_ONLY keyword {#ex_logical_only}

```sql
ALTER TABLE marketing_new.sales
ADD COLUMN client_id BIGINT
LOGICAL_ONLY
```