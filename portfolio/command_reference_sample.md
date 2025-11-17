---
layout: page
title: ALTER TABLE ADD COLUMN (command reference sample)
permalink: /command-reference-sample/
---

{: .text-delta }
- TOC
{:toc}

This sample is a reference that describes an SQL-like command for adding a column to a table.
It is written for a fictional system called Datafuel but is based on a real product documentation that I created.

* **Type:** Command reference
* **Objective:** A description of how to use the `ALTER TABLE ADD COLUMN` command
* **Audience:** Developers, system analysts, system architects, and deployment teams
* **Tools Used:** Markdown, Jekyll, Hydejack theme, and GitHub Pages

The sample begins below the line.

▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬

## Overview

The query adds a column to a specified logical or proxy table.
The new column is added after all the existed columns of the table.

## Execution results

When a query completes successfully, the system updates:
* the structure of the table in the logical schema
* the structure of the connected physical tables in the data storage

After changing the table's structure, some `SELECT` queries on logical and materialized views connected to the table 
may stop working.
If this happens, recreate the affected views by deleting them and creating them again.

## How a query works

A query goes through the operation queue and is processed on a first-come, first-served basis.
The system records every change to a table in the changelog, which can be viewed using `GET_CHANGES`.

If an error occurs while processing a syntactically correct query, all subsequent DDL queries in the target logical database are locked.
To reset a DDL lock, follow steps described in section <>.

## Syntax

```sql
ALTER TABLE [db_name.]table_name
ADD COLUMN [IF NOT EXISTS] column_name data_type
[LOGICAL_ONLY]
```

**Parameters:**

`db_name`

: The name of the logical database that contains the target table.
  Optional if a default logical database is selected.

`table_name`

: The name of the target table. 
  You can specify a logical table or a proxy table.

`column_name`

: The name of the column to add.

`data_type`

: The logical data type of the column to add.
  For the list of supported data types, see Section <>.

### The `IF NOT EXISTS` keyword

Enables a check for the existence of the column before adding it. 
If specified, the system returns a successful response both when the column is added by the query and when the column already exists.
If not specified, the system returns a successful response only when the column is added by the query.

### The `LOGICAL_ONLY` keyword

Allows you to add a column at the logical level only (to the logical schema) 
without updating the physical schema of the table in the data storage.

If not specified, the column is added at both levels – logical and physical.

## Response

A response contains:
* an empty ResultSet object for a successfully executed query
* an exception for an unsuccessfully executed query

## Limitations

### Execution limitations

* A query on a logical table is not available if the table participates in a running write operation.
* A query cannot be executed if DDL changes are denied in the logical database.

### Name limitations

* The column name must be unique within the table and must meet the naming requirements described in Section <>.

### Partitioning limitations

* After adding a column to a partitioned table, the same column must be added to every partition of the table, and vise versa.

### Other limitations

* The added column must be nullable.
* Changing a table may require recreating the connected logical and materialized views.
* The information schema is updated asynchronously, so the added column may appear there with a slight delay.

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
ADD COLUMN manager_id BIGINT;
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
ADD COLUMN IF NOT EXISTS description VARCHAR;
```

### Adding a column with the LOGICAL_ONLY keyword {#ex_logical_only}

```sql
ALTER TABLE marketing_new.sales
ADD COLUMN client_id BIGINT
LOGICAL_ONLY;
```