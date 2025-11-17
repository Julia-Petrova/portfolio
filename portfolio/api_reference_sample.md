---
layout: page
title: Get entity metadata (API reference sample)
permalink: /api-reference-sample/
---

{: .text-delta }
- TOC
{:toc}

This sample is a part of the entity HTTP API reference.
It describes a fictional system called Datafuel, but it is based on a real product documentation that I wrote.

* **Type:** API Reference
* **Objective:** A description of how to use an endpoint of the entity HTTP API
* **Audience:** Developers, system analysts, system architects, and deployment teams
* **Tools Used:** Markdown, Jekyll, Hydejack theme, and GitHub Pages

The sample begins below the line.

▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬

# Get entity metadata

GET `/api/v1/datamarts/{datamart}/entities/{entity}`

Retrieves the metadata for the specified entity.

## Path parameters

`datamart`
: The name of the logical database that contains the entity.

`entity`
: The name of the entity to retrieve.

## Headers

`x-request-id`
: An optional header containing a unique identifier for the request.
  If omitted, the system generates a UUID and returns it in the response.

## cURL

```text
curl -X 'GET' \
  'http://localhost:9090/api/v1/datamarts/marketing/entities/sales' \
  -H 'x-request-id: 3b8360c5-6c62-4a96-acf8-7a3724292ea5'
```

## Response

### Success — HTTP 200

Returns an object with the entity's metadata.

`id` (string)

: The unique identifier of the entity.

`name` (string)

: The entity name.

`schema` (string)

: The name of the logical database that contains the entity.

`entityType` (string)

: The type of the entity. Possible values:
  * `TABLE`
  * `UPLOAD_EXTERNAL_TABLE`
  * `DOWNLOAD_EXTERNAL_TABLE`
  * `READABLE_EXTERNAL_TABLE`
  * `WRITABLE_EXTERNAL_TABLE`

`externalTableFormat` (nullable string)

: The format of uploaded or downloaded data via the entity (currently, only Avro). 
  Applicable to `UPLOAD_EXTERNAL_TABLE` and `DOWNLOAD_EXTERNAL_TABLE`.

`externalTableSchema` (nullable string)

: The schema definition of the external table. Applicable to the following entities:
  * `UPLOAD_EXTERNAL_TABLE`
  * `DOWNLOAD_EXTERNAL_TABLE`
  * `READABLE_EXTERNAL_TABLE`
  * `WRITABLE_EXTERNAL_TABLE`

`externalTableLocationType` (nullable string)

: The location type for the external table.

  Possible values:
  * `KAFKA`: A Kafka broker used for uploading data via `UPLOAD_EXTERNAL_TABLE` or `WRITABLE_EXTERNAL_TABLE`.
  * `CORE_ADP`: A PostgreSQL datasource used for downloading data via `DOWNLOAD_EXTERNAL_TABLE` or reading data via `READABLE_EXTERNAL_TABLE`.

`externalTableLocationPath` (nullable string)

: The path to the external source: a Kafka topic or a standalone table in a datasource.

  The format depends on `externalTableLocationType`.
  * For `KAFKA`: `kafka_brokers://<kafka_broker_ip_or_name_1>[:<port_1>][,...]/<topic_name>`. 
  * For `CORE_ADP`: `core:<datasource_name>://<table_schema>.<table_name>`.

`externalTableOptions` (nullable string)

: The set of entity options in the format `option=value[; ... ]`.

  Possible options:
  * `auto.create.sys_op.enable`: Specifies whether the system column `sys_op` was added to `UPLOAD_EXTERNAL_TABLE` (`true` by default).
  * `auto.create.table.enable`: Specifies whether the corresponding standalone table was automatically created 
    for `READABLE_EXTERNAL_TABLE` or `WRITABLE_EXTERNAL_TABLE` (`false` by default).

`externalTableDownloadChunkSize` (nullable integer)

: The maximum number of records downloaded from the storage in a single Kafka message.
  Defaults to `EDML_DEFAULT_CHUNK_SIZE`.
  Applicable to `DOWNLOAD_EXTERNAL_TABLE`.

`destination` (nullable array)

: The list of datasources where the entity data is stored. Applicable to `TABLE` and `WRITABLE_EXTERNAL_TABLE`.

`fields` (array)

: The list of objects, each describing an entity column.

`fields.ordinalPosition` (integer)

: The column order, starting from 0.

`fields.name` (string)

: The column name.

`fields.type` 

: The column data type.

  Possible values:
  * `BOOLEAN`
  * `CHAR`
  * `VARCHAR`
  * `LINK`
  * `UUID`
  * `BIGINT`
  * `INTEGER`
  * `DOUBLE`
  * `FLOAT`
  * `DATE`
  * `TIME`
  * `TIMESTAMP`

`fields.size` (nullable integer)

: The column size.
  Applicable to the `CHAR` and `VARCHAR` data types.

`fields.accuracy` (nullable integer)

: The precision of the value. Applicable to the `TIME` or `TIMESTAMP` data types.

`fields.nullable` (boolean)

: Whether the column is nullable.

`fields.primaryOrder` (nullable integer)

: The order of the column in the primary key.

`fields.shardingOrder` (nullable integer)

: The order of the column in the sharding key. 
  Applicable to the following entity types:
  * `TABLE`
  * `READABLE_EXTERNAL_TABLE`
  * `WRITABLE_EXTERNAL_TABLE`

**Example response:**
```json
{
  "id": "8c0bb73b-4e50-4f00-b151-ed65ee689e83",
  "name": "sales",
  "schema": "marketing",
  "entityType": "TABLE",
  "externalTableFormat": null,
  "externalTableSchema": null,
  "externalTableLocationType": null,
  "externalTableLocationPath": null,
  "externalTableOptions": null,
  "externalTableDownloadChunkSize": null,
  "externalTableUploadMessageLimit": null,
  "destination": [
    "ADP",
    "ADP2"
  ],
  "fields": [
    {
      "ordinalPosition": 0,
      "name": "id",
      "type": "BIGINT",
      "size": null,
      "accuracy": null,
      "nullable": false,
      "primaryOrder": 1,
      "shardingOrder": 1
    },
    {
      "ordinalPosition": 1,
      "name": "transaction_date",
      "type": "TIMESTAMP",
      "size": null,
      "accuracy": 6,
      "nullable": false,
      "primaryOrder": null,
      "shardingOrder": null
    },
    {
      "ordinalPosition": 2,
      "name": "product_code",
      "type": "VARCHAR",
      "size": 256,
      "accuracy": null,
      "nullable": false,
      "primaryOrder": null,
      "shardingOrder": null
    },
    {
      "ordinalPosition": 3,
      "name": "product_units",
      "type": "BIGINT",
      "size": null,
      "accuracy": null,
      "nullable": false,
      "primaryOrder": null,
      "shardingOrder": null
    },
    {
      "ordinalPosition": 4,
      "name": "store_id",
      "type": "BIGINT",
      "size": null,
      "accuracy": null,
      "nullable": false,
      "primaryOrder": null,
      "shardingOrder": null
    },
    {
      "ordinalPosition": 5,
      "name": "description",
      "type": "VARCHAR",
      "size": 256,
      "accuracy": null,
      "nullable": true,
      "primaryOrder": null,
      "shardingOrder": null
    }
  ]
}
```

### Error

Returns a `exceptionMessage` string with a human-readable explanation of the problem.

**HTTP 500**
An explanation specific to the problem if the called endpoint exists.

**HTTP 404**
`Unknown endpoint` if the called endpoint doesn't exist.

**Example error response (`HTTP 500`):**

```json
{
  "exceptionMessage": "Database marketing23 does not exist"
}
```

**Example error response (`HTTP 404`):**

```json
{
  "exceptionMessage": "Unknown endpoint"
}
```
