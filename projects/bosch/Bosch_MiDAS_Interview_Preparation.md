# Bosch Interview Preparation --- MiDAS / GenAI Platform

## 1. Bosch Story --- 30 Seconds

> I worked on **MiDAS**, a microservices-based enterprise GenAI platform
> for automating software engineering workflows across the V-Model
> lifecycle. My work was primarily around **platform engineering and
> reusable GenAI capabilities**. My main contribution was architecting
> and developing the **LLMOps prompt-management capability using
> Langfuse**, exposed through reusable REST APIs. I also developed GenAI
> solutions using **RAG and prompt engineering** on top of the MiDAS
> platform.

**Business impact:** projected \~**USD 650K annual productivity/cost
savings**.

------------------------------------------------------------------------

## 2. MiDAS --- One Sentence

> **MiDAS is a reusable enterprise AI platform providing common LLM,
> retrieval, orchestration, LLMOps, integration, security and analytics
> capabilities for Bosch software engineering.**

### Why a platform?

-   Avoid duplicated AI infrastructure
-   Standardize APIs and engineering patterns
-   Centralize security/governance
-   Improve developer productivity
-   Enable independent scaling
-   Improve observability and operational maturity

**Key distinction:** MiDAS = **platform**; RSA/other solutions = **use
cases**.

------------------------------------------------------------------------

## 3. Architecture --- Remember the Layers

``` text
Clients / Interfaces
   ├── PMT
   ├── IDE Plug-ins
   └── Web Portal
          |
          v
      APIs / MCP
          |
          v
   Intelligence Layer
   ├── Orchestration
   ├── Embeddings
   └── Agentic workflows
          |
     +----+----+
     |    |    |
     v    v    v
    LLM  RAG  LLMOps
         |     |
     Context   Prompt Mgmt
     Catalogue Observability

Security + Analytics are cross-cutting.
```

### 12 subsystems --- one-line memory

  SS     Remember
  ------ --------------------------
  SS1    LLMs
  SS2    Data Pipeline
  SS3    Context Catalogue
  SS4    LLMOps
  SS5    Intelligence
  SS6    APIs / MCP
  SS7    PMT integration
  SS8    IDE plug-ins
  SS9    Web portal
  SS10   IT tools integration
  SS11   Analytics / feedback
  SS12   Security / authorization

------------------------------------------------------------------------

## 4. End-to-End Request

``` text
Engineer
  ↓
PMT / IDE / Web
  ↓
API / MCP
  ↓
Orchestrator
  ├── retrieve context
  ├── get managed prompt
  └── call LLM
  ↓
Grounded response
  ↓
Client
```

Security and observability apply throughout.

**Senior-level point:** the value is not merely calling an LLM; the
platform coordinates **data + retrieval + prompts + models +
orchestration + security + observability**.

------------------------------------------------------------------------

## 5. Your Strongest Contribution --- Prompt Management

### Problem

Hard-coded/scattered prompts cause:

-   Difficult versioning
-   Poor reproducibility
-   Difficult experimentation
-   Difficult rollback
-   Weak governance

### Your solution

``` text
Application
    ↓
MiDAS Prompt API
    ↓
Prompt Management
    ↓
Langfuse
```

Treat prompts as **managed production artifacts**.

Important concepts:

-   Versioning
-   Retrieval
-   Environment management
-   Testing
-   Rollback
-   Governance

### Best interview phrase

> "I treated prompts as production artifacts rather than hard-coded
> strings inside application code."

------------------------------------------------------------------------

## 6. Why Langfuse?

Think:

``` text
LLM Application
      ↓
   Langfuse
   ├── Prompt management
   ├── Tracing
   ├── Usage
   ├── Latency
   └── Observability
```

**Answer:**

> "Langfuse provides specialized LLM observability and prompt-management
> capabilities, so we didn't need to build those capabilities from
> scratch. MiDAS then exposed the required functionality through
> enterprise APIs."

If asked **why not build internally?**

> "Building it ourselves would increase engineering and maintenance
> cost. Using a specialized capability lets the platform focus on
> enterprise integration, governance and developer experience."

------------------------------------------------------------------------

## 7. LLMOps --- Must Know

### MLOps

``` text
Model → Deploy → Monitor
```

### LLMOps

``` text
Prompt + Model + Context
          ↓
       Deploy
          ↓
       Trace
          ↓
Evaluate quality / cost / latency
          ↓
       Improve
```

Remember:

**Prompt → Model → Context → Trace → Evaluate → Cost → Feedback**

------------------------------------------------------------------------

## 8. RAG --- Rapid Revision

``` text
Documents
 ↓
Parse
 ↓
Chunk
 ↓
Embed
 ↓
Index
 ↓
Retrieve
 ↓
Rerank (optional)
 ↓
Context
 ↓
Prompt
 ↓
LLM
 ↓
Grounded response
```

### Why RAG?

-   Enterprise knowledge changes
-   Knowledge remains outside model weights
-   Easier updates
-   Better grounding

### RAG vs Fine-tuning

> **RAG changes/accesses knowledge; fine-tuning adapts model
> behavior/task performance.**

------------------------------------------------------------------------

## 9. Agentic AI

### Deterministic

``` text
A → B → C → D
```

### Agentic

``` text
Request
   ↓
Agent
 ┌─┴──────┐
Tool A  Tool B
 └──┬─────┘
    ↓
 Result
```

Use agents when the next action depends on intermediate results.

**Important senior point:**

> "I would not make every workflow agentic. If a deterministic workflow
> solves the problem, it is simpler and easier to operate."

### LangChain vs LangGraph

-   **LangChain:** compose LLM/tool operations
-   **LangGraph:** stateful workflows, branching, loops, complex agent
    behavior

------------------------------------------------------------------------

## 10. Microservices

### Benefits

-   Separation of concerns
-   Independent scaling
-   Independent deployment
-   Clear ownership
-   Failure isolation

### Cost

-   Network failures
-   Distributed tracing
-   Service discovery
-   Deployment/version complexity

**Answer:**

> "Microservices make sense when capabilities have different ownership,
> scaling or deployment characteristics. The trade-off is
> distributed-system complexity."

------------------------------------------------------------------------

## 11. Production / Scalability

If asked **"How would you scale MiDAS?"**, think by layer.

### API

-   Stateless services
-   Horizontal replicas
-   Load balancing
-   Rate limiting

### AI

-   Concurrency control
-   Queues for long-running work
-   Timeouts
-   Retries for transient failures
-   Model-specific scaling

### Retrieval

-   Index optimization
-   Caching
-   Metadata filtering
-   Horizontal scaling

### Platform

-   Kubernetes
-   HPA
-   Health checks
-   Monitoring

### Reliability keywords

**Timeout → Retry → Circuit breaker → Fallback → Queue → Backpressure →
Autoscaling → Monitoring**

------------------------------------------------------------------------

## 12. Observability

A request may cross:

``` text
API → Retrieval → Prompt → LLM → Response
```

Track:

-   Latency
-   Errors
-   Model
-   Prompt/version
-   Tokens
-   Cost
-   Retrieval behavior

**Answer:**

> "Distributed tracing helps identify where latency and failures occur
> across a multi-service AI workflow."

------------------------------------------------------------------------

## 13. Enterprise AI Security

Think beyond authentication:

``` text
Identity
Authorization
RBAC
Data isolation
Secrets
Encryption
Audit
Prompt injection
Tool authorization
```

For RAG:

> Users should retrieve only documents they are authorized to access.

For agents:

> Tools should have least-privilege access.

------------------------------------------------------------------------

## 14. RSA / Use Cases

Keep this distinction clear:

``` text
MiDAS
  ↓
Reusable platform capabilities
  ↓
RSA / other engineering use cases
```

RSA demonstrates how capabilities such as **embeddings/retrieval** can
be consumed by a real engineering workflow.

------------------------------------------------------------------------

## 15. Most Likely Questions

### MiDAS / Architecture

1.  What is MiDAS?
2.  Why did Bosch need a platform?
3.  Platform vs application?
4.  Explain the architecture.
5.  Walk through a request end-to-end.
6.  Why microservices?
7.  How would you scale it?
8.  How would you make it highly available?

### Prompt Management / LLMOps

9.  Why centralized prompt management?
10. Why Langfuse?
11. How would you version prompts?
12. How would you roll back a bad prompt?
13. How would you test prompt changes?
14. What is LLMOps?
15. What should be monitored in an LLM system?

### GenAI

16. Explain RAG.
17. How do you improve retrieval?
18. Vector vs hybrid search?
19. How do you reduce hallucinations?
20. RAG vs fine-tuning?
21. How do you evaluate RAG?
22. When would you use an agent?

### Your work

23. What exactly did you build?
24. What was your role in prompt management?
25. What was the hardest technical problem?
26. What would you do differently today?
27. How did you ensure reuse across use cases?

------------------------------------------------------------------------

## 16. Short Answers to Memorize

**What is MiDAS?**

> "A reusable enterprise AI platform for Bosch software engineering."

**Your main contribution?**

> "I architected and developed the LLMOps prompt-management capability
> using Langfuse and exposed it through reusable REST APIs."

**Why Langfuse?**

> "It provides specialized prompt management and LLM observability
> without requiring us to build those capabilities from scratch."

**What is LLMOps?**

> "The operational lifecycle around LLM applications --- prompts,
> models, context, tracing, evaluation, cost, latency and feedback."

**What is RAG?**

> "Retrieve relevant enterprise knowledge at runtime and provide it as
> context to the LLM for a grounded response."

**Why microservices?**

> "Independent scaling, deployment and ownership, at the cost of
> distributed-system complexity."

**Why observability?**

> "To understand latency, failures, model behavior, token usage and cost
> across a distributed AI workflow."

------------------------------------------------------------------------

## 17. Best 2-Minute Bosch Answer

> "At Bosch I worked on MiDAS, a microservices-based enterprise GenAI
> platform designed to automate software engineering workflows across
> the V-Model lifecycle. The goal was to provide reusable AI
> capabilities so individual engineering teams would not need to build
> their own LLM, retrieval, orchestration and LLMOps infrastructure.
>
> My main contribution was in the LLMOps area. I architected and
> developed a prompt-management capability using Langfuse and exposed it
> through reusable REST APIs. The objective was to treat prompts as
> managed production artifacts rather than embedding them directly in
> application code.
>
> I also developed GenAI solutions using RAG and prompt engineering on
> top of MiDAS capabilities, including engineering use cases such as
> requirement similarity. This gave me experience not only building
> GenAI applications but also thinking about platform concerns such as
> API design, scalability, observability, security and reuse.
>
> The broader business objective of MiDAS was to automate software
> engineering workflows, with the platform projected to deliver around
> USD 650K in annual productivity and cost savings."

------------------------------------------------------------------------

## 18. Senior / Architect Answer Pattern

For system-design questions:

``` text
1. Requirements
      ↓
2. High-level architecture
      ↓
3. Data / AI flow
      ↓
4. Scalability
      ↓
5. Reliability
      ↓
6. Security
      ↓
7. Observability
      ↓
8. Cost
      ↓
9. Trade-offs
```

Don't jump directly to technologies.

------------------------------------------------------------------------

## 19. Do Not Overclaim

Only state exact implementation details you personally remember.

If unsure:

> "I don't remember the exact configuration, but architecturally the
> purpose was..."

Avoid inventing:

-   Production traffic
-   Latency numbers
-   Evaluation scores
-   Exact vector DB
-   Exact embedding model
-   Exact deployment topology
-   Exact MCP production status
-   Exact Langfuse hosting configuration

------------------------------------------------------------------------

## 20. Final 10-Minute Revision Card

``` text
MiDAS
= Enterprise GenAI Platform
= Reuse + Governance + Scale

Your work
→ LLMOps
→ Langfuse
→ Prompt Management
→ REST APIs
→ RAG / GenAI

Prompt Management
→ Version
→ Retrieve
→ Govern
→ Test
→ Rollback

RAG
→ Parse
→ Chunk
→ Embed
→ Retrieve
→ Rerank
→ Context
→ LLM

LLMOps
→ Prompt
→ Model
→ Trace
→ Evaluate
→ Cost
→ Feedback

Production
→ Stateless
→ Kubernetes
→ Scale
→ Timeout
→ Retry
→ Observability
→ Security

Key distinction
MiDAS = Platform
RSA = Use case
```

### Three sentences to remember

> **"MiDAS is a reusable enterprise AI platform, not a single AI
> application."**

> **"My strongest contribution was architecting and developing prompt
> management in the LLMOps layer using Langfuse and exposing it through
> reusable APIs."**

> **"My Bosch experience moved me from building individual GenAI
> solutions toward building reusable AI platform capabilities, which is
> the direction I want to continue at a senior/architect level."**
