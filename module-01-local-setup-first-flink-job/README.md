# Module 1: Local Setup + First Flink Job

## Module objective

Set up a small Apache Flink cluster on your own machine using only Docker Compose, then submit and observe your first Flink job.

This module uses:

* `1 JobManager`
* `1 TaskManager`

By keeping the setup small, you can focus on the Flink basics before moving to bigger examples.

## Prerequisites

Before you start, make sure you have:

* Docker Desktop installed and running
* Docker Compose available through `docker compose`
* At least 4 GB RAM available for Docker
* A terminal such as PowerShell, Command Prompt, or bash

Check that Docker is ready:

```bash
docker --version
docker compose version
docker ps
```

If `docker ps` works without errors, Docker is ready.

## What each container does

### JobManager

The `JobManager` is the control center of Flink.

It is responsible for:

* accepting jobs
* creating the execution plan
* coordinating job execution
* showing cluster and job details in the Flink Web UI

### TaskManager

The `TaskManager` is the worker node of Flink.

It is responsible for:

* running the actual job tasks
* using task slots to process work
* sending status and metrics back to the JobManager

For this beginner module, one TaskManager is enough.

## Folder structure

```text
apache-flink-101-docker-labs/
├── README.md
└── module-01-local-setup-first-flink-job/
    ├── docker-compose.yml
    └── README.md
```

## docker-compose.yml

This is the exact Docker Compose file used in this module:

```yaml
services:
  jobmanager:
    image: flink:1.20.1-scala_2.12-java17
    container_name: flink-jobmanager
    command: jobmanager
    ports:
      - "8081:8081"
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager

  taskmanager:
    image: flink:1.20.1-scala_2.12-java17
    container_name: flink-taskmanager
    command: taskmanager
    depends_on:
      - jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
      - TASK_MANAGER_NUMBER_OF_TASK_SLOTS=2
```

## Step-by-step commands

Move into the module folder:

```bash
cd module-01-local-setup-first-flink-job
```

Start the Flink environment:

```bash
docker compose up -d
```

Check that both containers are running:

```bash
docker compose ps
```

You should see:

* a `jobmanager` container
* a `taskmanager` container

Open the logs if you want to watch the startup:

```bash
docker compose logs -f
```

Stop log streaming with `Ctrl+C`.

## Start and stop the environment

### Start

```bash
docker compose up -d
```

### Stop

```bash
docker compose down
```

### Restart

```bash
docker compose down
docker compose up -d
```

## Access the Flink Web UI locally

Once the containers are running, open this URL in your browser:

```text
http://localhost:8081
```

In the Web UI, you should be able to see:

* cluster overview
* available task slots
* running or completed jobs
* job details after submission

## Practical exercise

In this exercise, you will:

1. Start the Flink cluster locally
2. Verify JobManager and TaskManager are running
3. Open the Flink Web UI
4. Run one basic example job
5. Check whether the job is running or completed
6. Inspect logs if something fails

### Exercise step 1: Start the Flink cluster locally

From the module folder, run:

```bash
docker compose up -d
```

### Exercise step 2: Verify JobManager and TaskManager are running

Run:

```bash
docker compose ps
```

You should see both services in the `running` state.

You can also check container names with:

```bash
docker ps
```

### Exercise step 3: Open the Flink Web UI

Open:

```text
http://localhost:8081
```

What to check in the UI:

* the cluster is reachable
* at least one TaskManager is listed
* task slots are available

### Exercise step 4: Run one basic example job

This module uses a simple built-in example from the Flink Docker image: `WordCount`.

Why this example is good for beginners:

* it is small
* it is a classic Flink example
* it lets you practice job submission and monitoring first

### Exercise step 5: Submit the job

Submit the example job from inside the JobManager container:

```bash
docker compose exec jobmanager ./bin/flink run ./examples/batch/WordCount.jar
```

If your local image stores the example in a different folder, list the examples first:

```bash
docker compose exec jobmanager ls ./examples
```

Then list inside a subfolder if needed:

```bash
docker compose exec jobmanager ls ./examples/batch
docker compose exec jobmanager ls ./examples/streaming
```

If `WordCount.jar` is not present in `batch`, use the jar location you find in the image.

### Exercise step 6: Check whether the job is running or completed

Use the Web UI:

* open the Jobs section
* look for your submitted job
* check whether the status is `RUNNING`, `FINISHED`, or `FAILED`

You can also use the CLI:

```bash
docker compose exec jobmanager ./bin/flink list
```

For completed jobs, this may finish very quickly, so the Web UI is often the easiest place to confirm that the job was submitted and completed.

### Exercise step 7: Inspect logs if something fails

Check both container logs:

```bash
docker compose logs jobmanager
docker compose logs taskmanager
```

To continuously follow logs:

```bash
docker compose logs -f jobmanager
docker compose logs -f taskmanager
```

Useful things to look for in logs:

* container startup errors
* missing example jar path
* port conflicts
* TaskManager not connecting to the JobManager

## Validation checklist

Use this checklist after completing the exercise:

* Docker Compose starts without errors
* `jobmanager` container is running
* `taskmanager` container is running
* Flink Web UI opens at `http://localhost:8081`
* At least one TaskManager appears in the Web UI
* You can submit an example Flink job
* You can see the job status as running or completed
* You know how to inspect logs when something fails

## Expected outcome

By the end of this module, you should be able to:

* Start Flink locally
* Access the Web UI
* Submit a job
* See running/completed jobs

## Common errors and fixes

### Error: `port is already allocated`

Cause:

Another service is already using port `8081`.

Fix:

* stop the other service using port `8081`
* or change the port mapping in `docker-compose.yml`, for example `8082:8081`

Then open the new port in your browser.

### Error: `Cannot connect to the Docker daemon`

Cause:

Docker Desktop is not running.

Fix:

Start Docker Desktop and wait until it is fully ready, then run:

```bash
docker ps
```

### Error: TaskManager does not appear in the Web UI

Cause:

The `taskmanager` container may not have started correctly or may not be able to reach the JobManager.

Fix:

Run:

```bash
docker compose ps
docker compose logs taskmanager
docker compose logs jobmanager
```

If needed, restart the environment:

```bash
docker compose down
docker compose up -d
```

### Error: container name is already in use

Cause:

You already have an older Flink container on your machine using the same name.

Fix:

Remove the old containers if they exist:

```bash
docker rm -f flink-jobmanager
docker rm -f flink-taskmanager
```

This module now avoids fixed container names, so after pulling the latest Compose file you can usually just run:

```bash
docker compose up -d
```

### Error: example jar not found

Cause:

The Flink image may store example jars in a slightly different path.

Fix:

Inspect the example folders:

```bash
docker compose exec jobmanager ls ./examples
docker compose exec jobmanager find ./examples -name "*.jar"
```

Then run the job again using the correct jar path.

### Error: Web UI does not load

Cause:

The JobManager container may not be ready yet, or the port mapping may be blocked.

Fix:

* wait 10 to 20 seconds and refresh
* run `docker compose ps`
* check `docker compose logs jobmanager`
* confirm that port `8081` is not blocked by another service

## Quick command summary

```bash
cd module-01-local-setup-first-flink-job
docker compose up -d
docker compose ps
docker compose exec jobmanager ./bin/flink run ./examples/batch/WordCount.jar
docker compose exec jobmanager ./bin/flink list
docker compose logs taskmanager
docker compose logs jobmanager
docker compose down
```
