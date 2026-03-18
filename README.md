 
---

# Apache Flink 101 — 4 Hands-On Modules

**Mode:** Local only
**Environment:** Docker / Docker Compose
**Goal:** Learn Flink basics by running everything on your own machine

---

## Module 1: Local Setup + First Flink Job

Hands-on guide: [module-01-local-setup-first-flink-job/README.md](./module-01-local-setup-first-flink-job/README.md)

### What you will learn

* What Apache Flink is
* Difference between stream processing and batch processing
* How to run Flink locally with Docker
* How to submit and observe your first Flink job

### Topics covered

* Introduction
* Intro to Stream Processing with Apache Flink
* Docker Setup for the Hands-on Exercises
* Batch and Stream Processing (Docker)

### Hands-on tasks

1. Install Docker Desktop
2. Pull Flink Docker images
3. Start a local Flink cluster
4. Open Flink Web UI
5. Run a sample Flink job
6. Stop and restart the cluster

### Practical exercise

Create a local Flink environment with:

* JobManager
* TaskManager

Run a basic example job such as:

* Word count
* Datastream print job
* Simple SQL query job

### Expected outcome

By the end of this module, you should be able to:

* Start Flink locally
* Access the Web UI
* Submit a job
* See running/completed jobs

---

## Module 2: Flink Runtime + Web UI Monitoring

### What you will learn

* How Flink runtime works internally
* Role of JobManager and TaskManager
* Slots, parallelism, job graph, execution flow
* How to use the Flink Web UI for monitoring

### Topics covered

* The Flink Runtime
* The Flink Web UI (Docker)

### Hands-on tasks

1. Run Flink with multiple TaskManagers
2. Change job parallelism
3. Observe task slots in UI
4. Submit a job with higher parallelism
5. Inspect job graph, subtasks, and operator flow
6. Check logs from Docker containers

### Practical exercise

Do the following:

* Start 1 JobManager and 2 TaskManagers
* Configure parallelism = 2 or 4
* Run a simple streaming job
* Observe:

  * running tasks
  * subtasks
  * job status
  * checkpoint section if enabled

### Expected outcome

By the end of this module, you should understand:

* How Flink distributes work
* How parallelism affects execution
* How to use Web UI for monitoring and debugging

---

## Module 3: Kafka + Flink with Docker

### What you will learn

* How Flink consumes data from Kafka
* How Flink writes processed data back to Kafka
* Basic streaming pipeline using Docker only
* Intro to Flink SQL with Kafka sources/sinks

### Topics covered

* Using Kafka with Flink
* Flink and Kafka (Docker)
* Intro to Flink SQL

### Hands-on tasks

1. Add Kafka to Docker Compose
2. Create input and output Kafka topics
3. Produce sample events into Kafka
4. Use Flink to read from Kafka
5. Transform/filter/map the data
6. Write results to another Kafka topic
7. Verify output using Kafka console consumer

### Practical exercise

Build a mini pipeline:

**input topic** → **Flink job** → **output topic**

Example use cases:

* Filter valid orders only
* Uppercase messages
* Count events by type
* Route events based on field value

### Optional SQL exercise

Use Flink SQL to:

* create Kafka source table
* create Kafka sink table
* run `INSERT INTO ... SELECT ...`

### Expected outcome

By the end of this module, you should be able to:

* Connect Flink and Kafka locally
* Build a simple streaming pipeline
* Understand source, transformation, and sink flow

---

## Module 4: Stateful Processing + Event Time + Recovery

### What you will learn

* What stateful stream processing means
* Why event time and watermarks matter
* How late events are handled
* How checkpoints and recovery work in Flink

### Topics covered

* Stateful Stream Processing with Flink SQL
* Streaming Analytics (Docker)
* Event Time and Watermarks
* Watermarks (Docker)
* Checkpoints and Recovery
* Failure Recovery (Docker)

### Hands-on tasks

1. Create event stream with timestamps
2. Use tumbling or hopping windows
3. Apply watermark logic
4. Perform aggregations by key
5. Enable checkpoints
6. Simulate job failure
7. Restart job and verify recovery behavior

### Practical exercise

Build a local analytics use case such as:

* count orders per 1-minute window
* sum transaction amount by customer
* detect late-arriving events
* maintain state by key

Then test recovery:

* enable checkpointing
* kill the TaskManager container
* restart it
* observe whether the job resumes correctly

### Expected outcome

By the end of this module, you should understand:

* stateful processing
* windowing
* event time vs processing time
* watermarks
* checkpoint-based recovery

---

# Suggested Docker-Only Learning Flow

## Week 1

**Module 1**

* Setup
* First job
* Basic stream/batch idea

## Week 2

**Module 2**

* Runtime
* Parallelism
* Web UI monitoring

## Week 3

**Module 3**

* Kafka integration
* Flink SQL basics
* End-to-end streaming pipeline

## Week 4

**Module 4**

* Stateful analytics
* Watermarks
* Recovery and checkpoints

---

# Recommended Local Docker Stack

You can keep the setup minimal:

* `zookeeper` if using classic Kafka image
* `kafka`
* `jobmanager`
* `taskmanager`
* optional `sql-client`

For newer setups, Kafka can also run in **KRaft mode**, so Zookeeper may not be needed.

---

# Final 4 Module Names

## 1. Flink Local Setup and First Job

## 2. Flink Runtime, Parallelism, and Web UI

## 3. Kafka and Flink Streaming Pipeline with Docker

## 4. Stateful Processing, Watermarks, and Recovery

---

# Best beginner project after these 4 modules

Build a local **Order Stream Analytics System**:

* Kafka topic receives order events
* Flink reads events
* Flink filters invalid records
* Flink groups by product/category
* Flink calculates live counts/sums
* Output goes to another Kafka topic

That project will cover almost all 4 modules.
 
