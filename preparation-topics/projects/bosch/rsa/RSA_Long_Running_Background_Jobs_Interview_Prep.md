# Long-Running Background Jobs — Interview Preparation Guide

## RSA / MiDAS Context

**Target role:** Senior / Staff / Principal AI Engineer  
**Project context:** Bosch MiDAS — Requirement Similarity Assist (RSA)  
**Focus:** Long-running jobs, asynchronous processing, scalability, reliability, and production architecture

---

## 1. Executive Summary

A long-running background job is work that should not be performed inside the normal synchronous HTTP request/response lifecycle because it may take seconds, minutes, or hours.

The standard production pattern is:

```text
Client
  |
  | POST /jobs
  v
API
  |
  | Create job + enqueue work
  v
Queue / Orchestrator
  |
  v
Worker Pool
  |
  +--> External services
  |
  v
Result Storage
  |
  v
Job Status / Result API
```

The API responds quickly with a **job ID** rather than waiting for the complete computation.

For RSA, there are two execution modes:

1. **Interactive:** `POST /rsa/ss6/v1/similar-requirements`
   - Processes one requirement.
   - Embedding → SS3 vector search → SS1 LLM scoring → filtering → response.
   - Synchronous because the user expects an interactive result.

2. **Batch:** `POST /rsa/ss6/v1/pipeline`
   - Processes a complete requirements file.
   - Triggers an Azure Data Factory (ADF) pipeline.
   - ADF reads the input from Azure Blob Storage, processes requirements in parallel using SS3 and SS1, and writes results back to Blob Storage.
   - The ADF `run_id` is used to track the long-running operation through `GET /pipeline/{run_id}`.

**Key interview takeaway:**

> Use synchronous APIs for short interactive operations. Use asynchronous job processing for long-running work. Decouple request handling from execution and design the job system around reliability, scalability, observability, and cost.

---

# 2. Why Long-Running Jobs Need a Different Architecture

Consider RSA's single-requirement flow:

```text
Requirement
    |
    v
Embedding
    |
    v
SS3 — Vector Search
    |
    v
Top-K candidates
    |
    v
SS1 — LLM Similarity Scoring
    |
    v
Threshold Filtering
    |
    v
Response
```

For one requirement, this can reasonably fit inside an HTTP request.

Now consider a file containing 10,000 requirements:

```text
10,000 requirements
        |
        +--> Embedding / retrieval
        |
        +--> SS3
        |
        +--> SS1 LLM scoring
        |
        +--> Result processing
        |
        v
Large result set
```

Processing can take minutes or longer.

Keeping an HTTP request open for that duration creates problems:

- Open connections consume resources.
- API gateway/load-balancer timeouts may be exceeded.
- Client disconnects become difficult to handle.
- Retries become ambiguous.
- Failure recovery becomes harder.
- API instances remain tied to long-running work.
- Scaling becomes inefficient.
- Progress tracking is awkward.

Increasing the HTTP timeout is therefore usually not the right architectural solution.

---

# 3. Synchronous vs Asynchronous Processing

## 3.1 Synchronous

The client waits for the operation to complete.

```text
Client
  |
  | Request
  v
API
  |
  | Processing
  |
  v
Result
  |
  v
Client
```

Good for:

- CRUD operations
- Fast inference
- Single-record similarity search
- Interactive user requests
- Operations that normally complete quickly

RSA example:

```text
POST /rsa/ss6/v1/similar-requirements
```

---

## 3.2 Asynchronous

The client submits work and receives an acknowledgement immediately.

```text
Client
  |
  | POST /jobs
  v
API
  |
  | Submit job
  v
Queue / Orchestrator
  |
  v
Worker
  |
  v
Result Storage

Client
  |
  | GET /jobs/{job_id}
  v
API
  |
  v
Job Status
```

Good for:

- Large batch inference
- Document processing
- File ingestion
- Report generation
- Data pipelines
- Large-scale LLM processing
- Video/image processing
- Long-running workflows

RSA's `/pipeline` endpoint follows this conceptual model.

---

# 4. The Standard Production Pattern

A production-grade background processing architecture normally separates:

1. **API**
2. **Job metadata**
3. **Queue/orchestrator**
4. **Workers**
5. **External services**
6. **Result storage**
7. **Observability**

```text
                       Client
                         |
                         v
                  +--------------+
                  |   API Layer  |
                  |   FastAPI    |
                  +------+-------+
                         |
                   Create Job
                         |
                         v
                  +--------------+
                  | Queue /      |
                  | Orchestrator |
                  +------+-------+
                         |
                 +-------+-------+
                 |       |       |
                 v       v       v
              Worker  Worker  Worker
                 |       |       |
                 +-------+-------+
                         |
              +----------+----------+
              |                     |
              v                     v
           Vector DB              LLM
              |                     |
              +----------+----------+
                         |
                         v
                  Result Storage
                         |
                         v
                    Job Status
```

The API should not own the actual long-running computation.

---

# 5. Job Lifecycle

A useful generic job state machine is:

```text
             CREATE
                |
                v
            +--------+
            | QUEUED |
            +---+----+
                |
                v
          +-----------+
          | PROCESSING|
          +-----+-----+
                |
          +-----+------+
          |            |
          v            v
     +---------+   +--------+
     |COMPLETED|   | FAILED |
     +---------+   +--------+
```

Additional states may include:

- `CREATED`
- `RETRYING`
- `CANCELLED`
- `PARTIALLY_COMPLETED`

A job record might conceptually contain:

```json
{
  "job_id": "abc-123",
  "status": "PROCESSING",
  "created_at": "...",
  "started_at": "...",
  "completed_at": "...",
  "progress": 65,
  "retry_count": 1,
  "result_location": "..."
}
```

The exact schema depends on the application.

---

# 6. API Contract for Asynchronous Jobs

A common API pattern is:

### Submit

```http
POST /jobs
```

Response:

```http
202 Accepted
```

```json
{
  "job_id": "abc-123",
  "status": "QUEUED"
}
```

`202 Accepted` communicates:

> The request has been accepted for processing, but processing is not complete.

### Poll

```http
GET /jobs/abc-123
```

Response:

```json
{
  "job_id": "abc-123",
  "status": "PROCESSING",
  "progress": 65
}
```

Eventually:

```json
{
  "job_id": "abc-123",
  "status": "COMPLETED",
  "result_location": "..."
}
```

For large outputs, the result should normally be stored outside the API process.

---

# 7. RSA Architecture

## 7.1 Interactive RSA

Endpoint:

```text
POST /rsa/ss6/v1/similar-requirements
```

Purpose:

> Find similar requirements in real time for a single requirement.

Flow:

```text
Client
  |
  v
RSA / SS6
  |
  v
Embed input requirement
  |
  v
SS3 / Context Catalogue
  |
  | Top-K similar candidates
  v
SS1 / LLM
  |
  | Precise similarity scores
  v
Threshold Filtering
  |
  v
Response
```

This should remain synchronous because it is an interactive single-requirement lookup.

---

## 7.2 RSA Batch

Endpoint:

```text
POST /rsa/ss6/v1/pipeline
```

Purpose:

> Trigger processing of all requirements in an uploaded current file.

Flow:

```text
Uploaded Requirements File
            |
            v
      Azure Blob Storage
            |
            v
      POST /pipeline
            |
            v
       RSA / SS6
            |
            v
   Azure Data Factory
            |
     +------+------+
     |             |
     v             v
    SS3           SS3
     |             |
     v             v
    SS1           SS1
     |             |
     +------+------+
            |
            v
      Structured JSON
            |
            v
      Azure Blob Storage
            |
            v
       ZIP Download
```

ADF performs the large-scale batch processing and the client tracks the pipeline through the ADF run ID.

---

# 8. ADF `run_id` as the Job Identifier

The important conceptual mapping is:

```text
RSA batch request
       |
       v
ADF pipeline execution
       |
       v
ADF run_id
```

The client can then use:

```text
GET /pipeline/{run_id}
```

to retrieve the status.

Conceptually:

```text
Client
  |
  | POST /pipeline
  v
RSA
  |
  | Start ADF pipeline
  v
ADF
  |
  | run_id
  v
RSA / Client
  |
  | GET /pipeline/{run_id}
  v
ADF status
```

This is a standard:

> **Submit → Track → Retrieve**

asynchronous pattern.

---

# 9. Why ADF Is a Good Fit for RSA

ADF is designed around data integration and pipeline orchestration.

RSA's batch workload is fundamentally:

```text
Large file
    |
    v
Read records
    |
    v
Process records in parallel
    |
    +--> Vector search
    |
    +--> LLM scoring
    |
    v
Aggregate/write output
```

Therefore ADF is a reasonable fit.

It provides a managed execution/orchestration layer for:

- Large-scale data processing
- Parallel activities
- Azure Blob integration
- Pipeline execution
- Pipeline monitoring
- Integration with external APIs

The key architectural point is:

> ADF is acting as the batch orchestration/execution engine, rather than FastAPI holding an HTTP request open while the batch runs.

---

# 10. Why Blob Storage Is Used

Large inputs and outputs should not normally flow through the API server.

Instead:

```text
Input:

Client
  |
  v
Blob Storage
  |
  v
ADF
```

and:

```text
Output:

ADF
  |
  v
Blob Storage
  |
  v
Client Download
```

This separates:

- API traffic
- Batch processing
- Large object transfer

It also prevents the API service from becoming a bottleneck for large files.

RSA follows this pattern.

---

# 11. Redis + Celery / RQ Alternative

A different architecture is a task queue.

Example:

```text
                    API
                     |
                     | enqueue
                     v
              +-------------+
              |    Redis    |
              | Task Queue  |
              +------+------+
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Worker 1   Worker 2   Worker 3
          |          |          |
          +----------+----------+
                     |
                     v
              Result Storage
```

For Python, common technologies include:

- Celery + Redis
- RQ + Redis

Redis can act as a broker and can also support task/job state depending on the design.

---

# 12. ADF vs Celery/Redis

Neither is universally better.

| Dimension | ADF | Celery + Redis |
|---|---|---|
| Primary abstraction | Data pipeline | Background task |
| Large file/data pipeline | Excellent | Good |
| Azure integration | Excellent | Good |
| Parallel processing | Strong | Strong |
| Fine-grained task control | Moderate | Strong |
| Task priorities | Less natural | Strong |
| Application-level retries | Good | Strong |
| Fine-grained progress | More work | Easier |
| Worker-level control | Managed by platform | Strong |
| Operational overhead | Lower in Azure-native setup | Higher |
| Best RSA fit | Strong | Alternative |

### Interview rule

Use ADF when the problem looks like:

```text
Data
  |
Pipeline
  |
Transform
  |
External services
  |
Output
```

Use a task queue when the problem looks like:

```text
Many independent application tasks
  |
Queue
  |
Workers
  |
Retries / priority / fine-grained control
```

---

# 13. Is RSA Scalable?

Based on the supplied RSA architecture, **yes, the batch architecture is conceptually scalable**.

The key scaling mechanism is parallel processing.

Instead of:

```text
Req1 → Req2 → Req3 → Req4
```

the pipeline can process:

```text
             ADF
              |
       +------+------+------+
       |      |      |      |
      Req1   Req2   Req3   Req4
       |      |      |      |
      SS3    SS3    SS3    SS3
       |      |      |      |
      SS1    SS1    SS1    SS1
```

This reduces elapsed time.

However:

> **More parallelism does not mean unlimited scalability.**

The system is constrained by downstream services.

---

# 14. The Real Bottleneck: Downstream AI Services

RSA depends on:

```text
SS3 → Vector Search
SS1 → LLM Scoring
```

If the batch has:

```text
10,000 requirements
```

and each requirement produces:

```text
5 candidate comparisons
```

then the theoretical number of candidate evaluations can become:

```text
10,000 × 5 = 50,000
```

If each candidate requires an LLM scoring operation, that can become a significant:

- Latency problem
- Cost problem
- Rate-limit problem
- Reliability problem

Therefore scalable AI systems require **controlled concurrency**.

---

# 15. Controlled Concurrency

Bad design:

```text
10,000 requirements
       |
       +---- 10,000 simultaneous LLM calls
```

Potential consequences:

- Rate-limit errors
- API throttling
- Increased latency
- Resource exhaustion
- Retry storms
- Large unexpected costs

Better:

```text
10,000 requirements
       |
       v
Controlled queue
       |
       +--> Worker pool
              |
              +--> N concurrent LLM calls
```

Where `N` is determined by:

- LLM provider limits
- Throughput requirements
- Cost
- Available worker capacity
- Latency requirements

The goal is:

> **Maximum sustainable throughput, not maximum instantaneous concurrency.**

---

# 16. Backpressure

Backpressure is the mechanism used when incoming work is faster than processing capacity.

Example:

```text
Incoming:
1000 jobs/sec

Processing capacity:
100 jobs/sec
```

Without controls:

```text
Queue → 100 → 1000 → 10000 → 100000 → ...
```

Eventually the system becomes unstable.

Backpressure mechanisms include:

- Queue limits
- Rate limiting
- Concurrency limits
- Worker autoscaling
- Admission control
- Request rejection
- Priority queues
- Batch-size controls

For AI workloads, backpressure is especially important because external LLM APIs have quotas and rate limits.

---

# 17. Retry and Exponential Backoff

Long-running jobs depend on external systems, so transient failures are expected.

Example:

```text
Requirement R7231
       |
       v
SS1
       |
    timeout
       |
       v
Retry
```

Typical strategy:

```text
Attempt 1
   |
 failure
   |
 wait 2 sec
   |
Attempt 2
   |
 failure
   |
 wait 4 sec
   |
Attempt 3
```

This is **exponential backoff**.

Adding randomness is called **jitter**:

```text
Base delay + random component
```

Jitter prevents thousands of workers from retrying at exactly the same time.

---

# 18. Retry the Smallest Unit Possible

Suppose a batch contains:

```text
10,000 requirements
```

and requirement `R7231` fails.

Bad behavior:

```text
R7231 fails
    |
    v
Restart all 10,000
```

Better:

```text
R7231 fails
    |
    v
Retry R7231
```

This is **failure isolation**.

For a large production system, task-level retry is preferable where the processing model supports it.

The supplied RSA context does not establish exactly how fine-grained retry is implemented, so this should be treated as a production design principle rather than an RSA implementation claim.

---

# 19. Idempotency

Retries introduce another problem.

Suppose:

```text
Worker
  |
  | SS1 request
  v
SS1
  |
  | Processing succeeded
  |
  X network timeout
```

The worker does not know whether the request actually completed.

It retries.

Now the same operation may execute twice.

Therefore production systems should use an idempotency key, for example:

```text
job_id + requirement_id
```

Example:

```text
abc-123:R7231
```

The principle is:

> **Retries are easy; safe retries require idempotency.**

---

# 20. Partial Failure

For a large batch, one failure should not necessarily destroy all successful work.

Prefer:

```text
10,000 requirements
    |
    +-- 9,950 successful
    +-- 30 retried
    +-- 20 permanently failed
```

rather than:

```text
10,000 requirements
    |
    +-- one failure
    |
    v
Entire batch failed
```

A production result can contain:

```text
Total:      10,000
Successful:  9,950
Failed:         20
Retried:        30
```

Whether RSA currently exposes this exact behavior is not specified by the supplied context.

---

# 21. Progress Tracking

RSA uses polling:

```text
GET /pipeline/{run_id}
```

This is simple and appropriate for many batch workflows.

A more detailed job API might expose:

```json
{
  "status": "PROCESSING",
  "total": 10000,
  "completed": 6500,
  "failed": 12,
  "progress": 65
}
```

Common progress mechanisms:

### Polling

```text
Client → GET status
Client → GET status
Client → GET status
```

### Server-Sent Events

```text
Server ───────> Client
       progress events
```

### WebSocket

```text
Client <──────> Server
```

For RSA, polling is a reasonable design for batch status.

---

# 22. Result Storage

For large batch output:

```text
Worker / ADF
      |
      v
Blob Storage
      |
      v
Download
```

rather than:

```text
Worker
  |
  v
FastAPI
  |
  v
500 MB HTTP response
```

Object storage provides a cleaner separation between:

- Computation
- API
- Large data transfer

RSA writes structured JSON results to Blob Storage and packages the results into a downloadable ZIP.

---

# 23. Production-Grade Background Job Checklist

A mature system should consider these dimensions:

### Job Management

- Job ID
- Persistent job state
- Status transitions
- Job history
- Result location

### Reliability

- Retries
- Exponential backoff
- Jitter
- Idempotency
- Failure isolation
- Dead-letter handling

### Scalability

- Worker scaling
- Controlled concurrency
- Backpressure
- Rate limiting
- Queue depth monitoring

### User Experience

- `202 Accepted`
- Job status API
- Progress information
- Cancellation where needed
- Result download

### Data

- Durable input storage
- Durable result storage
- Large-object handling
- Metadata persistence

### Observability

- Job ID
- Run ID
- Requirement ID
- Trace ID
- Start/end time
- Duration
- Retry count
- Failure reason
- Downstream latency
- LLM token usage
- LLM cost

---

# 24. AI-Specific Scalability Considerations

AI background jobs have an additional dimension: **model cost and token consumption**.

Suppose:

```text
100,000 requirements
      ×
5 candidates
      =
500,000 candidate evaluations
```

Even if the infrastructure can handle it, the LLM cost might become unacceptable.

Therefore AI batch systems should consider:

```text
                AI Batch
                   |
        +----------+----------+
        |          |          |
        v          v          v
   Concurrency   Cost       Accuracy
     control    control      control
        |          |          |
        v          v          v
     Rate limit   Token      Candidate
                  budget      reduction
```

Important techniques include:

- Candidate reduction
- Reranking before LLM
- Model selection
- Batching
- Caching
- Token budgeting
- Rate limiting
- Prompt optimization
- Monitoring cost per batch

---

# 25. Candidate Reduction in RSA

RSA currently follows the conceptual flow:

```text
Requirement
    |
    v
SS3
    |
    v
Top-K candidates
    |
    v
SS1 LLM scoring
```

A potential optimization is:

```text
Requirement
    |
    v
SS3
    |
    v
Top-K
    |
    v
Cheap filtering/reranking
    |
    v
Top-N
    |
    v
SS1 LLM scoring
```

For example:

```text
Top 20 vector candidates
          |
          v
Cheap reranker
          |
          v
Top 3 candidates
          |
          v
LLM scoring
```

This can reduce expensive LLM calls.

However, whether this is appropriate depends on the accuracy requirements and is **not established as part of the supplied RSA implementation**.

---

# 26. Batch LLM Processing

If the model/API supports suitable batching, multiple evaluations can sometimes be combined.

Instead of:

```text
Requirement
  |
  +--> Candidate A → LLM
  +--> Candidate B → LLM
  +--> Candidate C → LLM
```

a batch request might be possible:

```text
Requirement
  |
  v
LLM
  |
  +--> Candidate A score
  +--> Candidate B score
  +--> Candidate C score
```

This can improve throughput and sometimes cost.

But it depends on:

- Model API
- Context window
- Prompt design
- Output format
- Token limits
- Latency characteristics
- Reliability

Do not assume batching is always better.

---

# 27. Caching

Potentially cacheable work includes:

```text
Requirement
   |
   v
Embedding
```

If exactly the same requirement is processed repeatedly, embedding generation may be reused.

For LLM results, caching is more complicated because the output can depend on:

- Model version
- Prompt version
- Context
- System instructions
- Temperature
- Candidate data

Therefore a cache key may need to include these versions/inputs.

---

# 28. Cancellation

A mature job system may support:

```http
POST /jobs/{job_id}/cancel
```

Conceptually:

```text
Job
 |
 +--> Worker 1 → stop
 +--> Worker 2 → stop
 +--> Worker 3 → finish current unit
 +--> Worker 4 → stop
```

Cancellation is easier when the work is divided into independently manageable tasks.

ADF can support pipeline-level management, while task queues can provide more fine-grained task-level control.

The supplied RSA context does not establish task-level cancellation.

---

# 29. Observability

For RSA batch processing, useful metrics/log fields include:

```text
job_id
run_id
requirement_id
trace_id
status
start_time
end_time
duration
retry_count
SS3 latency
SS1 latency
LLM tokens
LLM cost
failure reason
```

This allows troubleshooting questions such as:

> Why did this batch take 2 hours?

You want to be able to decompose:

```text
Total = 120 minutes

SS3          = 20 min
SS1          = 80 min
Retries      = 15 min
Storage      =  5 min
Other        =  0 min
```

This turns observability into an engineering optimization tool rather than merely logging.

---

# 30. A More General Production Architecture

If RSA evolved into a very high-volume application workload, one possible architecture would be:

```text
                         Client
                           |
                           v
                     API Gateway
                           |
                           v
                        RSA API
                           |
                    Create Job Record
                           |
                           v
                       Task Queue
                           |
              +------------+------------+
              |            |            |
              v            v            v
           Worker 1     Worker 2     Worker 3
              |            |            |
              +------------+------------+
                           |
                 +---------+---------+
                 |                   |
                 v                   v
                SS3                 SS1
           Vector Search            LLM
                 |                   |
                 +---------+---------+
                           |
                           v
                     Result Storage
                           |
                           v
                      Blob Storage

                           ^
                           |
                     Job Metadata
                       DB/Redis
```

Potential advantages:

- Fine-grained task management
- Worker scaling
- Priority queues
- Custom retries
- Task-level progress
- Better task isolation

But it also introduces:

- More infrastructure
- More operational complexity
- Queue management
- Worker lifecycle management
- More monitoring
- More failure modes

Therefore it is not automatically superior to ADF.

---

# 31. ADF vs Queue-Based Redesign — Decision Framework

Ask these questions:

### Is the workload primarily a data pipeline?

If yes:

```text
ADF
```

is attractive.

### Are there many independent tasks?

If yes:

```text
Task queue
```

becomes attractive.

### Do you need task priority?

Consider:

```text
Celery/Redis
or managed messaging
```

### Do you need task-level retries?

A task queue may be more natural.

### Do you need rich progress events?

A custom worker/task architecture may provide more control.

### Do you want Azure-managed data orchestration?

ADF is a strong fit.

### Do you want minimal custom infrastructure?

Prefer a managed orchestration service where it meets the requirements.

---

# 32. RSA Architecture Assessment

Based only on the RSA context supplied for interview preparation:

| Area | Assessment |
|---|---|
| Async batch processing | Strong |
| Large file handling | Strong |
| Parallel processing | Strong |
| Azure Blob integration | Strong |
| Job tracking | Strong |
| API responsiveness | Strong |
| Data pipeline fit | Strong |
| Fine-grained task control | Not established |
| Task-level retry | Not established |
| Priority queues | Not established |
| Real-time progress events | Polling |
| Task-level cancellation | Not established |
| Dynamic application worker management | ADF-managed |
| LLM cost optimization | Important area to investigate |

### Overall assessment

**RSA's ADF-based architecture is a sensible and scalable design for its described file-oriented batch workload.**

The architecture should not be criticized simply because Redis/Celery is another possible approach.

The stronger engineering position is:

> Keep ADF if the workload remains a data-processing pipeline. Consider a task-queue architecture if application-level task management becomes the dominant requirement.

---

# 33. What I Would Improve for Larger Scale

If the RSA workload became significantly larger, I would investigate:

### 1. Controlled concurrency

Prevent SS1/SS3 overload.

### 2. Fine-grained retry

Retry individual requirements where appropriate.

### 3. Idempotency

Make retries safe.

### 4. Partial failure

Allow successful records to survive individual failures.

### 5. Dead-letter handling

Separate permanently failed records.

### 6. Progress metrics

Expose:

```text
processed / total
success / failure
```

### 7. LLM optimization

Reduce unnecessary SS1 calls.

### 8. Cost monitoring

Track:

```text
tokens / requirement
cost / batch
cost / successful match
```

### 9. Backpressure

Prevent incoming work from overwhelming downstream services.

### 10. Observability

Trace the complete journey:

```text
Job
  → Requirement
  → SS3
  → SS1
  → Result
```

These are improvement opportunities, not claims about the current RSA implementation.

---

# 34. The Strong Interview Answer

If asked:

> **How did you handle long-running processing in RSA?**

A strong answer is:

> RSA had two execution modes. For interactive similarity search, we had a synchronous endpoint that processed a single requirement through embedding, vector retrieval from SS3, and LLM-based scoring through SS1.
>
> For large requirement files, processing everything synchronously would create long-running HTTP requests and wouldn't scale well. Therefore, we separated the batch workload from the API request. The batch endpoint triggered an Azure Data Factory pipeline and used the ADF run ID to track the asynchronous operation. ADF read the input requirements from Azure Blob Storage, processed them in parallel through SS3 and SS1, and wrote the results back to Blob Storage. The client could poll the RSA status endpoint using the run ID and download the completed result.
>
> I see this as a submit-and-track asynchronous pattern. ADF was a good fit because the workload was essentially a large data-processing pipeline. If the workload later required fine-grained task priorities, task-level retries, dynamic worker management, or richer progress events, I would consider a queue-based architecture such as Celery with Redis or a managed cloud messaging service.

---

# 35. If Asked "Would You Redesign RSA?"

Avoid saying:

> "Yes, I would replace ADF with Redis."

A better answer:

> I wouldn't replace ADF just for the sake of using a task queue. The current architecture is appropriate for a large file-oriented data pipeline. I would first identify the actual bottleneck. If the bottleneck were LLM throttling, I would introduce better concurrency control and rate limiting. If reliability required task-level retries and idempotency, I would improve the task granularity. If the product required priority queues, dynamic worker scaling, or real-time task progress, then I would consider a queue-based architecture.

This demonstrates architectural judgment.

---

# 36. Common Interview Questions

## Fundamentals

### Q: What is a long-running job?

Work that takes longer than a normal request/response lifecycle and therefore should be processed asynchronously.

### Q: Why not keep the HTTP request open?

Because long-lived requests consume resources, can hit infrastructure timeouts, complicate failure handling, and reduce scalability.

### Q: What does HTTP 202 mean?

The server accepted the request for processing, but the operation has not completed.

### Q: What is a job ID?

An identifier that lets the client track the state/result of an asynchronous operation.

---

## Queue / Worker

### Q: What is a task queue?

A mechanism that stores work until workers are available to execute it.

### Q: What is a worker?

A process/service that consumes queued tasks and executes them.

### Q: Why use Redis with Celery?

Redis can provide a fast broker for distributing background tasks to workers and can also support job/task state depending on the architecture.

---

## Reliability

### Q: How do you retry failed jobs?

Use bounded retries with exponential backoff and jitter, especially for transient errors.

### Q: Why use exponential backoff?

To avoid immediately hammering a temporarily unavailable downstream service.

### Q: Why use jitter?

To prevent many workers from retrying simultaneously.

### Q: What is idempotency?

Executing the same operation multiple times should not incorrectly change the final outcome.

### Q: What is a dead-letter queue?

A place to move tasks that repeatedly fail and cannot be processed successfully.

---

## Scalability

### Q: How do you scale workers?

Increase worker instances and/or concurrency, subject to downstream capacity and rate limits.

### Q: Is more concurrency always better?

No. Excessive concurrency can overload databases, vector stores, LLM providers, network resources, and queues.

### Q: What is backpressure?

A mechanism for controlling incoming work when the processing system cannot keep up.

---

# 37. RSA-Specific Questions

### Q: Why is `/similar-requirements` synchronous?

It processes one requirement and is intended for interactive real-time lookup.

### Q: Why is `/pipeline` asynchronous?

It processes an entire requirements file and may take too long for a normal HTTP request.

### Q: Why use ADF?

Because the workload is a large file-oriented data-processing pipeline with parallel processing and Azure storage/API integration.

### Q: What is the ADF run ID?

The identifier for a specific ADF pipeline execution. RSA exposes it to track the long-running operation.

### Q: Why use Blob Storage?

To decouple large input/output files from API processing and avoid moving large payloads through the application server.

### Q: Why not Redis/Celery?

Redis/Celery is a valid alternative, but it is more appropriate when fine-grained application task management is required. ADF is a natural fit for the current data-pipeline-oriented workload.

### Q: What are potential bottlenecks?

Potential bottlenecks include:

- SS1 LLM latency
- LLM rate limits
- LLM cost
- SS3/vector search throughput
- Excessive concurrency
- Retries
- Large batch sizes
- Storage/network throughput

---

# 38. Senior-Level Follow-Up Questions

Be ready for these:

### "How would you scale RSA to 1 million requirements?"

Discuss:

1. Partitioning the input.
2. Controlled parallelism.
3. Queue depth.
4. Worker scaling.
5. SS3 capacity.
6. LLM rate limits.
7. LLM cost.
8. Retry strategy.
9. Idempotency.
10. Partial failure.
11. Result partitioning.
12. Monitoring.

Do not simply say:

> "Add more workers."

---

### "What happens if SS1 goes down?"

A good answer:

```text
SS1 unavailable
     |
     v
Transient failure detection
     |
     v
Bounded retry
     |
 exponential backoff + jitter
     |
     v
Retry
     |
     +--> Success
     |
     +--> Permanent failure
                |
                v
           Dead-letter /
           failed record
```

The exact behavior depends on the orchestration platform.

---

### "What happens if the worker crashes?"

The job must not disappear.

This is why production systems need:

- Durable job state
- Durable queue/orchestrator
- Retry/re-delivery
- Idempotency

---

### "What if the same job is submitted twice?"

Use:

- Request idempotency key
- Job deduplication
- Input/job fingerprinting

For example:

```text
client_request_id
```

or:

```text
hash(input_file + configuration)
```

---

# 39. Rapid Revision Sheet

Remember this:

```text
LONG RUNNING JOB
       |
       v
Don't block HTTP request
       |
       v
Create Job
       |
       v
Return Job ID / 202
       |
       v
Queue / Orchestrator
       |
       v
Worker Pool
       |
       +--> Retry
       +--> Backoff
       +--> Idempotency
       +--> Concurrency Control
       +--> Rate Limiting
       |
       v
Result Storage
       |
       v
Status / Download
```

For RSA:

```text
SINGLE REQUIREMENT
       |
       v
Synchronous API
       |
       +--> Embedding
       +--> SS3
       +--> SS1
       +--> Result


LARGE FILE
       |
       v
Asynchronous API
       |
       v
ADF
       |
       +--> Parallel requirements
       |       |
       |       +--> SS3
       |       +--> SS1
       |
       v
Blob Storage
       |
       v
run_id / polling
       |
       v
Download
```

---

# 40. One-Line Definitions

| Concept | Interview Definition |
|---|---|
| Long-running job | Work that exceeds the practical request/response lifecycle |
| Async processing | Accept work now and execute it later |
| Job ID | Identifier used to track asynchronous work |
| Task queue | Durable mechanism for holding work for workers |
| Worker | Process that executes background tasks |
| Retry | Re-execution after failure |
| Exponential backoff | Increasing delay between retries |
| Jitter | Randomized retry delay |
| Idempotency | Safe repeated execution of an operation |
| Dead-letter queue | Destination for repeatedly failed tasks |
| Backpressure | Control of incoming work when capacity is limited |
| Concurrency control | Limit on simultaneously executing work |
| Partial failure | Some tasks fail while others succeed |
| Polling | Client periodically asks for job status |
| ADF | Azure data integration and pipeline orchestration service |
| Celery | Python distributed task processing framework |
| Redis | In this pattern, commonly used as a task broker/state store |
| Result storage | Durable storage for potentially large job outputs |

---

# 41. Final Mental Model

The most important architecture to remember is:

```text
                INTERACTIVE

Client
  |
  v
/similar-requirements
  |
  v
SS3 + SS1
  |
  v
Response


                BATCH

Client
  |
  v
/pipeline
  |
  v
Create / trigger job
  |
  v
ADF
  |
  +------ Req1 --> SS3 --> SS1
  |
  +------ Req2 --> SS3 --> SS1
  |
  +------ Req3 --> SS3 --> SS1
  |
  +------ ...
  |
  v
Blob Storage
  |
  v
Result

Client
  |
  v
GET /pipeline/{run_id}
  |
  v
Status
```

And the general production architecture:

```text
                  API
                   |
                   v
              Job Created
                   |
                   v
           Queue / Orchestrator
                   |
                   v
              Worker Pool
                   |
        +----------+----------+
        |                     |
        v                     v
      Vector DB              LLM
        |                     |
        +----------+----------+
                   |
                   v
             Result Store
                   |
                   v
              Job Status


     Reliability:
     - Retry
     - Backoff
     - Jitter
     - Idempotency
     - Partial failure
     - Dead-letter handling

     Scalability:
     - Worker scaling
     - Concurrency control
     - Backpressure
     - Rate limiting

     AI concerns:
     - Token usage
     - LLM cost
     - Model limits
     - Candidate reduction
     - Caching

     Operations:
     - Logging
     - Metrics
     - Tracing
     - Job monitoring
```

## Final Interview Principle

> **The architectural goal is not simply to run work in the background. The goal is to decouple request handling from execution while making the work durable, scalable, observable, retryable, and cost-controlled.**

For RSA specifically:

> **The ADF-based implementation is a sensible architecture for a large, file-oriented batch pipeline. A queue/worker architecture such as Celery + Redis would become more attractive if RSA evolved toward fine-grained application tasks requiring task-level retries, priorities, dynamic workers, richer progress tracking, or more granular control.**
