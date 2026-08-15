# RAG Interview Brush-Up Guide

Retrieval-Augmented Generation — Foundation to Interview-Ready Concepts

*Based on the RAG knowledge-sharing session*

---

## 1. RAG at a Glance

RAG (Retrieval-Augmented Generation) combines information retrieval with an LLM. Instead of relying only on the model's internal knowledge, the system retrieves relevant information from an external knowledge base and provides that information to the LLM as context.

**Core mental model:**

`Documents → Parsing → Chunking → Embeddings → Vector Store → Retriever → Context Construction / Prompting → LLM → Final Answer`

---

## 2. RAG Has Two Logical Phases

### A. Indexing / Ingestion

`Documents → Parsing → Chunking → Embeddings → Vector Store`

This phase prepares the knowledge base so that relevant information can be found efficiently later.

### B. Query / Retrieval

`User Query → Retriever → Relevant Chunks → Context Construction / Prompt → LLM → Answer`

This phase finds the information relevant to the user's question and uses it to generate the answer.

---

## 3. Documents

Documents are the source of knowledge for the RAG system.

**Examples:** PDFs, Word files, web pages, product manuals, company policies, database records.

> **Interview explanation:** "The first step is to identify and collect the knowledge sources that the RAG system needs to answer questions from."

---

## 4. Parsing / Loading

Parsing extracts usable text or structured content from different document formats. A PDF, for example, is converted into text and/or structured elements that downstream processing can use.

**Examples of tools:** PyPDF, Unstructured, Docling, Apache Tika.

> **Interview explanation:** "Parsing converts different document formats into a usable text or structured representation."

---

## 5. Chunking

Chunking breaks large documents into smaller retrieval units. It is common in RAG, but it is not technically mandatory.

### Why chunk?

- Large documents are difficult or expensive to send in full for every query.
- Fine-grained chunks allow the retriever to return only the relevant section.
- Smaller retrieval units generally improve retrieval precision.
- Chunking helps RAG scale across large collections of documents.

### When chunking may not be needed

If the knowledge units are already small and self-contained, such as short FAQs or small database records, each record can be treated as a retrieval unit without additional chunking.

Modern long-context LLMs also make whole-document approaches practical for some moderate documents, especially when the question requires understanding the document globally.

---

## 6. Common Chunking Strategies

### 6.1 Fixed-size chunking

Split content into chunks of a configured size, often with overlap. It is simple, predictable, and useful for general-purpose or uniform text.

- **Example:** chunk size 500, overlap 50–100.
- **Best starting scenarios:** quick prototypes, large volumes of relatively uniform text, documents without reliable structure.
- **Limitation:** can cut through sentences or logical sections.

### 6.2 Recursive chunking

LangChain's commonly used class is `RecursiveCharacterTextSplitter`. It recursively tries a hierarchy of separators, starting with larger/natural boundaries and moving to smaller ones when a piece is still too large.

**Typical default separator hierarchy:**

`"\n\n"` → `"\n"` → `" "` → `""`

**Conceptually:** paragraph boundary → newline → space/word boundary → individual character.

> **Important:** this is not AI/semantic intelligence. The splitter follows a predefined separator hierarchy. It does not inherently understand linguistic concepts such as "this is a sentence."

The separator list can be customized if sentence boundaries or domain-specific boundaries are desired.

**Example:**

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
chunks = splitter.split_text(text)
```

### 6.3 Semantic chunking

Attempts to create chunks around semantically coherent topics rather than using only fixed boundaries.

- **Useful when:** topic changes are important and preserving meaning is more important than uniform chunk size.
- **Trade-off:** more complex and potentially more computationally expensive.

### 6.4 Structure-aware / document-aware chunking

Uses document structure such as chapters, sections, headings, tables, or other logical boundaries.

Particularly useful for legal documents, policies, technical documentation, regulatory documents, and financial reports.

---

## 7. Choosing Chunk Size and Overlap

There is no universal magic chunk size. Choose a baseline based on the document type and the amount of context normally needed to answer one question, then validate it with retrieval evaluation.

A practical starting experiment might compare roughly 300–500, 500–800, and 800–1,000 token-sized chunks when using token-aware splitting. These are starting points, not rules.

**Overlap** helps preserve information that crosses chunk boundaries. A common starting point is around 10–20% of the chunk size.

- Too small → insufficient context.
- Too large → more irrelevant information and higher token/cost overhead.

If context is frequently split across chunks, increase overlap or consider structure-aware/semantic chunking.

> **Important LangChain detail:** `RecursiveCharacterTextSplitter` is character-based by default, so `chunk_size` should not automatically be interpreted as a token count. Token-aware length functions/configuration can be used when token-based limits are desired.

> **Interview answer:** "I would start with a baseline based on document characteristics, then evaluate retrieval quality. If relevant information is being split, I would increase overlap or use structure-aware chunking; if chunks contain too much irrelevant information, I would reduce chunk size."

---

## 8. Embeddings

An embedding model converts text into a numerical vector that captures semantic information.

**Example:** "How do I reset my password?" → embedding vector such as `[0.12, -0.45, 0.78, ...]`

Semantically similar text tends to have vectors that are close under the selected similarity/distance metric.

The same embedding model/configuration should generally be used consistently for documents and queries.

---

## 9. Vector Store / Vector Database

Stores document chunks, their embeddings, and usually metadata, and supports similarity search.

**Examples:** FAISS, ChromaDB, Pinecone, OpenSearch, pgvector.

Conceptually, a stored item contains: **Chunk + Embedding + Metadata**.

---

## 10. Similarity Measures

### 10.1 Cosine similarity

Measures the angle/direction between two vectors. It focuses on orientation rather than magnitude. It is commonly used for text embeddings.

Higher cosine similarity generally means more similar direction.

### 10.2 Dot product

Calculates the sum of element-wise products. It measures vector alignment while also being affected by magnitude.

**Example:** `[1, 2] · [2, 4] = (1×2) + (2×4) = 10`

A vector database can be configured to use dot product / inner product as its search metric. The exact supported metrics depend on the database and index configuration.

### 10.3 Euclidean distance

Measures straight-line distance between vectors. Smaller distance means closer vectors.

**Easy memory:** cosine = direction, dot product = alignment/magnitude relationship, Euclidean = physical distance.

### Cosine vs Dot Product — Key Interview Point

A vector database does not universally use cosine similarity. It may support cosine similarity, dot product, or Euclidean distance, depending on the system and configuration.

If vectors are normalized to unit length, cosine similarity and dot product produce the same ranking because the vector magnitudes are 1.

---

## 11. Retriever

The retriever takes the user's query and finds the most relevant pieces of information from the knowledge base.

**Typical vector retrieval flow:**

`User Query → Query Embedding → Similarity Search → Top-K Relevant Chunks`

The retriever does **not** generate the answer. It finds candidate information that the LLM can use.

> **Important distinction:** the vector database stores vectors and performs search; the retriever is the application-level retrieval mechanism that decides how to retrieve and may add filtering, thresholds, hybrid search, or reranking.

---

## 12. Retriever Controls and Advanced Retrieval

### 12.1 Metadata filtering

Restricts retrieval using attributes stored with chunks, such as department, country, document type, date, product, customer, or access permissions.

**Example:** search only chunks where `country = India` before/alongside semantic retrieval.

**Mental model:** "Search only in this subset."

### 12.2 Top-K

Specifies how many candidate results should be returned. For example, `K=5` means return the five highest-ranked results.

- K too small → potentially miss useful information.
- K too large → more irrelevant context, tokens, cost, and possible noise.

Top-K is normally tuned using evaluation rather than chosen as a universal value.

### 12.3 Score threshold

Rejects results whose similarity/relevance score is below a configured threshold.

This helps prevent weak results from being passed to the LLM even if they technically fall within Top-K.

> **Important:** score ranges and meaning depend on the similarity metric and implementation, so a value such as 0.8 is not a universal threshold.

### 12.4 Hybrid search

Combines semantic vector search with lexical/keyword search such as BM25.

- Vector search is useful for **meaning and semantic similarity**.
- Keyword search is useful for **exact terms, names, identifiers, product codes**, and other lexical matches.

**Mental model:** "Use both meaning and exact keywords."

### 12.5 Reranking

A reranker takes a set of retrieved candidates and reorders them using a more sophisticated relevance model.

**Typical flow:** initial retrieval → candidate chunks → reranker → best-ranked chunks → LLM.

**Key distinction:** Retriever finds candidates; reranker improves their ordering.

Reranking is especially useful when you retrieve many candidates but want to pass only the strongest few to the LLM.

---

## 13. Context Construction / Prompting

Context construction is the bridge between retrieval and the LLM. The retrieved chunks are organized and combined with the user question and instructions into an LLM-ready prompt.

**Typical structure:**

`System Instructions + Retrieved Context + User Question → LLM`

**Example instruction:** "Answer using the provided context. If the answer is not present, say that you do not have enough information."

Context construction can involve selecting the best chunks, removing duplicates, applying thresholds, preserving source metadata, formatting the chunks, and staying within the model's context window.

---

## 14. Long Context vs Chunking

Modern LLMs may support very large context windows, so chunking is not automatically mandatory for every document.

For a moderate 20–40 page document, **whole-document prompting** can be reasonable when:

- There is only one/few documents.
- The question requires global understanding of the document.
- The cost and latency are acceptable.
- The model's effective context capacity is sufficient for the actual document and prompt.

**Chunking and retrieval are usually preferable when:**

- The knowledge base contains many documents.
- Questions target specific sections.
- You need lower token usage and latency.
- You need fine-grained retrieval and scalable search.
- You want to reduce irrelevant context.

> **Strong interview framing:** "Large context windows give us another option; they do not eliminate the retrieval problem. For global questions over a moderate document, whole-document long-context prompting may be appropriate. For targeted questions or large knowledge bases, chunking and retrieval scale much better."

---

## 15. End-to-End Interview Mental Model

**INDEXING:**

`Documents → Parsing → Chunking → Embeddings → Vector Store`

**RETRIEVAL:**

`User Query → Query Embedding → Retrieval → Top-K / Filters / Threshold → Optional Reranking`

**GENERATION:**

`Retrieved Context + Instructions + User Query → LLM → Final Answer`

---

## 16. One-Line Definitions for Fast Revision

| Term | Definition |
|------|------------|
| **Documents** | Source knowledge used by the RAG system. |
| **Parsing** | Extract usable text or structure from source documents. |
| **Chunking** | Break documents into manageable retrieval units. |
| **Embedding** | Convert text into a numerical vector representing semantic information. |
| **Vector Store** | Store embeddings and support similarity search. |
| **Retriever** | Find relevant chunks for a user query. |
| **Top-K** | Return the best K retrieval candidates. |
| **Metadata filtering** | Restrict retrieval to records matching specific attributes. |
| **Score threshold** | Discard results below a relevance/similarity cutoff. |
| **Hybrid search** | Combine semantic vector search with keyword/lexical search. |
| **Reranking** | Reorder retrieved candidates using a stronger relevance model. |
| **Context construction** | Organize retrieved information with instructions and the query for the LLM. |
| **LLM** | Generate the final response using the supplied context and instructions. |

---

## 17. Interview Questions You Should Be Ready For

- Is chunking mandatory in RAG?
- Why do we chunk documents?
- What is `RecursiveCharacterTextSplitter` in LangChain?
- How does recursive chunking work?
- What are the common chunking strategies?
- How do you choose chunk size and chunk overlap?
- What is an embedding?
- What is the difference between a vector database and a retriever?
- What is Top-K retrieval?
- What is metadata filtering?
- What is a score threshold?
- What is hybrid search and why use BM25?
- What is reranking?
- What is the difference between a retriever and a reranker?
- Does a vector database use cosine similarity or dot product?
- When would you use whole-document long-context prompting instead of chunk-based RAG?

---

## 18. Final Interview Summary

The most important thing is to explain RAG as a flow rather than as a collection of isolated components. Documents are parsed and prepared for indexing. Large or complex documents are commonly chunked into meaningful retrieval units. Embeddings represent those chunks as vectors, which are stored in a vector database. At query time, the retriever uses the query to find relevant chunks, optionally applying metadata filters, Top-K limits, score thresholds, hybrid search, and reranking. The selected context is then combined with instructions and the user question and sent to the LLM for answer generation.

> **Interview principle:** there is rarely one universally correct configuration. Chunk size, overlap, Top-K, similarity metric, filtering, reranking, and even whole-document vs chunk-based retrieval should be chosen based on document characteristics, use case, scalability, cost/latency, and evaluation results.