## 1. Title

Local Flink Cluster Execution, Parallelism, and Web UI Monitoring

## 2. Objective

In this lab, you will start a local Apache Flink cluster with:

* `1 JobManager`
* `2 TaskManagers`

Then you will:

1. run a simple streaming job
2. set parallelism to `2`
3. change parallelism to `4`
4. watch what changes in the Flink Web UI

The goal is to understand, in simple words:

* how Flink distributes work
* what subtasks are
* how parallelism affects execution
* how to use the Web UI to monitor a job

## 3. Prerequisites

Make sure you have:

* Docker Desktop installed and running
* Docker Compose available through `docker compose`
* a terminal such as PowerShell, Command Prompt, or bash

Check your setup:

```bash
docker --version
docker compose version
docker ps
```

### What you should notice

If `docker ps` works, Docker is ready.

## 4. Project setup

Use this folder structure:

```text
apache-flink-101-docker-labs/
└── module-02-runtime-parallelism-web-ui/
    ├── docker-compose.yml
    └── README.md
```

Move into the lab folder:

```bash
cd module-02-runtime-parallelism-web-ui
```

### Sample `docker-compose.yml`

```yaml
services:
  jobmanager:
    image: flink:1.20.1-scala_2.12-java17
    command: jobmanager
    ports:
      - "8081:8081"
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager

  taskmanager-1:
    image: flink:1.20.1-scala_2.12-java17
    command: taskmanager
    depends_on:
      - jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
      - TASK_MANAGER_NUMBER_OF_TASK_SLOTS=2

  taskmanager-2:
    image: flink:1.20.1-scala_2.12-java17
    command: taskmanager
    depends_on:
      - jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
      - TASK_MANAGER_NUMBER_OF_TASK_SLOTS=2
```

### Simple explanation: JobManager vs TaskManager

* `JobManager`: the coordinator. It accepts the job, plans it, and tracks its progress.
* `TaskManager`: the worker. It runs the actual pieces of the job.

### Simple explanation: parallelism vs subtasks

* `Parallelism` means how many copies of the work Flink should run at the same time.
* A `subtask` is one copy of that work.

In simple words:

* parallelism `2` means Flink creates `2` subtasks for an operator
* parallelism `4` means Flink creates `4` subtasks for an operator

### How Flink distributes work

Flink splits the job into smaller pieces and sends those pieces to available task slots on the TaskManagers.

In this lab:

* each TaskManager has `2` task slots
* total slots in the cluster = `4`

That means:

* parallelism `2` fits easily
* parallelism `4` uses all available slots

### What you should notice

This setup is small on purpose. It is enough to see how work moves across workers without adding extra systems.

## 5. Start the Flink cluster

Run:

```bash
docker compose up -d
```

To stop the cluster later:

```bash
docker compose down
```

To follow startup logs:

```bash
docker compose logs -f
```

Stop log streaming with `Ctrl+C`.

### What you should notice

Docker should create three containers:

* `jobmanager`
* `taskmanager-1`
* `taskmanager-2`

## 6. Verify JobManager and TaskManagers are running

Run:

```bash
docker compose ps
```

You can also list all containers:

```bash
docker ps
```

### What you should notice

You should see:

* one JobManager container
* two TaskManager containers
* status `Up` or `running`

If one container is missing, check logs before moving on.

## 7. Open and use the Flink Web UI

Open this URL in your browser:

```text
http://localhost:8081
```

In the UI, look for:

* cluster overview
* number of TaskManagers
* available task slots
* Jobs section

### What you should notice

Before you submit a job:

* you should see `2` TaskManagers
* you should see `4` total task slots
* the Jobs page may be empty

## 8. Configure parallelism = 2

For the first run, use parallelism `2`.

This command submits a simple built-in streaming example in detached mode:

```bash
docker compose exec jobmanager ./bin/flink run -d -p 2 ./examples/streaming/StateMachineExample.jar
```

If your image stores example jars in a different path, find them first:

```bash
docker compose exec jobmanager find ./examples -name "*.jar"
```

### What you should notice

With parallelism `2`, Flink will try to run `2` subtasks for each parallel operator.

## 9. Run a simple streaming job

If you have not already submitted the job in the previous step, run:

```bash
docker compose exec jobmanager ./bin/flink run -d -p 2 ./examples/streaming/StateMachineExample.jar
```

Sample command to list running jobs:

```bash
docker compose exec jobmanager ./bin/flink list
```

Sample command to cancel the job later:

```bash
docker compose exec jobmanager ./bin/flink cancel <JOB_ID>
```

To stop everything at the end of the lab:

```bash
docker compose exec jobmanager ./bin/flink list
docker compose exec jobmanager ./bin/flink cancel <JOB_ID>
docker compose down
```

### What you should notice

This is a streaming job, so it usually keeps running until you cancel it.

That makes it useful for learning because you have time to inspect the UI.

## 10. Observe

### running tasks

Open the job in the Web UI and check whether the job status is `RUNNING`.

### What you should notice

A running streaming job should stay visible in the Jobs section.

### subtasks

Click an operator in the job graph and look for subtask details.

Layman explanation:

* if an operator has parallelism `2`, it has `2` workers doing the same kind of work
* each worker is a subtask

### What you should notice

With parallelism `2`, many operators will show `2` subtasks.

### job status

Look at the current state of the job.

Possible values:

* `RUNNING`
* `FINISHED`
* `FAILED`
* `CANCELED`

### What you should notice

For this lab, the job should normally show `RUNNING`.

### task slots

Go back to the cluster overview or TaskManagers page.

### What you should notice

You should see:

* `2` TaskManagers
* `4` total slots in the cluster
* some slots in use while the job is running

### operator view

Open the job graph and click different operators.

### What you should notice

You will usually see:

* the operator name
* the operator parallelism
* subtask metrics
* how data flows from one operator to another

### checkpoint section if enabled

Open the job details page and look for a `Checkpoints` tab or section.

### What you should notice

If the example job enables checkpointing, you will see checkpoint history there.

If the section is empty or there are no checkpoints, that usually means checkpointing is not enabled in that job. The cluster can still be healthy.

## 11. Change parallelism = 4

First, get the running job ID:

```bash
docker compose exec jobmanager ./bin/flink list
```

Cancel the old job:

```bash
docker compose exec jobmanager ./bin/flink cancel <JOB_ID>
```

Now submit the same job with parallelism `4`:

```bash
docker compose exec jobmanager ./bin/flink run -d -p 4 ./examples/streaming/StateMachineExample.jar
```

### What you should notice

Now Flink can create up to `4` subtasks for each parallel operator, which matches the `4` total slots in the cluster.

## 12. Compare what changed

Compare the two runs in the Web UI.

With parallelism `2`:

* fewer subtasks are created
* fewer slots are used
* the load is lighter

With parallelism `4`:

* more subtasks are created
* more slots are used
* work can spread across both TaskManagers more fully

### What you should notice

The most important difference is this:

* parallelism changes how many copies of the work Flink runs
* more parallelism usually means more subtasks in the UI

## 13. Enable checkpointing if possible

Checkpointing depends on the job, not only on the cluster.

Some built-in example jobs do not enable it. If your chosen example supports checkpointing, the Checkpoints tab will show activity while the job is running.

To inspect whether your current job shows checkpoints:

1. open the running job in the Web UI
2. click `Checkpoints`
3. wait a little and refresh

If you want to try another streaming example jar, first list what is available:

```bash
docker compose exec jobmanager find ./examples/streaming -name "*.jar"
```

Then run one of those jars and check the `Checkpoints` tab again.

### What you should notice

If checkpoints appear, you will see entries with a status such as `COMPLETED`.

If no checkpoints appear, the job is probably not using checkpointing.

## 14. Explain checkpoint status in simple words

A checkpoint is a saved progress point for a streaming job.

Simple meaning:

* `COMPLETED`: Flink successfully saved the job state
* `FAILED`: Flink tried to save state but something went wrong
* no checkpoints shown: the job probably does not have checkpointing enabled

Why this matters:

If a streaming job fails, Flink can use a successful checkpoint to continue from a recent safe point instead of starting everything from the beginning.

### What you should notice

A successful checkpoint is a good sign that Flink can recover state for that job.

## 15. Expected outcome

By the end of this lab, you should understand:

* how Flink distributes work
* how parallelism affects execution
* how to use the Web UI for monitoring and debugging

You should also be able to:

* start a Flink cluster locally
* verify 1 JobManager and 2 TaskManagers
* open the Flink Web UI at `http://localhost:8081`
* submit a streaming job
* compare parallelism `2` and `4`

## 16. Troubleshooting

### Problem: the Web UI does not open

Try:

```bash
docker compose ps
docker compose logs jobmanager
```

Check that:

* the JobManager is running
* port `8081` is free

### What you should notice

If the JobManager is not healthy, the Web UI will not load.

### Problem: only one TaskManager appears

Try:

```bash
docker compose logs taskmanager-1
docker compose logs taskmanager-2
```

### What you should notice

Both TaskManagers must connect to the JobManager before the cluster shows all workers.

### Problem: example jar not found

Run:

```bash
docker compose exec jobmanager find ./examples -name "*.jar"
```

Then use the exact jar path you find.

### What you should notice

Jar paths can vary slightly between Flink images.

### Problem: the job finishes too quickly

Use a streaming example instead of a batch example.

Try:

```bash
docker compose exec jobmanager find ./examples/streaming -name "*.jar"
```

### What you should notice

A long-running streaming job is easier to observe in the Web UI.

### Problem: parallelism 4 does not run well

Check available slots in the UI.

In this lab, you have:

* `2` TaskManagers
* `2` slots each
* `4` total slots

If you use more than `4`, some work may wait for free slots.

### What you should notice

Parallelism higher than available slots can still be submitted, but not everything can run at once.

### Common mistakes beginners make

* using a batch job when they want something long-running to observe
* forgetting `-d` and blocking the terminal with attached job output
* checking the Web UI before all containers are ready
* using parallelism higher than available slots without understanding why tasks wait
* assuming empty checkpoint data means the cluster is broken

## 17. Extra practice tasks

1. Stop one TaskManager and refresh the Web UI.
2. Start it again and see how the cluster view changes.
3. Run the job with parallelism `1`, then `2`, then `4`.
4. Count how many subtasks appear for one operator each time.
5. Try a different streaming example jar from the image.
6. Cancel a running job and confirm the status changes to `CANCELED`.
7. Use `docker compose logs -f taskmanager-1` while the job is running.

### What you should notice

These small experiments help you connect the CLI, the cluster view, and the job graph.

## Final summary for beginners

Apache Flink runs a job by splitting it into smaller parallel pieces.

* the `JobManager` coordinates the work
* the `TaskManagers` do the work
* `parallelism` decides how many copies of the work run
* each copy is called a `subtask`

When you increased parallelism from `2` to `4`, the Web UI showed more subtasks and more slot usage. That is the core idea behind cluster execution in Flink.
