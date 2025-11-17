---
layout: page
title: Upload data within a stream (tutorial sample)
permalink: /tutorial-sample/
---

{: .text-delta }
- TOC
{:toc}

This sample explains how to upload data to a data mart building system within a stream using the feature described in [the concept sample](concept_sample.md).
It is for a fictional system Datafuel but based on a real product documentation that I wrote.

* **Type:** Tutorial
* **Objective:** A description of how to use the streaming data upload feature
* **Audience:** Developers, system analysts, system architects, and deployment teams
* **Tools Used:** Markdown, Jekyll, Hydejack theme, GitHub Pages

The sample begins below the line.

▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬

# Upload data within a stream

You can stream <> data to:
* logical tables of any type
* proxy tables
* standalone tables

For small updates (up to a few hundred records), consider using the data update feature instead.
{:.note title="Tip"}

## Create a writable table for a standalone table

To upload data to a standalone table, you need an external writable table linked to it.
If it does not exist, create one:
* If both the standalone table and its external writable table do not exist, run `CREATE WRITABLE EXTERNAL TABLE` <> with
  `OPTIONS ('auto.create.table.enable=true')` <>.
* Otherwise, run `CREATE WRITABLE EXTERNAL TABLE` without `OPTIONS`.

## Send data as a stream

To stream data in your code (see examples below):
1. Open an HTTP/2 or HTTP/1.1 connection.
2. Call the /upload <> endpoint.
3. Send a stream of Avro or CSV data chunks.
4. Close the connection.

### Example: sending CSV data as a stream

The following Java example (using Vertx) implements a basic HTTP client for streaming CSV data.

```java
// Create a Vertx instance
Vertx vertx = Vertx.vertx();

// Configure an HTTP client with HTTP/2 support
HttpClientOptions clientOptions = new HttpClientOptions();
clientOptions.setProtocolVersion(HttpVersion.HTTP_2);
clientOptions.setHttp2ClearTextUpgrade(false); // Disable HTTP/1.1 to HTTP/2 upgrade
HttpClient httpClient = vertx.createHttpClient(clientOptions);

// Send a streaming upload request to 'marketing.sales'
// - queryId=12345: Identifies the request
// - sysOp=0: Adds or modifies records
// - commitOnDisconnect=true: Ensures data is saved if the connection is lost
Future<Void> requestFuture = httpClient.request(HttpMethod.POST, 9090, "localhost", "/api/v1/datamarts/marketing/entities/sales/upload?sysOp=0&queryId=12345&commitOnDisconnect=true")
    .compose(request -> {
        // Enable chunked transfer
        request.setChunked(true);
        // Set the Content-Type to CSV
        request.putHeader(HttpHeaders.CONTENT_TYPE, "text/csv");
        // Start streaming data
        Future<Void> requestSendFuture = request.sendHead()
            .compose(unused -> {
                // Send data in the chosen format
                // Use request.write(Buffer) to send a data chunk
                // After sending all data chunks, end the query using request.end()
              
                // Example: upload all data chunks in a single call
                return request.end(csvData());
            });
        // Process responses from Datafuel
        Future<Void> responseFuture = request.response()
            .compose(response -> {
                System.out.println("Response status code: " + response.statusCode());
                Promise<Void> promise = Promise.promise();
                response.exceptionHandler(promise::tryFail);
                response.handler(dataFromServer -> {
                    System.out.println("Response data: " + dataFromServer.toString());
                });
                response.endHandler(v -> {
                    promise.tryComplete();
                });
                return promise.future();
            })
            // Log success or errors
            .onComplete(ar -> {
                request.reset();
                if (ar.succeeded()) {
                    System.out.println("Response success");
                }
                if (ar.failed()) {
                    ar.cause().printStackTrace();
                }
            });
        return Future.join(requestSendFuture, responseFuture)
            .<Void>mapEmpty();
    })
    // Final upload status
    .onSuccess(v -> System.out.println("Upload done"))
    .onFailure(Throwable::printStackTrace);
// Wait for the process to complete
requestFuture.toCompletionStage()
    .toCompletableFuture()
    .get();
// Close the Vertx instance
vertx.close();
```

### Example: sending Avro data as a stream

The following Java example (using Vertx) implements a basic HTTP client for streaming Avro data.

```java
// create a vertx instance
Vertx vertx = Vertx.vertx();

// create an HTTP client with parameters set
HttpClientOptions clientOptions = new HttpClientOptions();
// switch to HTTP/2
clientOptions.setProtocolVersion(HttpVersion.HTTP_2);
// disable switching from HTTP/1.1 to HTTP/2
clientOptions.setHttp2ClearTextUpgrade(false);
HttpClient httpClient = vertx.createHttpClient(clientOptions);

// send a query for streaming data upload to the logical table 'marketing.sales'
// the query with an identifier queryId=12345 adds and/or modifies records of the target table (sysOp=0)
// if the connection is lost, ask the system to save uploaded data (commitOnDisconnect=true)
Future<Void> requestFuture = httpClient.request(HttpMethod.POST, 9090, "localhost", "/api/v1/datamarts/marketing/entities/sales/upload?sysOp=0&queryId=12345&commitOnDisconnect=true")
    .compose(request -> {
        // divide data into chunks
        request.setChunked(true);
        // add the Content-Type header specifying the data format (Avro)
        request.putHeader(HttpHeaders.CONTENT_TYPE, "application/avro");
        // start a data stream
        Future<Void> requestSendFuture = request.sendHead()
            .compose(unused -> {
                // send data in the chosen format
                // use request.write(Buffer) for sending a data chunk
                // after sending all data chunks, end the query using request.end()
              
                // example: upload all data chunks using a single call
                return request.end(avroData());
            });
        // process responses from Datafuel
        Future<Void> responseFuture = request.response()
            .compose(response -> {
                // print the response code
                System.out.println("Response status code: " + response.statusCode());
                Promise<Void> promise = Promise.promise();
                // handle errors
                response.exceptionHandler(promise::tryFail);
                // process the data received from Datafuel
                response.handler(dataFromServer -> {
                    System.out.println("Response data: " + dataFromServer.toString());
                });
                // end building and processing of the response
                response.endHandler(v -> {
                    promise.tryComplete();
                });
                return promise.future();
            })
            // final processing: print the successful response or log the error
            .onComplete(ar -> {
                request.reset();
                if (ar.succeeded()) {
                    System.out.println("Response success");
                }
                if (ar.failed()) {
                    ar.cause().printStackTrace();
                }
            });
        return Future.join(requestSendFuture, responseFuture)
            .<Void>mapEmpty();
    })
    // finish the upload: print the completion message or log the error
    .onSuccess(v -> System.out.println("Upload done"))
    .onFailure(Throwable::printStackTrace);
requestFuture.toCompletionStage()
    .toCompletableFuture()
    // wait for the process to finish
    .get();
// close the vertx instance
vertx.close();
```