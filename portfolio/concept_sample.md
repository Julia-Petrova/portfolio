---
layout: page
title: Streaming data upload (concept sample)
permalink: /concept-sample/
---

{: .text-delta }
- TOC
{:toc}

This sample provides a high-level overview of the streaming upload feature in a data mart building system
and includes links to further reading on the topic.
It is written for a fictional system called Datafuel but is based on a real product documentation that I created.

* **Type:** Concept topic
* **Objective:** An introduction to a new type of data upload in a data mart building system
* **Audience:** Developers, system analysts, system architects, and deployment teams
* **Tools Used:** Markdown, Jekyll, Hydejack theme, and GitHub Pages

The sample begins below the line.

▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬

# Streaming data upload

The streaming upload feature enables the transfer of large data streams into Datafuel in chunks.
Data is streamed over HTTP, bypassing Apache Kafka.
The external system responsible for streaming controls the chunk size and content.

## Tables and DBMS support

Data can be streamed into any table of the types listed in Section <>.
The target table can reside in any supported DBMS.

## Interfaces and protocols

Data can be streamed through the program's HTTP interface using either HTTP/2 or HTTP/1.1.

## Data formats

Data can be streamed in CSV or Avro format.
Each data chunk within a stream must be a complete data unit:
* CSV: Includes a header and data records.
* Avro: Contains a header with the Avro schema and data blocks.

You can adjust CSV parsing settings (delimiter, quotes, escape symbols, etc.) in the `csvParser` configuration section.

## Send a data stream

To send a data stream to Datafuel, use the /upload endpoint <> of the HTTP API.
For a full tutorial, see Section <>.

## Processing Stages

An external system streams chunked data to a destination table in Datafuel, which processes the stream in stages:
1. **Receiving and buffering incoming data:** Datafuel receives incoming data chunks and writes them to a raw data buffer,
   limited by the `STREAMING_UPLOAD_INPUT_BUFFER_SIZE_MB` parameter (default: 10 MB).
2. **Processing records:** Data chunks move to a processed records buffer, 
   limited by the `STREAMING_UPLOAD_DB_BUFFER_SIZE` parameter (default: 10,000 records).
3. **Batch transfer to datasources:** Records are sent in batches to the table's DBMS,
   with batch size controlled by the `STREAMING_UPLOAD_DB_BATCH_SIZE` parameter (default: 1,000 records).
4. **Intermediate responses:** For HTTP/2 uploads, Datafuel sends an intermediate response after each successful batch transfer.
5. **Final response:** Once the stream is complete, Datafuel returns a final response indicating the upload results.

## Data saving process

Streaming data is saved to a logical table in the following steps:
1. During transfer to DBMS, data is written as part of an unfinished write operation.
2. If an error occurs or the upload is canceled, data is automatically discarded.
3. Upon successful completion or cancellation with a specific option (see <>), 
   data is saved to the table as its current data version.

All data uploaded within a stream is part of a single write operation.
{:.note title="Note"}

When being streamed to a proxy or standalone table, data is saved immediately upon transfer to the target DBMS.
Once transferred, it cannot be canceled, even if an error occurs.

## Interrupted stream handling

An interrupted stream is handled in the following way:
* If an error occurs or the upload is canceled, Datafuel closes the write operation
  as unsuccessful and discards all the data uploaded within the stream.
* If the connection to the external system is lost or closed, the outcome depends on the request's `commitOnDisconnect` option <>:
  * **Enabled:** Datafuel closes the write operation as successful and saves the uploaded data.
  * **Disabled:** Datafuel treats the situation like an error or cancellation, discarding all data uploaded in the stream.