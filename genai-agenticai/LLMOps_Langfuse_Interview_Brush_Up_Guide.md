# LLMOps with Langfuse — Interview Brush-Up Guide

**High-Level Understanding, Architecture & Interview Preparation**

> **Goal:** Build enough LLMOps + Langfuse understanding to explain the concepts clearly in an AI/ML or GenAI interview, especially for production LLM applications, RAG systems, and agentic workflows.

> **Source basis:** Langfuse official documentation and Langfuse's Product Engineering Architecture documentation. Structure and interview-oriented style are aligned with the RAG and MCP interview brush-up guides.

---

## Contents

1. LLMOps at a Glance
2. Why LLMOps Is Needed
3. Where Langfuse Fits
4. Langfuse Core Capabilities
5. End-to-End LLMOps Lifecycle
6. Observability: The Most Important Concept
7. Langfuse Data Model — Sessions, Traces and Observations
8. Observation Types
9. Example: Tracing a RAG Application
10. Example: Tracing an Agent
11. Prompt Management
12. Prompt Versions and Labels
13. Prompt Caching
14. Linking Prompts to Traces
15. Evaluation
16. Scores
17. Offline Evaluation — Datasets and Experiments
18. Online Evaluation
19. LLM-as-a-Judge vs Code Evaluators vs Human Evaluation
20. Cost, Token Usage and Latency
21. Metrics and Dashboards
22. Langfuse Platform Architecture
23. Why ClickHouse + PostgreSQL?
24. Ingestion Architecture
25. Production Deployment / Self-Hosting
26. OpenTelemetry and Integrations
27. LLMOps Architecture for an Enterprise Application
28. Development → Production Workflow
29. What Langfuse Is Not
30. One-Line Definitions for Fast Revision
31. Interview Questions You Should Be Ready For
32. Scenario Questions
33. Final Interview Summary
34. Official References

---

# 1. LLMOps at a Glance

**LLMOps** is the set of engineering practices used to develop, evaluate, deploy, observe, and continuously improve applications built with large language models.

Traditional ML systems already need:

```text
Training
Deployment
Monitoring
Evaluation
Versioning
```

LLM applications add additional challenges:

```text
Prompt changes
Model changes
Non-deterministic outputs
Token usage
LLM cost
Long context
Tool calls
RAG retrieval
Agent workflows
Human feedback
Quality evaluation
```

A useful mental model is:

```text
                 LLM Application
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
   Observability   Evaluation   Prompt Management
        |              |              |
        +--------------+--------------+
                       |
                       v
                 Continuous
                  Improvement
```

### Interview answer

> "LLMOps is the operational discipline for building and running LLM applications reliably. It covers observability, prompt and model versioning, evaluation, cost and latency monitoring, production monitoring, and continuous improvement."

---

# 2. Why LLMOps Is Needed

An LLM application can fail in many different ways.

For example:

```text
User Question
      |
      v
   Retriever
      |
      v
Wrong Context
      |
      v
     LLM
      |
      v
Wrong Answer
```

Or:

```text
Prompt Version 7
      |
      v
LLM
      |
      v
Higher Token Usage
      |
      v
Higher Cost
```

Or:

```text
Agent
 |
 +--> LLM call
 |
 +--> Tool call
 |
 +--> Retrieval
 |
 +--> LLM call
 |
 +--> Final answer
```

Without tracing, it is difficult to determine:

- Which model was called?
- Which prompt version was used?
- What context was retrieved?
- Which tool was called?
- How long did each step take?
- How many tokens were consumed?
- How much did the request cost?
- Where did quality degrade?

LLMOps provides the infrastructure to answer these questions systematically.

---

# 3. Where Langfuse Fits

Langfuse is an **open-source AI engineering / LLM observability platform** that helps teams debug, analyze, evaluate, and iterate on LLM applications.

Its major capabilities are:

```text
                    Langfuse
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
 Observability    Prompt Mgmt      Evaluation
       |               |               |
       v               v               v
 Traces          Versions         Scores
 Sessions        Labels           Experiments
 Costs           Playground       LLM-as-Judge
 Latency         Deployment       Human feedback
```

Langfuse is API-first, supports native SDKs and integrations, and is based on OpenTelemetry.

### Interview answer

> "I think of Langfuse as an LLM engineering control plane for observability, prompt management, and evaluation. It gives us visibility into the execution of an LLM application and connects that runtime data with prompts, evaluations, costs, and experiments."

---

# 4. Langfuse Core Capabilities

The official documentation groups the main capabilities into:

### Observability

- Trace LLM and non-LLM operations.
- Inspect prompts and responses.
- Track retrieval and tool calls.
- Analyze latency.
- Track token usage and cost.
- Follow multi-turn sessions.
- Visualize agent workflows.

### Prompt Management

- Store prompts centrally.
- Version prompts.
- Assign labels such as `production` or `staging`.
- Test prompts in a playground.
- Link prompts to traces.
- Compare prompt performance.

### Evaluation

- LLM-as-a-Judge.
- Code evaluators.
- Human annotations.
- User feedback.
- Custom scores.
- Datasets.
- Experiments.
- Online production evaluation.

### Platform

- API-first architecture.
- Open source.
- Self-hosting.
- OpenTelemetry compatibility.
- Data export capabilities.
- Enterprise security features.

---

# 5. End-to-End LLMOps Lifecycle

A useful interview mental model is:

```text
                 DEVELOPMENT
                     |
             Prompt / Model Design
                     |
                     v
              Offline Evaluation
                     |
                     v
                  Deploy
                     |
                     v
                PRODUCTION
                     |
             Online Observability
                     |
        +------------+------------+
        |            |            |
        v            v            v
      Quality      Cost        Latency
        |
        v
     Feedback
        |
        v
   Dataset / Test Cases
        |
        v
   New Experiment
        |
        v
     Improvement
```

This is the core LLMOps feedback loop.

### Strong interview framing

> "The important part of LLMOps is not just logging production requests. It is closing the loop between production observability, evaluation datasets, experimentation, and controlled deployment."

---

# 6. Observability: The Most Important Concept

Traditional application logging might tell us:

```text
Request received
Request completed
HTTP 200
```

That is not enough for an LLM application.

We want:

```text
User request
   |
   +--> Prompt version
   |
   +--> Retrieval
   |      |
   |      +--> Retrieved documents
   |
   +--> LLM generation
   |      |
   |      +--> Model
   |      +--> Tokens
   |      +--> Cost
   |      +--> Latency
   |
   +--> Tool calls
   |
   v
Final response
```

Langfuse describes tracing as a structured representation of the execution flow, including LLM and non-LLM operations.

### Why tracing is especially important for LLM applications

LLM applications are often:

- Non-deterministic.
- Multi-step.
- Tool-using.
- Retrieval-augmented.
- Prompt-sensitive.
- Cost-sensitive.

Tracing lets us reconstruct **why a response happened**.

---

# 7. Langfuse Data Model — Sessions, Traces and Observations

These three concepts are essential.

## Session

A **session** groups multiple related traces.

Example:

```text
User conversation
      |
      +--> Trace 1: "What is my balance?"
      |
      +--> Trace 2: "What about last month?"
      |
      +--> Trace 3: "Explain the difference."
```

Useful for multi-turn applications.

## Trace

A **trace** represents a single request or operation.

Example:

```text
User asks:
"What is the refund policy?"
```

That complete request can be represented by one trace.

## Observation

An **observation** represents an individual step inside a trace.

For example:

```text
Trace
 |
 +--> Retrieval
 |
 +--> LLM generation
 |
 +--> Tool call
 |
 +--> Final generation
```

### Mental model

```text
Session
   |
   +--> Trace
         |
         +--> Observation
         +--> Observation
         +--> Observation
```

### Interview answer

> "A session groups related interactions, a trace represents one end-to-end request or operation, and observations represent the individual steps within that trace, such as LLM generations, retrieval, tool calls, or other application work."

---

# 8. Observation Types

Langfuse supports different observation types so that traces can represent the semantics of an application.

Important types include:

| Observation | Meaning |
|---|---|
| `event` | Discrete event |
| `span` | Duration-based unit of work |
| `generation` | LLM generation, including model, prompt, usage and cost |
| `agent` | Agentic decision / orchestration |
| `tool` | Tool or API action |
| `chain` | Link between application steps |
| `retriever` | Retrieval operation |
| `evaluator` | Evaluation operation |

The most important interview distinction is:

```text
LLM call       -> generation
Tool call      -> tool
Retrieval      -> retriever
Agent workflow -> agent
Generic work   -> span
```

Correct observation types make filtering, debugging, evaluation, and dashboards more useful.

---

# 9. Example: Tracing a RAG Application

Suppose we have:

```text
User
 |
 v
RAG Application
 |
 +--> Query embedding
 |
 +--> Vector search
 |
 +--> Reranking
 |
 +--> LLM
 |
 v
Answer
```

A Langfuse trace can represent the flow as:

```text
Trace: "What is our leave policy?"
 |
 +--> Retriever
 |      |
 |      +--> Vector DB
 |      +--> Top-K documents
 |
 +--> Generation
        |
        +--> Prompt version: HR-QA / production
        +--> Model: <LLM>
        +--> Input tokens
        +--> Output tokens
        +--> Latency
        +--> Cost
```

This allows us to investigate:

> "The answer was wrong — was the problem retrieval, the prompt, or the model?"

That is one of the strongest practical reasons to use LLM observability.

---

# 10. Example: Tracing an Agent

Agentic systems are even more complex.

Example:

```text
User
 |
 v
Agent
 |
 +--> Generation: decide next action
 |
 +--> Tool: search customer
 |
 +--> Tool: retrieve order
 |
 +--> Generation: reason over results
 |
 +--> Tool: create ticket
 |
 +--> Generation: final response
```

A trace tree can expose the complete workflow.

A good trace should preserve the parent-child relationships between operations.

### Interview answer

> "For an agent, tracing is particularly valuable because the application may dynamically choose tools and execute multiple model calls. A hierarchical trace lets us see which generation caused which tool call and how the final answer was produced."

---

# 11. Prompt Management

Prompt management means treating prompts as **production artifacts**, rather than hardcoding them inside application code.

Instead of hardcoding a prompt in Python, manage it centrally:

```text
Langfuse Prompt Management
          |
          +--> Prompt: customer-support
          |
          +--> Version 1
          +--> Version 2
          +--> Version 3
```

Benefits:

- Centralized management.
- Version history.
- Collaboration.
- Controlled deployment.
- Prompt experimentation.
- Rollback.
- Traceability.

### Interview answer

> "Prompt management separates prompt lifecycle from application code. Instead of deploying application code for every prompt change, prompts can be versioned and deployed independently."

---

# 12. Prompt Versions and Labels

This is an important Langfuse-specific concept.

### Version

Each prompt update creates an immutable version.

```text
customer-support
   |
   +--> v1
   +--> v2
   +--> v3
```

### Label

A label points to a specific version.

For example:

```text
production -> v3
staging    -> v4
latest     -> v5
```

The application can request:

```text
prompt = "customer-support"
label  = "production"
```

rather than hardcoding a numeric version.

### Why labels are powerful

Deployment becomes:

```text
Before:

production -> v3

After:

production -> v4
```

No application code change is required.

Rollback:

```text
production -> v3
```

### Interview answer

> "Versions provide immutable prompt history, while labels are deployment pointers. I would normally have application code reference a stable label such as `production`, allowing prompt versions to be promoted or rolled back without changing application code."

---

# 13. Prompt Caching

Prompt management introduces a potential runtime dependency:

```text
Application
     |
     v
Langfuse
     |
     v
Prompt
```

If every request requires a network call to fetch the prompt, it could add latency and create an availability dependency.

Langfuse addresses this with client-side caching.

Conceptually:

```text
Application
    |
    v
SDK local cache
    |
    +--> Cache hit -> prompt immediately
    |
    +--> Cache miss -> Langfuse API
```

Langfuse documents a default SDK cache TTL of 60 seconds and supports background revalidation.

It can also support fallback behavior.

### Interview answer

> "Prompt retrieval should not become a synchronous dependency on every request. Langfuse SDKs cache prompts locally, which reduces latency and improves availability."

---

# 14. Linking Prompts to Traces

Prompt management becomes much more useful when prompt versions are connected to runtime traces.

Example:

```text
Prompt
customer-support
      |
      +--> v1
      +--> v2
      +--> v3
             |
             v
          Traces
             |
       +-----+-----+
       |           |
     Cost        Quality
       |           |
    Latency      Scores
```

This allows questions such as:

> "Did prompt version 3 improve answer quality?"

or:

> "Which prompt version is producing higher cost?"

The official Langfuse documentation describes linking prompts and traces as the basis for tracking metrics and evaluations per prompt version.

---

# 15. Evaluation

Observability tells us:

> **What happened?**

Evaluation tells us:

> **Was the result good?**

This distinction is critical.

```text
Observability
     |
     v
What happened?
     |
     v
Evaluation
     |
     v
Was it good?
```

Examples of evaluation dimensions:

- Correctness
- Relevance
- Helpfulness
- Groundedness
- Toxicity
- Format compliance
- Retrieval relevance
- Business-specific quality

---

# 16. Scores

Langfuse uses **scores** as the common data object for evaluation results.

A score has:

```text
Name
Value
Data type
Optional comment
```

Examples:

```text
correctness = 0.92
helpfulness = 0.80
groundedness = true
classification = "correct"
```

Scores can be attached to:

- Traces
- Observations
- Sessions
- Dataset runs

Supported score types include:

```text
NUMERIC
CATEGORICAL
BOOLEAN
TEXT
```

### Interview answer

> "Langfuse uses scores as a universal evaluation object. Scores can come from human feedback, LLM-as-a-Judge, deterministic code evaluators, or external evaluation pipelines."

---

# 17. Offline Evaluation — Datasets and Experiments

Before deploying a new prompt or model, we should test it against a fixed test set.

Example:

```text
Dataset
 |
 +--> Test case 1
 +--> Test case 2
 +--> Test case 3
 +--> ...
 +--> Test case 100
```

Run:

```text
Prompt v1 + Model A
          |
          v
       Scores

Prompt v2 + Model A
          |
          v
       Scores
```

Compare the results.

### Langfuse experiment model

```text
Dataset
   |
   v
Experiment
   |
   +--> Application / Prompt / Model
   |
   v
Outputs
   |
   v
Evaluators
   |
   v
Scores
   |
   v
Comparison
```

### Why this matters

It gives us reproducible evaluation before production deployment.

### Interview answer

> "I would maintain a representative evaluation dataset and run experiments when changing prompts, models, or application logic. The goal is to detect regressions before production."

---

# 18. Online Evaluation

Offline evaluation is not enough.

Production traffic contains:

- Unexpected inputs.
- Edge cases.
- New user behavior.
- New domains.
- Distribution shifts.

Therefore:

```text
Production Traffic
       |
       v
     Traces
       |
       v
Online Evaluators
       |
       v
     Scores
       |
       v
Dashboards / Alerts
```

For example:

```text
Production generation
       |
       v
LLM-as-Judge
       |
       v
Groundedness score
       |
       v
Dashboard
```

This can help detect quality degradation in live traffic.

---

# 19. LLM-as-a-Judge vs Code Evaluators vs Human Evaluation

### LLM-as-a-Judge

Use another LLM to evaluate the output.

```text
Question
Answer
Reference
   |
   v
Judge LLM
   |
   v
Correctness score
```

Good for:

- Helpfulness.
- Relevance.
- Tone.
- Quality.
- Semantic correctness.

### Code evaluator

Use deterministic logic.

```python
score = 1 if output_is_valid_json else 0
```

Good for:

- JSON schema validation.
- Exact matching.
- Business rules.
- Structured output.
- Deterministic checks.

### Human evaluation

A person reviews the output.

Good for:

- Building ground truth.
- Subjective quality.
- Difficult edge cases.
- Validating automated evaluators.

### Strong interview answer

> "I would use code evaluators wherever the criterion is deterministic, LLM-as-a-Judge for subjective semantic quality at scale, and human evaluation for ground-truth creation and cases where automated evaluation is not sufficiently trustworthy."

---

# 20. Cost, Token Usage and Latency

LLM applications have three important operational dimensions:

```text
Quality
Cost
Latency
```

A system can have high quality but still be unacceptable if it is too expensive or slow.

Langfuse tracks usage and costs on generation and embedding observations.

Example:

```text
Trace
 |
 +--> Generation
 |      |
 |      +--> Model
 |      +--> Input tokens
 |      +--> Output tokens
 |      +--> Cost
 |      +--> Latency
 |
 +--> Generation
        |
        +--> Input tokens
        +--> Output tokens
        +--> Cost
```

This allows analysis such as:

```text
Cost by model
Cost by user
Cost by application
Cost over time
Token usage over time
Latency by operation
```

### Interview answer

> "I would monitor quality together with latency and cost. In LLM systems, optimizing only for response quality can produce an operationally expensive or slow system."

---

# 21. Metrics and Dashboards

Once traces and scores are available, we can aggregate them.

Examples:

```text
Requests / hour
Average latency
P95 latency
Token usage
LLM cost
Error rate
Quality score
Groundedness
Tool failures
Retrieval quality
```

A useful dashboard:

```text
             LLM Application Dashboard

Requests       Cost          Latency       Quality
  10K          $120          1.8 sec        0.91
   |             |              |              |
   +-------------+--------------+--------------+
                         |
                         v
                    Drill-down
                         |
                         v
                       Trace
                         |
                         v
                 Prompt / Model /
              Retrieval / Tool calls
```

The key point is that aggregated metrics should allow us to drill into individual traces.

---

# 22. Langfuse Platform Architecture

The official Langfuse architecture consists of:

```text
                    LLM Application
                           |
                    SDK / OTel / API
                           |
                           v
                  +-------------------+
                  |    Langfuse Web   |
                  |   UI + APIs       |
                  +---------+---------+
                            |
                            v
                     Redis / Valkey
                       Queue/Cache
                            |
                            v
                  +-------------------+
                  | Langfuse Worker   |
                  | Async Processing   |
                  +---------+---------+
                            |
             +--------------+--------------+
             |                             |
             v                             v
       PostgreSQL                      ClickHouse
     Transactional data              Analytics data
             |                             |
             |                             |
             +-------------+---------------+
                           |
                           v
                         S3 /
                      Blob Storage
```

There can also be an optional external LLM API / gateway for features such as LLM-based evaluations or playground functionality.

---

# 23. Why ClickHouse + PostgreSQL?

This is a strong architecture interview question.

Langfuse uses **PostgreSQL for transactional workloads**.

Examples:

```text
Users
Organizations
Projects
API keys
Prompts
Datasets
Configuration
```

Langfuse uses **ClickHouse for analytical / observability workloads**.

Examples:

```text
Traces
Observations
Scores
Dashboards
Metrics
```

Why not put everything in PostgreSQL?

Observability data can become extremely large and query-heavy.

Typical queries are analytical:

```text
Cost over time
Average latency by model
Token usage by user
Quality score by prompt version
Trace volume by day
```

ClickHouse is designed for high-performance analytical workloads and columnar storage.

### Interview answer

> "The separation is workload-driven. PostgreSQL handles transactional application data, while ClickHouse handles large-scale analytical observability data such as traces, observations, and scores."

---

# 24. Ingestion Architecture

Langfuse SDKs instrument the application.

Conceptually:

```text
Application
    |
    | trace / observation events
    v
Langfuse API
    |
    +--> S3 / Blob Store
    |
    +--> Redis Queue
              |
              v
        Langfuse Worker
              |
              v
          ClickHouse
```

The important architectural idea is **asynchronous ingestion**.

The application should not have to wait for all observability processing to complete before returning the user response.

### Why use a queue?

```text
Application
     |
     v
Ingestion API
     |
     v
Queue
     |
     v
Async worker
```

Benefits:

- Decouples ingestion from processing.
- Smooths traffic spikes.
- Allows asynchronous enrichment and processing.
- Reduces coupling between user request latency and analytics processing.

### Interview answer

> "The observability pipeline should be decoupled from the application's critical request path. Langfuse uses an ingestion API, queueing, and asynchronous workers before analytical data is processed into ClickHouse."

---

# 25. Production Deployment / Self-Hosting

Langfuse is open source and can be deployed:

```text
Local
  |
Docker Compose

Cloud
  |
Kubernetes
AWS
Azure
GCP

Enterprise
  |
VPC
On-premises
```

The official architecture states that Langfuse can be deployed within a VPC or on-premises for high-security environments.

For production-scale self-hosting, Langfuse documents deployment options including Kubernetes/Helm and Terraform-based AWS, Azure, and GCP deployments.

Typical enterprise architecture:

```text
                 Load Balancer
                       |
                       v
                  Langfuse Web
                       |
               +-------+--------+
               |                |
               v                v
           PostgreSQL      Redis/Valkey
                                |
                                v
                              Worker
                                |
                                v
                           ClickHouse
                                |
                                v
                                S3
```

---

# 26. OpenTelemetry and Integrations

Langfuse is based on **OpenTelemetry**.

This matters because OpenTelemetry is a widely used standard for telemetry.

Langfuse supports:

- Native Python SDK.
- Native JS/TS SDK.
- OpenTelemetry.
- OpenAI integrations.
- LangChain.
- LangGraph.
- LlamaIndex.
- LiteLLM.
- Many other frameworks and AI tools.

### Interview answer

> "The OpenTelemetry foundation is important because it reduces vendor lock-in and allows Langfuse to fit into a broader observability ecosystem."

---

# 27. LLMOps Architecture for an Enterprise Application

A practical architecture might look like:

```text
                         User
                           |
                           v
                      Web / API
                           |
                           v
                  LLM Application
                  /      |                        /       |                        v        v         v
           Retriever    Agent     Prompt
                |         |       Management
                |         |
                v         v
            Vector DB   Tools/APIs
                |         |
                +----+----+
                     |
                     v
                    LLM
                     |
                     v
                 Response
                     |
                     v
                Langfuse SDK
                     |
                     v
              +-------------+
              |   Langfuse  |
              +-------------+
              | Observability|
              | Prompts      |
              | Evaluation   |
              | Metrics      |
              +-------------+
```

The important architectural principle:

> Langfuse is **outside the business execution path conceptually**, but it observes and evaluates the business execution path.

---

# 28. Development → Production Workflow

A strong LLMOps workflow looks like this:

```text
1. Create prompt
       |
       v
2. Test in Playground
       |
       v
3. Build evaluation dataset
       |
       v
4. Run experiment
       |
       v
5. Compare prompt/model versions
       |
       v
6. Promote version to staging
       |
       v
7. Run application tests
       |
       v
8. Promote to production
       |
       v
9. Observe production traces
       |
       v
10. Online evaluation
       |
       v
11. Capture failures / feedback
       |
       v
12. Add cases to dataset
       |
       +--------> New experiment
```

This is the most important LLMOps lifecycle to understand.

---

# 29. What Langfuse Is Not

### Langfuse is not an LLM

It does not generate your application's answers.

```text
Application -> LLM
       |
       +--> Langfuse observes
```

### Langfuse is not a vector database

It can observe retrieval operations, but it is not the primary vector store for your RAG application.

### Langfuse is not an agent framework

It can trace agent workflows, but it does not replace LangGraph or another orchestration framework.

### Langfuse is not just logging

Traditional logging:

```text
"LLM call failed"
```

LLM observability:

```text
Trace
 |
 +--> Prompt version
 +--> Retrieved context
 +--> Model
 +--> Tokens
 +--> Cost
 +--> Latency
 +--> Tool calls
 +--> Evaluation score
```

This is a much richer debugging model.

---

# 30. One-Line Definitions for Fast Revision

**LLMOps:** Engineering discipline for operating and continuously improving LLM applications.

**LLM Observability:** Visibility into the execution and behavior of an LLM application.

**Trace:** One end-to-end request or operation.

**Session:** Group of related traces, commonly representing a conversation.

**Observation:** Individual operation inside a trace.

**Generation:** Observation representing an LLM generation.

**Tool:** Observation representing a tool/API action.

**Retriever:** Observation representing a retrieval step.

**Prompt Management:** Centralized storage, versioning, deployment and testing of prompts.

**Prompt Version:** Immutable version of a prompt.

**Prompt Label:** Pointer to a prompt version, such as `production`.

**Prompt Cache:** Client-side caching of prompts to reduce latency and availability dependency.

**Score:** Langfuse object representing an evaluation result.

**Dataset:** Collection of evaluation test cases.

**Experiment:** Running an application against a dataset to compare outputs and scores.

**LLM-as-a-Judge:** Using an LLM to evaluate another LLM application's output.

**Code Evaluator:** Deterministic evaluator implemented in code.

**Online Evaluation:** Evaluating production traffic.

**Offline Evaluation:** Evaluating a controlled dataset before deployment.

**ClickHouse:** Analytical database used by Langfuse for tracing and observability data.

**PostgreSQL:** Transactional database used for Langfuse application/configuration data.

**Redis/Valkey:** Queue/cache component used in Langfuse architecture.

**S3/Blob Storage:** Object storage for incoming events, multimodal data and large exports.

**OpenTelemetry:** Open standard used by Langfuse for telemetry interoperability.

---

# 31. Interview Questions You Should Be Ready For

## Fundamentals

- What is LLMOps?
- Why do LLM applications need LLMOps?
- What problems does Langfuse solve?
- How is LLM observability different from traditional application logging?
- Where does Langfuse fit in an LLM architecture?
- Is Langfuse an LLM?
- Is Langfuse an agent framework?

## Observability

- What is a trace?
- What is a session?
- What is an observation?
- What is a generation?
- How would you trace a RAG pipeline?
- How would you trace an agent?
- How do you monitor token usage?
- How do you monitor LLM cost?
- How do you investigate latency?

## Prompt Management

- Why manage prompts outside application code?
- What is prompt versioning?
- What is the difference between a prompt version and a label?
- How would you deploy a new prompt without changing application code?
- How would you rollback a prompt?
- Why is prompt caching needed?
- How do prompts get linked to traces?

## Evaluation

- What is the difference between observability and evaluation?
- What is a score?
- What is an evaluation dataset?
- What is an experiment?
- What is offline evaluation?
- What is online evaluation?
- What is LLM-as-a-Judge?
- When would you use a code evaluator?
- When would you use human evaluation?
- How would you detect quality degradation in production?

## Architecture

- Explain Langfuse's architecture.
- Why does Langfuse use ClickHouse?
- Why is PostgreSQL separate from ClickHouse?
- Why is Redis/Valkey used?
- Why is ingestion asynchronous?
- How does Langfuse scale?
- How would you self-host Langfuse?
- How does OpenTelemetry fit into Langfuse?

---

# 32. Scenario Questions

## Scenario 1 — Production answer quality dropped

**Question:**

> "Users report that answer quality has dropped after a deployment. How would you investigate?"

### Good answer

```text
1. Identify affected traces.
2. Compare recent vs previous traces.
3. Check prompt version.
4. Check model version.
5. Inspect retrieved context.
6. Check tool calls if applicable.
7. Check latency and token usage.
8. Check evaluation scores.
9. Compare against offline evaluation dataset.
10. Roll back prompt/model if required.
```

The key is to use the trace to identify **where** the degradation occurred.

---

## Scenario 2 — Cost suddenly increased

```text
Dashboard
   |
   v
Cost increase
   |
   +--> By model?
   +--> By user?
   +--> By application?
   +--> By prompt version?
   +--> Input tokens?
   +--> Output tokens?
   +--> Number of LLM calls?
```

Then drill down into traces.

Possible causes:

- Larger prompts.
- More retrieved context.
- Longer conversations.
- More agent iterations.
- Model change.
- Prompt regression.

---

## Scenario 3 — New prompt version

Suppose:

```text
Production -> prompt v7
New version -> v8
```

A good workflow:

```text
v8
 |
 v
Dataset
 |
 v
Experiment
 |
 v
Quality / Cost / Latency
 |
 +--> Better -> promote
 |
 +--> Worse -> do not promote
```

This is much safer than immediately deploying the new prompt.

---

## Scenario 4 — RAG answer is wrong

Use tracing to separate the problem:

```text
Question
   |
   v
Retriever
   |
   +--> Wrong documents?
   |       |
   |       +--> Retrieval problem
   |
   v
Context
   |
   v
Prompt
   |
   v
LLM
   |
   +--> Incorrect reasoning?
```

This is one of the most valuable applications of LLM observability.

---

## Scenario 5 — Design an enterprise LLMOps platform

A good high-level answer:

```text
                    Enterprise AI Apps
                     /      |                           /       |                         RAG      Agents    Assistants
                   \        |        /
                    \       |       /
                     v      v      v
                   Langfuse SDK / OTel
                           |
                           v
                    Langfuse Platform
                    /       |                          /        |                  Observability  Prompt Mgmt  Evaluation
                |             |            |
                v             v            v
             Traces        Versions      Scores
             Costs         Labels        Datasets
             Latency       Deployment    Experiments
```

For enterprise deployment:

```text
Load Balancer
      |
Langfuse Web
      |
+-----+---------+---------+
|               |         |
Postgres      Redis    Worker
                           |
                           v
                       ClickHouse
                           |
                           v
                           S3
```

---

# 33. Final Interview Summary

The simplest way to explain Langfuse is:

> **"Langfuse provides observability, prompt management, and evaluation capabilities for LLM applications."**

Then explain the lifecycle:

```text
BUILD
 |
Prompt + Model
 |
 v
EVALUATE
 |
Dataset + Experiment
 |
 v
DEPLOY
 |
Prompt Label / Production
 |
 v
OBSERVE
 |
Traces + Cost + Latency
 |
 v
EVALUATE
 |
Online Scores
 |
 v
LEARN
 |
Feedback + Production failures
 |
 v
IMPROVE
 |
New Prompt / Model
 |
 +---------> Experiment
```

### The three pillars to remember

```text
              LANGFUSE
                 |
       +---------+---------+
       |         |         |
       v         v         v
 Observability Prompts  Evaluation
       |         |         |
     Trace     Version    Score
     Cost      Label      Dataset
     Latency   Deploy     Experiment
```

### Strong interview answer

> "For an enterprise LLM application, I would use Langfuse as the LLMOps layer around the application. I would instrument the application so that every important execution path is represented as a trace with observations for generations, retrievals and tool calls. I would manage prompts centrally with immutable versions and environment-oriented labels. Before deployment, I would use datasets and experiments to compare prompt or model changes. In production, I would monitor traces, token usage, cost, latency and online evaluation scores. Production failures and user feedback would then feed back into the evaluation dataset, creating a continuous improvement loop."

---

# 34. Official References

1. Langfuse Documentation  
   https://langfuse.com/docs

2. Langfuse Platform Architecture  
   https://langfuse.com/handbook/product-engineering/architecture

3. Observability  
   https://langfuse.com/docs/observability/overview

4. Observability Data Model  
   https://langfuse.com/docs/observability/data-model

5. Prompt Management  
   https://langfuse.com/docs/prompt-management/overview

6. Prompt Version Control  
   https://langfuse.com/docs/prompt-management/features/prompt-version-control

7. Evaluation Core Concepts  
   https://langfuse.com/docs/evaluation/core-concepts

8. LLM-as-a-Judge  
   https://langfuse.com/docs/evaluation/evaluation-methods/llm-as-a-judge

9. Token and Cost Tracking  
   https://langfuse.com/docs/observability/features/token-and-cost-tracking

10. Self Hosting  
    https://langfuse.com/self-hosting

11. Langfuse Integrations  
    https://langfuse.com/integrations

---

# Quick Revision Card

```text
LLMOps
= Operate + evaluate + improve LLM applications

LANGFUSE
= Observability + Prompt Management + Evaluation

TRACE
= One request / operation

SESSION
= Group of related traces

OBSERVATION
= Individual step

GENERATION
= LLM call

TOOL
= Tool/API call

RETRIEVER
= Retrieval operation

PROMPT VERSION
= Immutable prompt history

LABEL
= Deployment pointer to a version

SCORE
= Evaluation result

DATASET
= Test cases

EXPERIMENT
= Run application against dataset

LLM-AS-JUDGE
= LLM evaluates output

ONLINE EVAL
= Evaluate production traffic

OFFLINE EVAL
= Evaluate controlled dataset

CLICKHOUSE
= Analytical observability store

POSTGRES
= Transactional application store

REDIS/VALKEY
= Queue + cache

S3
= Raw events / multimodal / exports

OTEL
= Open telemetry standard

CORE LLMOPS LOOP
= Experiment → Deploy → Observe → Evaluate → Improve
```

## Final Advice

For interviews, do not present Langfuse as just a "logging tool."

The stronger framing is:

> **"Langfuse connects the runtime, prompt, evaluation, and improvement sides of an LLM application."**

If you can clearly explain:

1. **Trace → Observation → Session**
2. **Prompt → Version → Label → Trace**
3. **Dataset → Experiment → Score**
4. **Production Trace → Online Evaluation**
5. **Application → Langfuse API → Redis → Worker → ClickHouse/Postgres/S3**

you have the core Langfuse/LLMOps understanding needed for a high-level AI Engineer / AI Architect interview.
