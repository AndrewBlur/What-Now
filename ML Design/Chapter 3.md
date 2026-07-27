## Fundamentals in Data Engineering

all about knowing how to collect, process, store, retrieve and process an increasingly growing amount of data

### Data Sources
- user input: needs heavy checking and processing, needs fast processing as users are assumed to be impatient
- system-generated data: logs, outputs, predictions, no need fast processing as compared to user input , only when something is on fire we need to look at system generated logs , these provide visibility for us 

examples: 
	```
	clicking,choosing a suggestion, scrolling, zooming,ignoring a pop-up,spending unusual amount of time on certain pages
	```
apps to log Logstash, Datadog, Logz.io etc
	`low access storage, low cost , can be used for logs`
	`higher-frequency access storage, high cost, for more important and fast access`

`first-party data: company data from the company's product`
`second-party data: data collected by another company on our customers`
`third-part data: data collected on public who arent our direct customers`

### Data formats
how we store data will determine the efficient retrieval 

```txt
The process of converting a data structure or object state into a format that
can be stored or transmitted and reconstructed later is data serialization
```

| Format   | Binary/Text | Example use cases             |
| -------- | ----------- | ----------------------------- |
| JSON     | Text        | Everywhere                    |
| CSV      | Text        | Everywhere                    |
| Parquet  | Binary      | Hadoop, Amazon Redshift       |
| Avro     | Binary      | Hadoop                        |
| protobuf | Binary      | Google, Tensorflow (TFRecord) |
| Pickle   | Binary      | Python, Pytorch serilization  |
|          |             |                               |
#### Row-Major Versus Column-Major Format
- **CSV is row-major**, so rows are stored together, making **row-wise access and frequent data writes faster**.
- **Parquet is column-major**, so columns are stored together, making **column-wise reads and selecting specific features much faster**.
- **Use CSV for write-heavy workloads** and **Parquet for analytics or ML workloads that frequently read selected columns from large datasets**.

#### Data Models
how the data is being represented
##### Data Normalization

- Removes duplicate data by splitting it into separate tables.
- Improves data integrity but requires joins across tables.

##### Relational Databases & SQL

- SQL is a **declarative language**—you specify _what_ data you want, not _how_ to retrieve it.
- Query optimizers choose the most efficient execution plan automatically.

#####  Declarative ML Systems

- Users define the data and task; the system selects and trains models automatically.
- Tools like **H2O AutoML** simplify modeling but don't solve production ML challenges.

##### NoSQL

- Designed for flexible schemas and specialized use cases.
- Main types are **Document** and **Graph** databases.

##### Document Model

- Stores data as self-contained JSON/XML/BSON documents with flexible schemas.
- Faster for retrieving related data, but joins across documents are inefficient.

#####  Graph Model

- Represents data as **nodes** and **edges** to model relationships.
- Best for relationship-heavy queries like social networks or recommendation systems


## 1. Data Passing Through Databases

- Data is shared by writing it to a common database that other processes can read.
- Simple but requires both processes to access the same database and suffers from higher latency due to database read/write operations.

## 2. Data Passing Through Services

- Processes communicate directly over a network using requests (request-driven architecture), commonly through REST or RPC.
- Enables independent services (microservices), but creates tight coupling and many inter-service requests as systems grow.

## 3. Data Passing Through Real-Time Transport

- Services publish events to a central broker instead of communicating directly, forming an event-driven architecture.
- Reduces service dependencies, improves scalability, and supports low-latency communication using in-memory brokers like Kafka, Kinesis, RabbitMQ, and RocketMQ.

## 4. Request-Driven vs Event-Driven Architecture

- **Request-driven:** Services request data synchronously from each other; suitable for logic-centric systems but can become a bottleneck.
- **Event-driven:** Services publish events to a broker, making systems loosely coupled and better suited for data-intensive applications.

## 5. Publish–Subscribe (Pub/Sub)

- Producers publish events to topics, and all subscribed services receive them.
- Producers are independent of consumers, and events are retained temporarily before deletion or permanent storage.

## 6. Message Queue

- Messages are sent to specific intended consumers rather than all subscribers.
- The message queue ensures each message reaches the correct consumer efficiently.

## 7. Batch Processing

- Processes historical data at scheduled intervals (e.g., daily) using engines like MapReduce and Spark.
- Best for computing slowly changing (static) features such as driver ratings.

## 8. Stream Processing

- Processes streaming data continuously or at short intervals for real-time decisions.
- Produces fast-changing (dynamic) features with low latency and avoids redundant computations by processing only new data.

## 9. Batch vs Stream Processing

- **Batch:** Efficient for large historical datasets and static features but has higher latency.
- **Stream:** Optimized for real-time, continuously changing data using engines like Apache Flink, KSQL, and Spark Streaming.

## 10. Stream Processing Engines

- Advanced stream processing requires dedicated engines to perform complex joins, aggregations, and feature extraction.
- Apache Flink, KSQL, and Spark Streaming are widely used, with Flink and KSQL offering SQL-like abstractions and treating batch processing as a special case of stream processing.