# Thomson Reuters — Interview Preparation Guide
## Lead Research Engineer | GenAI • RAG • NLP • ML • AWS • Production AI

> **Interview positioning:** Present TR as the point in your career where you moved from traditional ML/NLP into enterprise GenAI and learned to take research/PoCs into production. Emphasize **enterprise RAG, agentic QA, long-document AI, production inference, AWS, architecture, and mentoring**.

---

# 1. Your TR Story — 30 Seconds

“I worked as a Lead Research Engineer at Thomson Reuters Labs, focusing on AI solutions using Generative AI, Machine Learning, and NLP for the Audit and Legal domains.

One of my major projects was an enterprise conversational AI product for audit professionals. We used LLMs, RAG, vector databases, prompt engineering, and a ReAct-based agent to retrieve information from multiple enterprise knowledge sources and generate grounded answers.

I also worked on long-document summarization and production AI services using FastAPI and AWS. My responsibilities covered the lifecycle from problem definition and PoCs through data preparation, model development, inference services, deployment, architecture discussions, code reviews, and mentoring.”

---

# 2. TR Experience — The 5 Stories to Remember

### Story 1 — Enterprise Conversational AI
**Core:** LLM + RAG + Vector DB + ReAct + multiple knowledge sources

### Story 2 — Long-Document Summarization
**Core:** financial/regulatory documents + Docling + LLM/VLM + production integration

### Story 3 — Legal Tracker Invoice Auditing
**Core:** Block Billing + Duplicate Billing + ML + batch/real-time AWS inference

### Story 4 — Regulatory Intelligence
**Core:** BART fine-tuning + labeled data + SageMaker Inference Pipelines

### Story 5 — Audit QA Fine-Tuning PoC
**Core:** open-source LLM + Unsloth + LoRA/QLoRA + PEFT

---

# 3. Enterprise Conversational AI — Your Most Important TR Project

## Business Problem

Audit professionals need accurate answers from multiple authoritative enterprise knowledge sources without manually searching through documents and repositories.

The goal was to provide a natural-language conversational interface while keeping answers grounded in source material.

## Knowledge Sources

Your architecture handled information from:

- Thomson Reuters content
- SEC documents
- Regulatory documents
- Internal repositories
- User-uploaded documents

This is important because the system was not simply “chat with an LLM.” It was an **enterprise knowledge-grounded AI system**.

---

# 4. Conversational AI Architecture

A good interview-level representation:

```text
                         ┌──────────────────────┐
                         │   Enterprise Users   │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Conversational API   │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │   ReAct Agent        │
                         │ Reason + Act         │
                         └──────────┬───────────┘
                                    │
                    ┌───────────────┼────────────────┐
                    │               │                │
                    ▼               ▼                ▼
              TR Content       SEC/Regulatory   Internal/User
                    │               │                │
                    └───────────────┼────────────────┘
                                    ▼
                         ┌──────────────────────┐
                         │ Retrieval Layer      │
                         │ Vector DB / Search   │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Context Construction │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │        LLM           │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Grounded Answer      │
                         └──────────────────────┘
```

### Important distinction

Do not describe the ReAct agent as simply “the LLM answering the question.”

Explain:

> “The ReAct-based agent was used to reason about the task and determine the appropriate retrieval/action strategy. The retrieved information was then supplied as context to the LLM for grounded answer generation.”

---

# 5. Why RAG?

### Strong interview answer

“RAG was appropriate because the required knowledge existed in enterprise sources and could change over time. We wanted the model to retrieve relevant source information at query time rather than relying entirely on its parametric knowledge. This improves grounding, freshness, and the ability to trace an answer back to enterprise content.”

### RAG vs Fine-Tuning

| Requirement | Better fit |
|---|---|
| Frequently changing enterprise knowledge | RAG |
| Ground answers in source documents | RAG |
| Need source attribution | RAG |
| Change model behavior/style | Fine-tuning |
| Task-specific response format | Fine-tuning |
| Domain-specific behavior | Fine-tuning |
| Knowledge + behavior adaptation | RAG + Fine-tuning |

**Key sentence:**

> “I view RAG primarily as a knowledge-access mechanism, while fine-tuning is primarily a model-behavior/task-adaptation mechanism.”

---

# 6. Why Vector Databases?

Traditional keyword search can miss semantically similar content.

Vector retrieval:
1. Convert documents into embeddings.
2. Store embeddings in a vector index.
3. Convert the query into an embedding.
4. Retrieve semantically similar chunks.
5. Provide relevant chunks to the LLM.

```text
Document
   ↓
Chunk
   ↓
Embedding
   ↓
Vector DB

User Query
   ↓
Embedding
   ↓
Similarity Search
   ↓
Relevant Context
```

### Principal-level improvement

For an enterprise system, consider:

**Hybrid retrieval**

```text
Keyword Search ──┐
                 ├──> Candidate Results → Reranker → LLM
Vector Search ───┘
```

This combines exact-term matching with semantic matching.

---

# 7. ReAct Agent — Must Know

ReAct = **Reason + Act**

Conceptually:

```text
User Question
      ↓
Agent
      ↓
Reason about next action
      ↓
Select retrieval/tool
      ↓
Execute action
      ↓
Observe result
      ↓
Reason again if necessary
      ↓
Final grounded response
```

### Why use an agent?

A fixed RAG pipeline assumes one retrieval path.

An agent can make decisions such as:
- which knowledge source to use
- which retrieval strategy to apply
- whether additional information is required
- whether another action/tool is needed

### Trade-off

Agentic architecture introduces:
- additional latency
- additional LLM calls
- higher cost
- nondeterminism
- more difficult debugging
- potential loops

### Principal-level statement

> “I would use an agent only where dynamic decision-making adds value. If the retrieval path is deterministic, a conventional RAG pipeline is simpler, cheaper, and easier to operate.”

---

# 8. Long-Document Summarization

You worked on summarization for financial and regulatory documents and integrated these capabilities into production applications.

A useful architecture:

```text
Large Document
      ↓
Document Parsing
      ↓
Structure / Layout Extraction
      ↓
Chunking / Sectioning
      ↓
LLM / VLM Processing
      ↓
Intermediate Summaries
      ↓
Final Summary
```

## Why long-document summarization is difficult

- Context-window limitations
- Large token consumption
- Important information distributed across the document
- Tables and layouts
- Need for factual consistency
- Potential loss of context between sections

### Hierarchical summarization

For very large documents:

```text
Document
  ↓
Sections
  ↓
Section Summaries
  ↓
Combined Summary
  ↓
Final Summary
```

This is often preferable to attempting to send an entire huge document to a model in one call.

---

# 9. Docling + LLM/VLM

For the Cloud Audit Suite summarization service:

- Docling → document understanding/parsing
- LLM → language-based summarization
- VLM → useful when visual/layout information matters
- FastAPI → inference API
- Docker → packaging
- GitHub Actions → CI/CD
- ECR → container registry
- ECS/Fargate → container deployment

### Why VLM?

Text extraction alone may lose information represented through:
- tables
- visual structure
- complex layouts
- scanned/visual content

A VLM can reason over visual document information where appropriate.

---

# 10. Why Async FastAPI?

Long-document inference can take significantly longer than a normal API request.

Instead of:

```text
POST /summarize
       ↓
Wait
       ↓
Long inference
       ↓
Response
```

Prefer:

```text
POST /summarize
       ↓
Create Job
       ↓
Return Job ID
       ↓
Background Processing
       ↓
Store Result
       ↓
GET /result/{job_id}
```

### Benefits

- Non-blocking request handling
- Better scalability
- Better user experience
- Easier retry handling
- Separation of API and heavy processing

---

# 11. Productionizing the Summarization Service

Your production story:

```text
Developer
   ↓
GitHub
   ↓
GitHub Actions
   ↓
Docker Build
   ↓
Amazon ECR
   ↓
Amazon ECS / Fargate
   ↓
FastAPI Inference Service
```

### Be ready to explain each component

**FastAPI**
- Lightweight Python API framework
- Async support
- Good fit for inference services

**Docker**
- Reproducible packaging
- Dependency isolation
- Consistent deployment

**ECR**
- Private container image registry

**ECS/Fargate**
- Managed container execution
- No need to manage EC2 servers directly
- Horizontal scaling possible

**GitHub Actions**
- Automated build/test/deployment pipeline

---

# 12. Scaling the Long-Document Service

If interviewer asks:

> “What happens if traffic increases 10x?”

Answer:

1. Keep API containers stateless.
2. Put long-running jobs behind a queue.
3. Autoscale workers based on workload/queue depth.
4. Store large documents/results in object storage.
5. Separate document processing from model inference when useful.
6. Implement retries and dead-letter handling.
7. Add idempotency to avoid duplicate processing.
8. Monitor latency, throughput, errors and model cost.

### Principal-level architecture

```text
                 ┌───────────────┐
Users ──────────►│ API / Gateway │
                 └───────┬───────┘
                         │
                         ▼
                    Job Queue
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
       Document Workers       AI Inference
              │                     │
              └──────────┬──────────┘
                         ▼
                    Result Store
```

---

# 13. Legal Tracker — Invoice Auditing

You developed and productionized AI-powered invoice auditing capabilities for:

- Block Billing
- Duplicate Billing

The key interview message:

> “This was a production ML problem where the objective was to identify potentially problematic billing patterns and integrate model inference into the product workflow.”

## Batch vs Real-Time

### Batch
Useful when:
- large historical volumes need processing
- immediate response is not required
- throughput is more important than latency

### Real-time
Useful when:
- the user needs immediate feedback
- inference is part of the workflow
- low latency matters

---

# 14. Evaluating Invoice-Auditing Models

Do not say only:

> “We measured accuracy.”

Billing/audit problems can be imbalanced.

Discuss:

- Precision
- Recall
- F1
- PR-AUC
- False positives
- False negatives
- Manual-review workload
- Business impact

### Principal-level point

The optimal threshold is a **business decision**, not necessarily 0.5.

For example:
- High false-negative cost → favor recall.
- High manual-review cost → improve precision.

---

# 15. Regulatory Intelligence — BART

### Your story

You developed a transformer-based document summarization solution by fine-tuning Hugging Face BART models on labeled data and productionizing inference using SageMaker Inference Pipelines.

### Architecture

```text
Labeled Dataset
      ↓
Preprocessing
      ↓
Tokenization
      ↓
BART Fine-Tuning
      ↓
Evaluation
      ↓
SageMaker Inference Pipeline
      ↓
Production
```

### Why BART?

BART is an encoder-decoder transformer architecture suited to sequence-to-sequence tasks such as summarization.

Fine-tuning adapts the model to the domain and task.

---

# 16. Audit QA — Open-Source LLM Fine-Tuning

You built a domain-specific Audit QA PoC using:

- Open-source LLMs
- Unsloth
- LoRA
- QLoRA
- PEFT

## LoRA

Instead of updating the entire model:

```text
Base Model
   ↓
Freeze Parameters
   ↓
Train Small Adapter Matrices
```

Benefits:
- fewer trainable parameters
- lower memory requirement
- cheaper/faster fine-tuning
- adapter-based model customization

## QLoRA

Conceptually:

```text
Quantized Base Model
        +
      LoRA
        ↓
Parameter-Efficient Fine-Tuning
```

Main benefit: lower memory requirements while adapting larger models.

## PEFT

Parameter-Efficient Fine-Tuning is the broader family/approach; LoRA is one important technique within it.

## Unsloth

Tooling used to make open-source LLM fine-tuning more efficient.

---

# 17. Fine-Tuning vs RAG — Very Likely Question

### Answer

“I would first determine whether the problem is knowledge access or behavior adaptation.

If the knowledge changes frequently and answers must be grounded in authoritative enterprise documents, I would generally choose RAG.

If the goal is to change the model's behavior, response format, domain style, or task-specific capability, fine-tuning can be appropriate.

For an enterprise QA system, I would often consider combining them: RAG supplies current knowledge, while fine-tuning can adapt the model's behavior.”

---

# 18. GenAI Evaluation

For enterprise GenAI, don't evaluate only the final answer.

## Retrieval

- Recall@K
- Precision@K
- MRR / NDCG where appropriate

## Generation

- Correctness
- Relevance
- Faithfulness / groundedness
- Completeness
- Citation/source accuracy

## Safety

- Hallucination
- Prompt injection
- Sensitive-data leakage
- Unsafe output

## Production

- Latency
- Throughput
- Availability
- Token consumption
- Cost

### Strong answer

> “I separate retrieval evaluation from generation evaluation. Otherwise, it becomes difficult to determine whether a bad answer came from retrieving the wrong context or from the LLM failing to use the correct context.”

---

# 19. Hallucination — How Would You Reduce It?

Use multiple layers:

1. Improve retrieval quality.
2. Use authoritative sources.
3. Rerank retrieved content.
4. Explicitly instruct the model to stay grounded.
5. Require citations where appropriate.
6. Evaluate factual consistency.
7. Refuse when evidence is insufficient.
8. Monitor production failures.

### Important

Prompt engineering alone is not a complete hallucination strategy.

---

# 20. Enterprise AI Security

For your TR architecture, be ready to discuss:

### Authentication / Authorization
Users should only retrieve documents they are authorized to access.

### Data isolation
Different users/tenants must not leak information across boundaries.

### Prompt injection
Retrieved documents can contain malicious instructions.

Treat retrieved content as **untrusted data**, not trusted instructions.

### PII / Sensitive data
- minimize exposure
- encryption
- access control
- retention policies
- audit logging

### Tool/Agent security
For agentic systems:
- authorize tools
- restrict tool scope
- validate arguments
- use least privilege
- require human approval for high-risk actions

---

# 21. AWS — Your TR Interview Cheat Sheet

### ECS
Container orchestration/service management.

### Fargate
Serverless compute for containers.

### ECR
Container image registry.

### Lambda
Event-driven/serverless execution; good for short-lived functions rather than heavy long-running model inference.

### SageMaker
Managed ML platform for training/deployment/inference.

### SageMaker Inference Pipelines
Compose multiple inference/pre-processing steps into a managed inference flow.

### API layer
FastAPI can expose model/application APIs.

---

# 22. Production AI Lifecycle

This is an important Lead/Principal-level narrative:

```text
Business Problem
      ↓
Data
      ↓
PoC
      ↓
Model / LLM
      ↓
Evaluation
      ↓
Inference API
      ↓
Containerization
      ↓
CI/CD
      ↓
AWS Deployment
      ↓
Monitoring
      ↓
Feedback / Improvement
```

Your strongest TR positioning is that you participated across this lifecycle rather than only training models.

---

# 23. Architecture Trade-Offs You Should Be Ready For

### Agent vs deterministic workflow
Use agents when dynamic decisions are valuable.

### RAG vs fine-tuning
Knowledge access vs behavior adaptation.

### Batch vs real-time
Throughput/cost vs latency.

### Synchronous vs asynchronous
Simple/low-latency vs long-running workloads.

### VLM vs text-only model
Visual/layout understanding vs lower cost/simpler processing.

### Larger vs smaller model
Quality vs latency/cost.

### Managed vs self-hosted model
Operational simplicity vs control/cost/customization.

### Vector vs hybrid retrieval
Semantic matching vs exact-term + semantic coverage.

---

# 24. Likely TR Deep-Dive Questions

## RAG / Conversational AI

1. Explain your RAG architecture.
2. Why did you use a ReAct agent?
3. Why not a simple RAG pipeline?
4. What was stored in the vector database?
5. How were documents chunked?
6. How did you select embeddings?
7. How would you improve retrieval?
8. How would you handle document updates?
9. How would you prevent hallucinations?
10. How would you evaluate RAG?
11. How would you add citations?
12. How would you handle access control?
13. How would you defend against prompt injection?
14. How would you scale the system?
15. What happens if the LLM is unavailable?

## Summarization

1. Why is long-document summarization difficult?
2. How do you handle context limits?
3. Why use hierarchical summarization?
4. Why Docling?
5. Why VLM?
6. How do you evaluate summary quality?
7. How do you detect factual inconsistencies?
8. Why asynchronous inference?
9. How would you process a 10x larger document?
10. How would you reduce inference cost?

## Fine-Tuning

1. Explain LoRA.
2. Explain QLoRA.
3. Explain PEFT.
4. Why fine-tune instead of RAG?
5. How do you choose a base model?
6. How do you prevent overfitting?
7. How do you evaluate a fine-tuned LLM?
8. When would you use full fine-tuning?
9. How do you deploy the fine-tuned model?
10. What happens when domain knowledge changes?

## Production / AWS

1. Why FastAPI?
2. Why Docker?
3. Why ECS/Fargate?
4. How would you scale?
5. How would you monitor?
6. How would you handle retries?
7. How would you implement health checks?
8. How would you roll back a bad model?
9. How would you manage model versions?
10. How would you reduce AWS cost?

---

# 25. Principal-Level Architecture Answer Framework

When asked to design an AI system:

### Step 1 — Clarify
- users
- workload
- scale
- latency
- availability
- data sensitivity

### Step 2 — High-Level Architecture
Describe:
- API
- services
- storage
- model layer
- retrieval
- asynchronous components

### Step 3 — AI Concerns
Discuss:
- RAG
- model selection
- grounding
- evaluation
- hallucination

### Step 4 — Production
Discuss:
- scaling
- resilience
- observability
- CI/CD
- security

### Step 5 — Trade-offs
Explicitly discuss:
- quality vs cost
- latency vs quality
- agent vs deterministic workflow
- RAG vs fine-tuning
- managed vs self-hosted

---

# 26. Your 2-Minute TR Story

“I worked as a Lead Research Engineer at Thomson Reuters Labs, where I focused on AI solutions using Generative AI, Machine Learning, and NLP for the Audit and Legal domains.

One of my major projects was an enterprise conversational AI product for audit professionals. The objective was to allow users to ask questions in natural language and get accurate answers from multiple enterprise knowledge sources. The solution used LLMs, RAG, vector databases, prompt engineering, and a ReAct-based agent to determine the appropriate retrieval strategy and provide relevant context to the LLM. The knowledge sources included Thomson Reuters content, SEC and regulatory documents, internal repositories, and user-uploaded documents.

I also worked on long-document summarization for financial and regulatory documents. For the Cloud Audit Suite, I developed a production summarization service using Docling, LLMs and VLMs, with asynchronous FastAPI inference services and AWS-based deployment using Docker, GitHub Actions, ECR and ECS/Fargate.

In Legal Tracker, I worked on AI-powered invoice auditing capabilities for Block Billing and Duplicate Billing detection, supporting batch and real-time inference on AWS.

For Regulatory Intelligence, I developed a transformer-based summarization solution by fine-tuning Hugging Face BART models and productionizing inference using SageMaker Inference Pipelines. I also built a domain-specific Audit QA proof of concept by fine-tuning open-source LLMs using Unsloth, LoRA/QLoRA and PEFT.

Overall, my role at Thomson Reuters was about taking AI solutions from research and experimentation toward production, while also contributing to architecture discussions, code reviews, and mentoring junior engineers.”

---

# 27. Autodesk Positioning — Why TR Matters

Your TR experience maps strongly to the Autodesk Principal Agentic AI Platform role.

### Autodesk asks for

**Agentic AI**
→ You have ReAct-based agent experience.

**RAG**
→ You built enterprise document-grounded QA.

**Memory/context**
→ You have experience with enterprise context construction and retrieval.

**Tool/orchestration thinking**
→ ReAct agent and retrieval strategy.

**LLM integration**
→ Multiple LLM-based solutions.

**Evaluation**
→ Need to discuss retrieval + generation + safety evaluation.

**Production AI**
→ FastAPI + Docker + AWS.

**Cloud architecture**
→ ECS/Fargate, Lambda, SageMaker.

**Platform thinking**
→ TR conversational AI + production services.

**Leadership**
→ Architecture discussions + code reviews + mentoring.

---

# 28. TR + Bosch = Your Strongest Principal Narrative

### Thomson Reuters

**AI Solution Engineering**

```text
NLP / ML
   ↓
Transformers
   ↓
LLMs
   ↓
RAG
   ↓
Agentic QA
   ↓
Production AI
```

### Bosch

**AI Platform Engineering**

```text
GenAI Use Cases
   ↓
Reusable Platform Capabilities
   ↓
LLMOps
   ↓
Prompt Management
   ↓
Enterprise Platform
```

### Target Role

**Principal Agentic AI Platform**

```text
AI Solutions
     +
AI Platform Engineering
     +
Agentic Architecture
     ↓
Principal / Architect
```

This is the career narrative to emphasize.

---

# 29. What NOT to Overclaim

Be exact about details you personally remember.

Do not invent:
- exact vector DB name if not certain
- exact embedding model
- exact chunk size
- exact retrieval algorithm
- exact model names
- exact latency
- exact production traffic
- exact evaluation scores
- exact AWS components you did not personally use

A strong way to answer:

> “The implementation I worked on used X. If I were redesigning it today at larger scale, I would consider Y because…”

This demonstrates both experience and architectural maturity without overclaiming.

---

# 30. Rapid Revision Card

## TR Core

**Enterprise GenAI → RAG → ReAct → Summarization → Fine-Tuning → AWS Production**

### Conversational AI
**LLM + RAG + Vector DB + ReAct + Enterprise Sources**

### RAG
**Parse → Chunk → Embed → Retrieve → Rerank → Context → LLM**

### ReAct
**Reason → Act → Observe → Reason → Answer**

### Long Document
**Parse → Structure → Chunk → Summarize → Aggregate**

### LoRA
**Freeze Base + Train Adapters**

### QLoRA
**Quantized Base + LoRA**

### Production
**FastAPI + Docker + CI/CD + ECR + ECS/Fargate**

### ML Deployment
**SageMaker + Inference Pipeline**

### Evaluation
**Retrieval + Groundedness + Correctness + Safety + Latency + Cost**

### Security
**IAM + Access Control + Tenant Isolation + Encryption + Prompt Injection Defense + Audit**

---

# 31. Five Things to Memorize Before the Interview

### 1.
> “I worked on taking AI solutions from research and experimentation toward production.”

### 2.
> “RAG provided access to current enterprise knowledge, while fine-tuning was useful for adapting model behavior and task performance.”

### 3.
> “I would use an agent where dynamic decision-making adds value; otherwise, I prefer a simpler deterministic workflow.”

### 4.
> “For GenAI evaluation, I separate retrieval quality from generation quality.”

### 5.
> “For enterprise AI, security is not just model security — it includes data access, tenant isolation, prompt injection, tool authorization, auditability and responsible AI.”

---

## Final Interview Focus

For a Principal/Architect interview, spend the most preparation time on:

**1. Enterprise RAG architecture**
**2. ReAct / agentic architecture**
**3. Long-document processing**
**4. GenAI evaluation**
**5. AWS production architecture**
**6. AI security**
**7. RAG vs fine-tuning**
**8. Scalability and resilience**
**9. Your TR architecture decisions**
**10. How TR + Bosch prepared you for Principal Agentic AI Platform work**
