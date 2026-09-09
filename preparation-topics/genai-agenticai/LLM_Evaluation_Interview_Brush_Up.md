# LLM Evaluation — Principal / Staff AI Engineer Interview Brush-Up

> **Mental model:** LLM Evaluation = systematically measuring whether an AI application behaves as expected across correctness, relevance, grounding, retrieval, tool usage, safety, and other task-specific criteria.

---

## 1. Core Evaluation Flow

```text
Evaluation Dataset / Goldens
          │
          ▼
    Execute Application
          │
          ├── Response
          ├── Retrieved Context
          └── Tool Calls
          │
          ▼
       Evaluators
          │
   ┌──────┼────────┐
   ▼      ▼        ▼
 Rules  LLM Judge  Human
          │
          ▼
       Metrics
          │
          ▼
   Regression / Release
```

### Two phases

**1. Execution:** Run application against evaluation dataset and capture outputs, context and tool calls.

**2. Grading:** Evaluate those results against ground truth / rubrics and generate scores.

---

# 2. Golden vs Dataset

### Golden

One atomic evaluation test case.

```text
Golden
├── User Query
├── Ground Truth / Reference Answer
├── Expected Tool
├── Actual Response
├── Actual Tool
└── Retrieved Context
```

### Dataset

Collection of Goldens.

A good dataset should contain:

- Happy paths
- Edge cases
- Ambiguous queries
- Failure cases
- Domain-specific cases
- Historical production failures

> **Key insight:** Evaluation quality depends heavily on dataset quality.

---

# 3. Evaluation Methods

| Method | Best For | Strength | Limitation |
|---|---|---|---|
| **Rule-based** | Tool selection, schema, exact values | Cheap, deterministic | Poor semantic understanding |
| **Statistical** | Lexical similarity / specific tasks | Simple | Weak for open-ended semantics |
| **LLM-as-Judge** | Semantic quality | Scalable | Bias, cost, consistency |
| **Human** | Domain/subjective evaluation | High quality | Expensive, slow |

### Best practice

Use a **layered approach** rather than relying on one evaluator.

```text
Deterministic checks
        +
Metrics
        +
LLM-as-Judge
        +
Human calibration
```

---

# 4. LLM-as-a-Judge

A capable LLM evaluates another model's output against a rubric.

```text
Query
Reference
Context
Candidate Answer
      │
      ▼
  Judge LLM
      │
      ▼
 Score + Reason
```

### Advantages

- Scalable
- Faster than humans
- Handles semantic evaluation
- Supports open-ended answers

### Risks

- Judge bias
- Prompt sensitivity
- Inconsistent scoring
- Cost
- Judge hallucination

> **Important:** The evaluator itself needs calibration against human/SME judgments.

---

# 5. Core RAG / LLM Metrics

## 5.1 Faithfulness

**Question:** Is the answer supported by the retrieved context?

```text
Retrieved Context → Answer
```

Primarily measures **grounding / hallucination**.

---

## 5.2 Answer Relevancy

**Question:** Does the response actually address the user's question?

Important:

> **Relevancy ≠ Correctness**

An answer can be relevant but factually wrong.

---

## 5.3 Answer Correctness

**Question:** Is the generated answer correct compared with the reference / ground truth?

```text
Generated Answer ↔ Ground Truth
```

---

## 5.4 Context Precision

**Question:** Are the retrieved results relevant and appropriately ranked?

Think:

> **Quality / ranking of retrieved context**

---

## 5.5 Context Recall

**Question:** Did retrieval capture all information necessary to answer the query?

Think:

> **Completeness of retrieved context**

---

## 5.6 Tool Correctness

**Question:** Did the agent select the expected tool?

```text
Expected Tool ↔ Actual Tool
```

Often suitable for deterministic evaluation.

---

# 6. The Most Important Metric Distinctions

### Faithfulness vs Correctness

```text
Answer ──────► Retrieved Context
   │
   └── Faithfulness

Answer ──────► Ground Truth
   │
   └── Correctness
```

**Faithfulness:** Is the answer supported by context?

**Correctness:** Is the answer actually correct?

Therefore:

> **Faithfulness ≠ Correctness**

An LLM can faithfully summarize incorrect/incomplete context.

---

### Context Precision vs Recall

| Metric | Question |
|---|---|
| **Precision** | Did we retrieve relevant information with good ranking? |
| **Recall** | Did we retrieve all information needed? |

Memory shortcut:

> **Precision = quality**  
> **Recall = completeness**

---

# 7. RAG Evaluation

Evaluate retrieval and generation separately.

```text
                    RAG
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      Retrieval             Generation
          │                     │
          ▼                     ▼
Context Precision          Faithfulness
Context Recall             Answer Correctness
                            Answer Relevancy
```

This makes failures diagnosable.

### Example

```text
Context Recall = Low
Faithfulness   = Low
Correctness    = Low
```

Likely:

> **Retrieval problem**

Whereas:

```text
Context Recall = High
Faithfulness   = Low
Correctness    = Low
```

Likely:

> **Generation / grounding problem**

---

# 8. Agent Evaluation

For agents, evaluate both **process and outcome**.

```text
Agent
 │
 ├── Tool Selection
 ├── Tool Arguments
 ├── Tool Sequence
 ├── Task Completion
 └── Final Answer
```

Key metrics:

- Tool correctness
- Tool argument correctness
- Task completion
- Answer correctness
- Answer relevancy

---

# 9. Evaluation Frameworks

| Framework | Primary Focus |
|---|---|
| **Ragas** | RAG evaluation |
| **DeepEval** | Agentic / multi-agent evaluation |
| **LangSmith** | Observability + experimentation + evaluation |
| **Pydantic Logfire** | Structured tracing / schema-centric evaluation |
| **Google ADK** | Structured agent / enterprise ecosystem |

### Interview answer

Don't say:

> "I always use Ragas."

Instead:

> "I select evaluation tooling based on the architecture—Ragas for RAG-specific evaluation, agent-oriented frameworks for agent workflows, and observability platforms for tracing and experimentation."

---

# 10. Enterprise Evaluation Architecture

```text
                 Evaluation Dataset
                         │
                         ▼
                  AI Application
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
         Retrieval      Agent       LLM
             │           │           │
             ▼           ▼           ▼
          Context      Tools       Answer
             │           │           │
             └───────────┼───────────┘
                         ▼
                    Evaluators
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
          Rules       LLM Judge     Human
                         │
                         ▼
                      Metrics
                         │
                         ▼
                Regression / Release
```

---

# 11. Evaluation Feedback Loop

Evaluation should become part of the development lifecycle.

```text
Prompt / Code / Model Change
            │
            ▼
     Offline Evaluation
            │
            ▼
      Regression Check
            │
        ┌───┴───┐
       PASS    FAIL
        │        │
        ▼        ▼
     Deploy     Fix
        │
        ▼
 Production Monitoring
        │
        ▼
 Production Failure
        │
        ▼
    New Golden
        │
        └──────► Evaluation Dataset
```

> **Key principle:** Every important production failure should ideally become a regression test.

---

# 12. Evaluation Is Multi-Dimensional

Never rely on a single quality score.

```text
                  AI Quality
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Quality        Safety         Cost
       │             │             │
 Correctness     Guardrails      Tokens
 Relevancy       Policy         Latency
 Faithfulness    PII            Throughput
```

Depending on the application, evaluate:

- Quality
- Safety
- Cost
- Latency
- Reliability
- Tool correctness
- Task completion

---

# 13. Principal / Staff Interview Framework

If asked:

> **"How would you design an LLM evaluation system?"**

Answer in this order:

### 1. Define "good"

```text
Correctness
Relevancy
Grounding
Safety
Task completion
Tool usage
```

### 2. Build representative Goldens

```text
Production data
+ SME examples
+ Edge cases
+ Historical failures
```

### 3. Capture complete traces

```text
Input
Context
Tool calls
LLM calls
Response
```

### 4. Evaluate components separately

```text
Retrieval
Agent
Generation
```

### 5. Use layered evaluators

```text
Rules → Metrics → LLM Judge → Human
```

### 6. Define thresholds

Example:

```text
Faithfulness > 0.90
Correctness  > 0.85
Tool Accuracy > 0.95
```

### 7. Run regression tests

On:

- Model changes
- Prompt changes
- Retriever changes
- Embedding changes
- Chunking changes
- Tool changes

### 8. Feed production failures back into the dataset.

---

# 14. Top Interview Questions

### Q1. What is LLM evaluation?

> Systematically measuring whether an LLM application satisfies defined quality criteria such as correctness, relevance, grounding, safety and task completion.

### Q2. What is a Golden?

> One atomic evaluation test case containing the user query, expected behavior/reference answer and, where applicable, expected tool usage.

### Q3. Faithfulness vs Correctness?

> Faithfulness checks whether the answer is supported by retrieved context. Correctness checks whether it matches the expected ground truth.

### Q4. Context Precision vs Recall?

> Precision measures relevance/ranking of retrieved information; recall measures whether all necessary information was retrieved.

### Q5. Why use LLM-as-a-Judge?

> It provides scalable semantic evaluation for open-ended outputs, but must be calibrated because the judge can have bias and inconsistency.

### Q6. Would you use only LLM-as-a-Judge?

> No. I would combine deterministic checks, metrics, LLM judging and periodic human validation.

### Q7. How would you evaluate RAG?

> Separately evaluate retrieval using context precision/recall and generation using faithfulness, correctness and relevancy.

### Q8. How would you evaluate an agent?

> Evaluate tool selection, tool arguments, tool sequence, task completion and final-answer quality.

### Q9. How do you handle production failures?

> Convert important production failures into Goldens and add them to the regression dataset.

### Q10. How do you prevent prompt/model changes from degrading quality?

> Run the evaluation dataset as a regression suite and compare the complete metric profile before release.

---

# 15. One-Minute Interview Answer

> "I treat LLM evaluation as an engineering measurement system rather than just checking whether an answer looks good. First I define what good means for the application—correctness, relevance, grounding, safety, task completion and tool usage. Then I create a representative dataset of Goldens containing queries, reference answers and expected behavior. I execute the application and capture the full trace including retrieval, context, tool calls and final response. I use deterministic rules for structural checks, retrieval metrics such as context precision and recall, generation metrics such as faithfulness and correctness, and LLM-as-a-Judge for semantic quality. I calibrate automated evaluation with human SMEs and put regression thresholds around the important metrics. Finally, I feed production failures back into the evaluation dataset so evaluation continuously improves."

---

# 16. Final Cheat Sheet

```text
Golden
  → One test case

Dataset
  → Collection of Goldens

Faithfulness
  → Answer supported by context?

Correctness
  → Answer matches ground truth?

Relevancy
  → Answer addresses the question?

Context Precision
  → Relevant / well-ranked retrieval?

Context Recall
  → All required information retrieved?

Tool Correctness
  → Correct tool selected?

LLM-as-Judge
  → Scalable semantic evaluation

Human Evaluation
  → Calibration + subjective/domain validation

Ragas
  → RAG

DeepEval
  → Agents

LangSmith
  → Tracing + experimentation + evaluation

Core Engineering Loop
  → Dataset → Execute → Evaluate → Regression → Production
                         ↑                    │
                         └── New Goldens ─────┘
```

## The 5 Things to Remember

> **1. Evaluate the whole system, not just the LLM.**

> **2. Separate retrieval, agent behavior and generation.**

> **3. Use multiple evaluators—not only LLM-as-a-Judge.**

> **4. Goldens and datasets are the foundation of evaluation quality.**

> **5. Turn production failures into regression tests.**