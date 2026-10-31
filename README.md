# Distributed Task Queue

A distributed background job processing system I developed using **Python, FastAPI, Redis, and Docker** to explore reliable asynchronous processing, worker coordination, priority scheduling, retry mechanisms, and real-time job monitoring.

I designed this project as a simplified version of production task-processing systems such as Celery and RQ, focusing on practical backend engineering concepts, fault handling, observability, and maintainable architecture.

## Project Overview

Many applications rely on background processing for tasks such as sending emails, generating reports, processing data imports, executing webhooks, and running machine learning inference. These operations should not block incoming API requests and need reliable mechanisms for scheduling, execution, retries, and failure management.

I built this project to demonstrate how a Redis-backed task queue can coordinate API requests and independent worker processes while tracking each job throughout its lifecycle.

## Architecture

```text
                    Client
                       |
                 REST API / WebSocket
                       |
                       v
                 FastAPI Server
                       |
                       v
                     Redis
              ┌────────┼─────────┐
              │        │         │
         Priority   Delayed   Job State
          Queues     Jobs      Records
              │        │         │
              └────────┼─────────┘
                       |
                       v
                 Worker Processes
                       |
                       v
                  Task Executor
                       |
              ┌────────┼─────────┐
              │        │         │
           Success    Retry   Dead Letter
                       |
                       v
                Live Job Updates
                       |
                       v
                WebSocket Dashboard
```

The FastAPI application accepts job submissions and stores job information in Redis. Independent worker processes retrieve jobs according to their priority, execute tasks, update job states, and handle failures. WebSocket connections provide live updates to the monitoring dashboard.

## Key Features

* **REST API:** Developed endpoints for submitting jobs, retrieving job details, listing recent jobs, monitoring queue statistics, and checking application health.
* **Priority Scheduling:** Implemented high-, medium-, and low-priority queues to control job processing order.
* **Delayed Execution:** Used a Redis sorted set to schedule jobs for execution after a specified delay.
* **Independent Workers:** Implemented separate worker processes with configurable concurrency for background job execution.
* **Automatic Retries:** Added retry handling with exponential backoff to manage temporary task failures.
* **Dead-Letter Handling:** Implemented a dead-letter queue for jobs that exhaust their retry attempts.
* **Job Lifecycle Tracking:** Added states including `queued`, `delayed`, `running`, `succeeded`, `failed`, `retrying`, and `dead`.
* **Real-Time Monitoring:** Integrated WebSockets and Redis Pub/Sub to publish job status changes.
* **Monitoring Dashboard:** Built a browser-based dashboard to display queue statistics, job states, and recent events.
* **Structured Logging:** Integrated `structlog` for structured application logs.
* **Automated Testing:** Added a Pytest suite using Fakeredis to test API endpoints, queue operations, worker behavior, priority handling, and retries.
* **Containerization:** Configured Docker and Docker Compose to run the application components.
* **Continuous Integration:** Configured GitHub Actions to run linting and automated tests.

## Technology Stack

| Category                | Technologies              |
| ----------------------- | ------------------------- |
| Programming Language    | Python 3.11+              |
| Backend Framework       | FastAPI                   |
| Queue and State Storage | Redis                     |
| API Validation          | Pydantic                  |
| Real-Time Communication | WebSockets, Redis Pub/Sub |
| Testing                 | Pytest, Fakeredis         |
| Code Quality            | Ruff                      |
| Logging                 | Structlog                 |
| Containerization        | Docker, Docker Compose    |
| CI/CD                   | GitHub Actions            |

## Project Structure

```text
distributed-task-queue/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── models.py
│   ├── redis_client.py
│   ├── queue.py
│   ├── worker.py
│   ├── events.py
│   ├── dashboard.py
│   └── logging_config.py
├── tests/
│   ├── conftest.py
│   ├── test_api.py
│   ├── test_queue.py
│   ├── test_worker.py
│   ├── test_retries.py
│   └── test_priority.py
├── .github/
│   └── workflows/
│       └── ci.yml
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── README.md
├── .env.example
└── .gitignore
```

## Getting Started

### Prerequisites

Install the following tools before running the project:

* Python 3.11 or later
* Docker Desktop
* Git

### 1. Clone the Repository

```bash
git clone https://github.com/heshmasree2809/Distributed-Task-Queue.git
cd Distributed-Task-Queue
```

### 2. Create a Virtual Environment

**Windows:**

```powershell
py -m venv .venv
.venv\Scripts\activate
```

**Linux/macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -e ".[dev]"
```

### 4. Configure Environment Variables

Copy `.env.example` to `.env` and adjust the values if necessary.

**Windows:**

```powershell
Copy-Item .env.example .env
```

**Linux/macOS:**

```bash
cp .env.example .env
```

### 5. Start Redis

Using Docker:

```bash
docker run --rm -p 6379:6379 redis:7-alpine
```

Keep this terminal running.

### 6. Start the API Server

Open another terminal, activate the virtual environment, and run:

```bash
uvicorn app.main:app --reload
```

The API will be available at:

* API: `http://localhost:8000`
* Interactive API documentation: `http://localhost:8000/docs`
* Dashboard: `http://localhost:8000/dashboard`

### 7. Start a Worker

Open another terminal, activate the virtual environment, and run:

```bash
python -m app.worker
```

The worker will begin checking the configured queues for jobs to process.

## Docker Deployment

To start the application using Docker Compose:

```bash
docker compose up --build
```

The Compose configuration is intended to run the API, Redis, and worker components together.

Use the following local endpoints:

| Component                     | Address                           |
| ----------------------------- | --------------------------------- |
| API                           | `http://localhost:8000`           |
| Interactive API Documentation | `http://localhost:8000/docs`      |
| Monitoring Dashboard          | `http://localhost:8000/dashboard` |
| Redis                         | `localhost:6379`                  |

## API Usage

### Submit a Job

```bash
curl -X POST http://localhost:8000/jobs \
  -H "Content-Type: application/json" \
  -d '{
    "task_name": "send_email",
    "payload": {
      "to": "user@example.com",
      "subject": "Welcome"
    },
    "priority": "high",
    "delay_seconds": 0,
    "max_retries": 3
  }'
```

This request submits a high-priority job to the queue.

### Schedule a Delayed Job

```bash
curl -X POST http://localhost:8000/jobs \
  -H "Content-Type: application/json" \
  -d '{
    "task_name": "generate_report",
    "payload": {
      "report_id": "rpt_123"
    },
    "priority": "medium",
    "delay_seconds": 30,
    "max_retries": 2
  }'
```

This example schedules a job with a 30-second delay.

### Retrieve Job Status

Replace `<job_id>` with the ID returned when the job was submitted.

```bash
curl http://localhost:8000/jobs/<job_id>
```

### List Recent Jobs

```bash
curl http://localhost:8000/jobs
```

### View Queue Statistics

```bash
curl http://localhost:8000/queues/stats
```

### Check Application Health

```bash
curl http://localhost:8000/health
```

## Worker Execution and Failure Handling

I implemented a simulated task executor to make job execution and failure scenarios reproducible during development and testing.

The included executor supports these test scenarios:

* **Successful execution:** Normal jobs complete successfully.
* **Permanent failure:** Jobs named `always_fail` fail during execution.
* **Payload-triggered failure:** Jobs with `{"fail": true}` simulate a task failure.
* **Recoverable failure:** Jobs with `{"fail_until_attempt": 1}` fail once and succeed on a subsequent attempt.

These scenarios allow the retry mechanism and dead-letter handling to be tested without depending on external services.

### Retry Strategy

When a job fails and retry attempts remain, the worker schedules another attempt using exponential backoff:

```text
delay = RETRY_BACKOFF_BASE_SECONDS × 2^(attempts - 1)
```

For example, with a base delay of two seconds, successive retry delays are two, four, and eight seconds.

When the configured retry limit is exhausted, the job is marked `dead` and its ID is added to the dead-letter queue.

## Real-Time Monitoring

I integrated WebSockets and Redis Pub/Sub to support live job monitoring.

The dashboard displays:

* Total jobs submitted.
* Jobs waiting in queues.
* Jobs currently running.
* Successfully completed jobs.
* Failed and retrying jobs.
* Jobs in the dead-letter queue.
* Recent job events.

**WebSocket endpoint:**

```text
ws://localhost:8000/ws/jobs
```

**Dashboard:**

```text
http://localhost:8000/dashboard
```

## Testing and Code Quality

I included automated tests to validate important application behaviors.

Run the test suite:

```bash
pytest
```

Run the linter:

```bash
ruff check .
```

The tests use Fakeredis to simulate Redis interactions, allowing the test suite to run without a separate Redis server, provided all tested operations are supported by the configured Fakeredis version.

GitHub Actions is configured to automate linting and testing during the CI workflow.

## Configuration

The application uses environment variables for configuration.

Example `.env` settings:

```env
REDIS_URL=redis://localhost:6379/0
WORKER_CONCURRENCY=4
WORKER_POLL_INTERVAL_SECONDS=0.5
RETRY_BACKOFF_BASE_SECONDS=2
RECENT_JOBS_LIMIT=100
EVENT_STREAM_LIMIT=200
LOG_LEVEL=INFO
```

Adjust these values based on the local environment and expected workload.

## Engineering Challenges and Design Decisions

While developing this project, I focused on several backend engineering concerns:

* Separating API responsibilities from background worker execution.
* Representing job states consistently throughout the processing lifecycle.
* Scheduling jobs by priority and execution delay.
* Handling transient and permanent failures differently.
* Making failure scenarios reproducible for automated tests.
* Publishing job events for real-time monitoring.
* Keeping application configuration separate from source code.
* Automating code-quality checks and tests through CI.

These design choices make the project useful for exploring backend infrastructure, asynchronous execution, and fault-handling strategies.

## Potential Improvements

Future enhancements I would like to explore include:

* Registering typed task handlers instead of relying exclusively on simulated execution.
* Adding worker heartbeats and worker health monitoring.
* Supporting recurring scheduled jobs.
* Using Redis Streams or another durable delivery mechanism.
* Improving atomic job claiming and recovery from worker crashes.
* Adding Prometheus metrics and OpenTelemetry tracing.
* Introducing authentication and tenant-level job isolation.
* Adding dashboard filtering, job replay, and performance metrics.
* Benchmarking throughput and latency under different workloads.

## What I Learned

Developing this project helped me explore practical backend infrastructure concepts, including queue design, worker processes, asynchronous APIs, retry strategies, Redis data structures, automated testing, containerization, and CI workflows.

I built it as a hands-on engineering project to strengthen my understanding of how background processing systems are structured and how their behavior can be tested and monitored.

## Author

**Heshma Sree Avuthu**

GitHub: [@heshmasree2809](https://github.com/heshmasree2809)

Project: [Distributed Task Queue](https://github.com/heshmasree2809/Distributed-Task-Queue)
