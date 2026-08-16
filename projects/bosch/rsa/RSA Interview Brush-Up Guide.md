# RSA (Requirement Similarity Assist)
## High-Level Understanding & Interview Brush-Up Guide

> **Purpose:** Understand the RSA use case from business problem → requirement data → indexing → similarity search → result generation → architecture → engineering considerations.
>
> **Audience:** Senior / Lead AI Engineer / AI Architect interview preparation.
>
> **Scope:** High-level understanding first. Detailed APIs and source-code implementation should be added later as the RSA knowledge base is built.

---

# 1. RSA at a Glance

## What is RSA?

**RSA — Requirement Similarity Assist — is an AI-assisted requirements analysis application in the MiDAS platform that finds similar or related requirements across Bosch engineering projects.**

The goal is to reduce the manual effort involved in comparing requirements and identifying potentially reusable, duplicate, or related requirements.

At a high level:

```text
New Requirement
       │
       ▼
   RSA Search
       │
       ▼
Find semantically similar
requirements
       │
       ▼
Rank / score results
       │
       ▼
Engineer reviews results
```

The important idea is:

> **RSA is primarily a semantic search / similarity system, not a generative chatbot.**

MiDAS documentation identifies RSA as a requirements-analysis use case using **vector similarity search**, with Spaces, embeddings, LLM inference and analytics contributing to the overall solution. 
---

# 2. Business Problem

Automotive software development involves a very large number of requirements.

Requirements can exist across:

- Different projects
- Different vehicle programs
- Different V-Model stages
- Different engineering teams
- Different versions of systems

A new requirement may therefore be very similar to something that already exists.

For example:

```text
Requirement A

"The braking system shall activate within 50 ms
after emergency detection."
```

and

```text
Requirement B

"The emergency braking response time shall not
exceed 50 milliseconds."
```

The wording is different, but the underlying meaning is very similar.

A simple keyword search may not recognize this effectively.

RSA addresses this by using **semantic similarity**.

---

# 3. What Exactly Is a Requirement?

A requirement is a statement describing something that a system must do or a constraint it must satisfy.

Example:

```text
REQ-001

The braking system shall activate within 50 ms
after emergency detection.
```

Another example:

```text
REQ-002

The system shall detect pedestrians within
80 meters.
```

A requirement normally has more than just text.

Conceptually:

```text
Requirement
│
├── Requirement ID
├── Requirement Text
├── Project
├── Module / Component
├── V-Model context
├── Metadata
└── Status / other attributes
```

For RSA, the **requirement text is particularly important** because semantic similarity is calculated from its representation.

---

# 4. What Does RSA Actually Return?

Suppose the engineer provides:

```text
"The braking system shall activate within 50 ms."
```

RSA searches existing requirements and may return:

```text
REQ-1042
"The emergency braking response shall not
exceed 50 milliseconds."

Similarity: High
```

and:

```text
REQ-2381
"The vehicle shall initiate automatic braking
after emergency detection."

Similarity: Medium
```

The engineer can then decide whether the existing requirement is:

- Similar
- Related
- Reusable
- A potential duplicate
- Not actually relevant

### Important

RSA assists the engineer.

It does **not replace engineering judgement**.

---

# 5. RSA vs Traditional Keyword Search

This is one of the easiest ways to explain RSA in an interview.

### Keyword search

```text
Query
  │
  ▼
Match exact words
  │
  ▼
Results
```

### RSA

```text
Requirement
     │
     ▼
Embedding
     │
     ▼
Semantic representation
     │
     ▼
Vector similarity search
     │
     ▼
Semantically similar requirements
```

Consider:

```text
"Brake response must be below 50 milliseconds."

vs.

"Emergency braking shall respond within 0.05 seconds."
```

They use different words but express essentially the same concept.

Semantic embeddings allow RSA to capture this relationship.

---

# 6. The Core Mental Model

RSA can be understood as a specialized retrieval system.

```text
             RSA

   ┌─────────────────────┐
   │ Requirement Data    │
   └──────────┬──────────┘
              │
              ▼
       Embedding Model
              │
              ▼
       Vector Representation
              │
              ▼
        Vector Search
              │
              ▼
     Similar Requirements
              │
              ▼
       Ranking / Analysis
              │
              ▼
       Engineer Decision
```

This is closely related to the retrieval concepts you already studied for RAG, but there is an important difference:

> **RAG retrieves information to provide context to an LLM for answer generation. RSA primarily retrieves requirements to support similarity analysis.**

---

# 7. RSA and MiDAS

RSA is not a standalone AI platform.

It is a **use case built using MiDAS platform capabilities**.

Think of:

```text
                 MiDAS
                   │
        ┌──────────┼──────────┐
        │          │          │
   Embeddings   Vector     LLM
                Search   Inference
        │          │          │
        └──────────┼──────────┘
                   │
                   ▼
                  RSA
```

MiDAS is intentionally designed as a collection of reusable capabilities.

The same building blocks can support other engineering use cases.

The MiDAS documentation describes this architecture as reusable platform capabilities where Spaces, vector search and LLM inference are repeatedly combined for different V-Model use cases.

---

# 8. RSA Architecture — High Level

A useful mental model is:

```text
                         RSA User
                            │
                            ▼
                     RSA Application
                            │
                            ▼
                  ┌──────────────────┐
                  │ Requirement      │
                  │ Similarity Flow  │
                  └────────┬─────────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
        Space Mgmt     Embeddings    Vector Search
             │             │             │
             └─────────────┼─────────────┘
                           │
                           ▼
                  Similar Requirements
                           │
                           ▼
                    Ranking / Analysis
                           │
                           ▼
                     RSA Response
```

The exact implementation details behind these services should be treated as the next layer of the RSA knowledge base.

---

# 9. RSA Has Two Logical Phases

This is one of the most important concepts to understand.

RSA has a **preparation/indexing phase** and a **runtime query phase**.

```text
              RSA
               │
       ┌───────┴────────┐
       │                │
       ▼                ▼
   INDEXING           QUERY
   PHASE              PHASE
```

---

# 10. Phase 1 — Requirement Indexing

Before RSA can efficiently search requirements, existing requirements need to be prepared for semantic search.

Conceptually:

```text
Existing Requirements
        │
        ▼
   Parse / Extract
        │
        ▼
Requirement Records
        │
        ▼
Generate Embeddings
        │
        ▼
Vector Store
```

For example:

```text
REQ-001
"Brake system shall activate within 50 ms."
                  │
                  ▼
            Embedding Model
                  │
                  ▼
       [0.12, -0.37, 0.81, ...]
```

The resulting vector is stored along with the requirement information.

---

# 11. Why Embeddings?

An embedding converts text into a numerical representation that captures semantic characteristics.

Conceptually:

```text
Requirement Text
       │
       ▼
Embedding Model
       │
       ▼
Vector
```

For example:

```text
"The brake system shall activate within 50 ms."

              ↓

[0.12, -0.31, 0.78, ...]
```

A semantically similar requirement should produce a vector that is close to the first vector under the configured similarity metric.

The MiDAS platform documentation explicitly identifies embeddings as a core capability used by RSA.

---

# 12. Why Not Just Store the Requirement Text?

Because traditional database search is good at:

```text
WHERE text LIKE '%brake%'
```

but not naturally good at:

```text
Find requirements that mean
approximately the same thing.
```

Vector search solves this by comparing numerical representations of meaning.

---

# 13. Phase 2 — Runtime Similarity Search

When an engineer provides a requirement:

```text
"The braking system shall respond within 50 ms."
```

RSA performs approximately this flow:

```text
User Requirement
       │
       ▼
Generate Query Embedding
       │
       ▼
Vector Similarity Search
       │
       ▼
Retrieve Candidate Requirements
       │
       ▼
Rank / Score
       │
       ▼
Return Similar Requirements
```

This is conceptually the same retrieval pattern you learned in RAG:

```text
Query
 ↓
Query Embedding
 ↓
Similarity Search
 ↓
Top-K Candidates
```

But RSA's final objective is **requirements similarity analysis**, rather than generating a grounded answer.

---

# 14. Vector Search

Suppose we have:

```text
REQ-001 → Vector A
REQ-002 → Vector B
REQ-003 → Vector C
REQ-004 → Vector D
```

The query produces:

```text
Query → Vector Q
```

The vector search system compares:

```text
Q ↔ A
Q ↔ B
Q ↔ C
Q ↔ D
```

and returns the closest candidates.

Conceptually:

```text
                Query
                  │
                  ▼
              Vector Q
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Vector A   Vector B   Vector C
       │          │          │
     0.92       0.81       0.43
       │          │          │
       └──────────┼──────────┘
                  ▼
            Ranked Results
```

Higher similarity generally means greater semantic similarity, although the exact interpretation depends on the metric and implementation.

---

# 15. Top-K Retrieval

RSA doesn't normally need to compare the query against every result presented to the user.

Instead, retrieval can identify the best candidates.

For example:

```text
K = 5
```

means:

```text
Return the 5 strongest candidates.
```

Conceptually:

```text
1. REQ-1042   0.96
2. REQ-2841   0.93
3. REQ-1932   0.89
4. REQ-8321   0.86
5. REQ-4421   0.83
```

The exact RSA values for Top-K and similarity thresholds are **not established by the currently available documentation**, so they should be verified against your actual RSA implementation before being used in an interview.

---

# 16. Where Does the LLM Fit?

This is an important distinction.

RSA uses **LLM inference as one of the MiDAS capabilities**, but the core similarity mechanism is vector-based retrieval.

Therefore:

```text
Embedding Model
       │
       └── Finds semantic similarity

LLM
       │
       └── Can perform higher-level reasoning / analysis
```

Do not describe RSA as:

> "An LLM compares all requirements."

That would be an oversimplification.

A better explanation is:

> "RSA uses embeddings and vector similarity for efficient candidate retrieval, while LLM capabilities can be used for higher-level analysis or interpretation in the use-case workflow."

The exact role of the LLM in the current RSA implementation should be verified from the detailed developer guide/source code.

---

# 17. Why an Embedding Model Is Critical

The quality of RSA depends heavily on the embedding representation.

If two requirements have similar meaning:

```text
A:
"Brake response shall be below 50 ms."

B:
"Emergency braking shall respond within
0.05 seconds."
```

we want:

```text
Similarity(A, B) → High
```

If the embedding model fails to capture the semantic relationship:

```text
Similarity(A, B) → Low
```

RSA may miss an important candidate.

Therefore:

> **Embedding quality directly affects retrieval quality.**

This is similar to RAG: poor embeddings → poor retrieval → poor downstream results.

---

# 18. Why a Lightweight Embedding Model Can Make Sense

Your earlier RSA discussion identified `all-MiniLM-L6-v2` as the default embedding model.

From an architectural perspective, a lightweight model is attractive because RSA may need to embed a very large number of requirements during indexing.

Important considerations are:

- Inference speed
- Memory footprint
- Embedding dimensionality
- Semantic quality
- Deployment simplicity
- Cost
- Data privacy

The key architectural trade-off is:

```text
Embedding Quality
       ▲
       │
       │       Large models
       │          /
       │         /
       │   MiniLM
       │      /
       └──────────────────►
             Cost / Speed
```

The ideal choice is not necessarily the largest model.

It is the model that provides **sufficient retrieval quality at acceptable cost and latency**.

---

# 19. Requirement Data vs Document RAG

This distinction is important for interviews.

A normal RAG system often looks like:

```text
PDF
 │
 ▼
Text
 │
 ▼
Chunks
 │
 ▼
Embeddings
 │
 ▼
Vector DB
```

RSA deals with **structured requirement data**.

Conceptually:

```text
Requirement Dataset
 │
 ├── Requirement ID
 ├── Requirement Text
 ├── Project
 ├── Metadata
 └── Other attributes
          │
          ▼
      Embedding
          │
          ▼
      Vector Search
```

So don't automatically explain RSA as "PDF → chunks → RAG."

It is better understood as a **semantic retrieval system over engineering requirements**, implemented using MiDAS's knowledge/search capabilities.

---

# 20. RSA vs RAG

| Aspect | RSA | Typical RAG |
|---|---|---|
| Primary goal | Find similar requirements | Answer questions |
| Main input | Requirement | User question |
| Knowledge | Requirement datasets | Documents / knowledge base |
| Core retrieval | Vector similarity | Vector / hybrid retrieval |
| Output | Similar requirements | Retrieved context + answer |
| LLM | Supporting analysis where applicable | Usually answer generation |
| Main user | Requirements / system engineer | Knowledge worker / developer |
| Key quality metric | Similarity/retrieval quality | Retrieval + answer quality |

### Simple memory trick

> **RAG = Find information to answer a question.**

> **RSA = Find requirements similar to this requirement.**

---

# 21. RSA and the V-Model

RSA belongs to the requirements side of the automotive V-Model.

MiDAS documentation describes RSA as a requirements-analysis use case and places it around the **Requirements** stage, where capabilities such as content extraction, Space management, embeddings, vector search, LLM inference and analytics are combined.

A simplified view:

```text
        Requirements
             │
             │
            RSA
             │
             ▼
      System / Architecture
             │
             ▼
          Design
             │
             ▼
           Code
             │
             ▼
           Tests
```

RSA therefore supports engineers relatively early in the development lifecycle.

---

# 22. End-to-End Mental Model

This is the diagram I would memorize.

```text
                OFFLINE / PREPARATION

       Existing Requirement Data
                  │
                  ▼
          Extract / Prepare
                  │
                  ▼
           Requirement Records
                  │
                  ▼
           Embedding Model
                  │
                  ▼
         Requirement Embeddings
                  │
                  ▼
            Vector Search
               Index
                  │
                  │
══════════════════╪════════════════════════════
                  │
                  │
                ONLINE
                  │
                  ▼
           New Requirement
                  │
                  ▼
           Query Embedding
                  │
                  ▼
         Similarity Search
                  │
                  ▼
           Top Candidates
                  │
                  ▼
          Ranking / Analysis
                  │
                  ▼
        Similar Requirements
                  │
                  ▼
           Engineer Review
```

This is the **RSA mental model**.

---

# 23. What Happens If the Reference Dataset Changes?

Suppose a new project adds:

```text
10,000 new requirements
```

Those requirements need to become searchable.

Conceptually:

```text
New Requirements
      │
      ▼
Embedding Generation
      │
      ▼
Vector Index Update
      │
      ▼
Available for Search
```

This introduces an important engineering consideration:

> **Index freshness.**

The system needs some mechanism to ensure that newly added requirements eventually become searchable.

The exact RSA batch/update mechanism should be covered separately once we inspect the detailed RSA developer documentation.

---

# 24. Scalability Considerations

Suppose Bosch has:

```text
10 million requirements
```

A naive approach would be:

```text
Query
  ↓
Compare against 10M vectors
  ↓
Sort everything
```

This does not scale efficiently.

A production vector-search system uses an appropriate vector index / retrieval infrastructure to efficiently find nearest candidates.

Therefore:

```text
Large requirement corpus
        │
        ▼
Vector indexing
        │
        ▼
Efficient nearest-neighbor search
        │
        ▼
Top-K candidates
```

This is one of the reasons MiDAS provides vector search as a reusable platform capability rather than every use case implementing its own search engine. MiDAS documentation identifies vector search as one of the most reused platform building blocks across use cases.

---

# 25. Important Engineering Trade-offs

## Embedding model

```text
Large model
   ↓
Potentially better semantic quality
   ↓
Higher compute / latency / cost

Small model
   ↓
Faster and cheaper
   ↓
Potentially lower semantic quality
```

The correct choice should be validated using retrieval evaluation.

---

## Top-K

```text
Small K
→ Fast
→ Less noise
→ Risk missing candidates

Large K
→ More candidates
→ Better recall potential
→ More computation / noise
```

---

## Similarity threshold

A threshold can prevent very weak matches from being shown.

But:

```text
Threshold too high
→ False negatives

Threshold too low
→ Too many weak matches
```

Therefore thresholds should be determined through evaluation rather than arbitrary values.

---

# 26. What Can Go Wrong?

A good Senior AI Engineer should think beyond the happy path.

### Problem 1 — Poor embedding quality

Semantically similar requirements may not be retrieved.

### Problem 2 — Too many false positives

Requirements with similar vocabulary but different meanings may be returned.

### Problem 3 — Too strict threshold

Good candidates may be discarded.

### Problem 4 — Stale index

Recently added requirements may not appear in search.

### Problem 5 — Large corpus

Search performance may degrade if the vector infrastructure is not properly designed.

### Problem 6 — Metadata mismatch

A requirement may be semantically similar but belong to an irrelevant project or engineering scope.

This is why semantic similarity should generally be combined with appropriate metadata and business rules.

---

# 27. What I Would Say in an Interview

### 30-second answer

> "RSA, or Requirement Similarity Assist, is a MiDAS use case designed to help automotive engineers find semantically similar requirements across existing Bosch engineering projects. The core idea is to represent requirements as embeddings and use vector similarity search to retrieve the most relevant existing requirements. This is different from keyword search because semantically equivalent requirements can use completely different wording. RSA then presents the relevant candidates to the engineer so they can assess reuse, duplication or relationships. It leverages reusable MiDAS capabilities such as Space management, embeddings, vector search, LLM inference and analytics."

---

# 28. 2-Minute Architecture Answer

> "At a high level, RSA has two logical phases. The first is the indexing phase, where existing requirement data is prepared and converted into embeddings so it can be searched efficiently. Those embeddings are stored in the vector-search infrastructure along with the relevant requirement information and metadata.
>
> At runtime, an engineer provides a new requirement. RSA generates an embedding for that query and performs vector similarity search against the indexed requirements. The system retrieves the strongest candidates, applies the relevant ranking or analysis logic, and presents similar requirements to the engineer.
>
> The important architectural point is that RSA doesn't build all of this infrastructure itself. It is a MiDAS use case that composes reusable platform capabilities such as Space management, embeddings, vector search, LLM inference and analytics. This allows the use case to focus on requirements-specific business logic while the platform provides the common AI infrastructure."

---

# 29. Interview Questions to Be Ready For

## Foundation

1. What is RSA?
2. What business problem does RSA solve?
3. What is a requirement?
4. Who uses RSA?
5. Why is semantic similarity useful for requirements?

## Architecture

6. Where does RSA fit inside MiDAS?
7. What MiDAS services does RSA use?
8. What is the end-to-end RSA workflow?
9. What happens during indexing?
10. What happens when a user submits a new requirement?

## Embeddings

11. Why do we need embeddings?
12. Why not use keyword search?
13. Why was `all-MiniLM-L6-v2` considered suitable?
14. How would you evaluate a different embedding model?
15. What happens if the embedding model is poor?

## Vector Search

16. What is vector similarity search?
17. What is Top-K?
18. What similarity metric is being used?
19. How does vector search scale to millions of requirements?
20. What is the difference between a vector database and a retriever?

## Production

21. How are new requirements added to the index?
22. How do you handle stale embeddings?
23. How would you scale the indexing pipeline?
24. How would you handle millions of requirements?
25. How would you monitor retrieval quality?

## AI / LLM

26. Why is an LLM needed if vector search already finds similar requirements?
27. What role does the LLM play in RSA?
28. How would you evaluate the LLM component?
29. How would you prevent hallucination?
30. Would you call RSA a RAG application?

---

# 30. The Five Things to Remember

If you forget everything else, remember these five points:

### 1. RSA solves a requirements-engineering problem

> Find similar / related requirements.

### 2. RSA is primarily a semantic retrieval system

> It is not simply keyword search.

### 3. Embeddings are the bridge between text and semantic search

```text
Requirement → Embedding → Vector
```

### 4. Vector search finds the candidates

```text
Query Vector → Similarity Search → Top-K Requirements
```

### 5. RSA is built on MiDAS

> RSA combines reusable MiDAS capabilities rather than implementing the entire AI infrastructure itself.

---

# 31. Final Interview Mental Model

```text
                    RSA
                     │
                     ▼
          Requirement Similarity
                     │
          ┌──────────┴──────────┐
          │                     │
       OFFLINE                ONLINE
      INDEXING                QUERY
          │                     │
          ▼                     ▼
   Existing Requirements   New Requirement
          │                     │
          ▼                     ▼
    Prepare / Extract      Query Embedding
          │                     │
          ▼                     ▼
      Embeddings          Vector Similarity
          │                     │
          ▼                     ▼
     Vector Index          Top-K Results
          │                     │
          └──────────┬──────────┘
                     ▼
             Ranking / Analysis
                     │
                     ▼
           Similar Requirements
                     │
                     ▼
             Engineer Review
```

### One-line memory aid

> **RSA = Requirements → Embeddings → Vector Search → Similar Requirements → Engineer Decision.**

---

# 32. What We Should Learn Next

This document gives you the **Level-1 / Level-2 understanding**.

The next logical layer should be:

```text
RSA High Level
     ↓
Actual RSA Workflow
     ↓
RSA Components
     ↓
Batch / Indexing Pipeline
     ↓
Runtime API Flow
     ↓
Vectorization
     ↓
Similarity Search
     ↓
LLM / Scoring
     ↓
Deployment & Scaling
```

I would **not jump directly into APIs or source code yet**.

The most valuable next step is to take the actual RSA developer guide and reconstruct the **current end-to-end workflow** precisely—especially the distinction between the **offline/reference-project indexing flow** and the **online user query flow**. That will turn this high-level document into a reliable representation of the system you actually worked on.