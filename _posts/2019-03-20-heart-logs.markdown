---
title: I Heart Logs
date: 2019-03-20
categories: log
---

### What is Log in your mind?

![img](/assets/images/iheartlogs/1.png)

### Log - System Perspective

* A Log is a ***commit log*** or ***journal***. 
* Append only sequence of records ordered by time.

![img](/assets/images/iheartlogs/2.png)

### Log in a database

- Write-ahead logging(WAL)
  - Durability
  - Atomicity
- Replication data between databases

### Things to Consider in Distributed System

- Consistency
- Availability
- Partition Tolerance

### Log in Distributed System

* Log-Centric Design

  ![img](/assets/images/iheartlogs/3.png)

* Act as Consensus

  * Paxos
  * Raft

### Data Integration

![img](/assets/images/iheartlogs/4.png)

### Data Integration Painpoints

- Data is More Diverse
  - Every kind of data you can think of
- Explosion of Specialized Data Systems
  - OLAP
  - Search
  - Online Storage
  - Batch Processing
  - Stream Processing
  - Graph Analysis

### Log-Structured Data Flow

![img](/assets/images/iheartlogs/5.png)

### Fully Connected Architecture

![img](/assets/images/iheartlogs/6.png)

### Log-Centric Architecture

![img](/assets/images/iheartlogs/7.png)

### Scaling a Log

- Partitioning the Log
- Optimize by batching read and write
- Avoid needless data copy

### Stream Processing

> A stream processing job will be anything that reads from logs and writes output to logs or other systems. 

![img](/assets/images/iheartlogs/8.png)

### Log and Streaming Processing

- Logs make each data set to be a multi-subscriber
- Ensure the order is maintained in the processing done by each consumer
- Provide isolation and buffering

### Lambda Architecture

![img](/assets/images/iheartlogs/9.png)

- Good:
  - **Reprocessing** is possible as system envolve
- Bad:
  - Maintain code in two different complex system

### An Alternative

![img](/assets/images/iheartlogs/10.png)

### Log Compaction

![img](/assets/images/iheartlogs/11.png)

### Place of Log in System Architecture

- Handle data consistency
- Provide data replication between nodes
- Provide “commit” semantics to writer
- Provide external data subscription feed
- Provide capability to restore from failure
- Handle rebalancing of data between nodes

### References

- <https://en.wikipedia.org/wiki/Write-ahead_logging>
- <https://en.wikipedia.org/wiki/CAP_theorem>
- <https://en.wikipedia.org/wiki/Paxos_(computer_science)>
- <https://en.wikipedia.org/wiki/Maslow%27s_hierarchy_of_needs>



-End-
