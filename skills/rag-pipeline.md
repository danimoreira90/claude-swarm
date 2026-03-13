---
name: rag-pipeline
description: >
  End-to-end RAG pipeline skill. Ingestion → chunking → embedding → storage → retrieval → generation → eval.
  Covers Qdrant, ChromaDB, FAISS, OpenAI embeddings, LangChain, RAGAS evaluation.
  Use for any semantic search, Q&A, or document chat feature.
---

# RAG Pipeline Skill

> Retrieval-Augmented Generation: ground LLM responses in real data.

## Pipeline Architecture

```
Documents
    ↓
[Ingestion] → Parse, clean, deduplicate
    ↓
[Chunking] → Split into semantic units (1000 tokens, 200 overlap)
    ↓
[Embedding] → text-embedding-3-small / text-embedding-3-large
    ↓
[Storage] → Qdrant / ChromaDB / FAISS
    ↓
[Retrieval] → Dense + sparse hybrid, re-ranking
    ↓
[Generation] → LLM with retrieved context
    ↓
[Evaluation] → RAGAS / DeepEval metrics
```

## Implementation Checklist

### 1. Choose Vector Database

| DB | Best For |
|----|----------|
| **Qdrant** | Production, large scale, filtering, payload support |
| **ChromaDB** | Development, prototyping, local |
| **FAISS** | In-memory, no infrastructure, small-medium scale |
| **pgvector** | Already using PostgreSQL, simpler ops |

### 2. Chunking Strategy

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Default good settings:
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,      # tokens, not characters
    chunk_overlap=200,    # overlap prevents missing context at boundaries
    separators=["\n\n", "\n", ". ", " ", ""],  # respect document structure
)

# For code:
from langchain.text_splitter import Language
code_splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON,
    chunk_size=500,
    chunk_overlap=50,
)
```

### 3. Embedding Model Selection

| Model | Dimensions | Best For |
|-------|-----------|----------|
| `text-embedding-3-small` | 1536 | Balanced cost/quality |
| `text-embedding-3-large` | 3072 | Best quality |
| `nomic-embed-text` | 768 | Open source, local |
| `mxbai-embed-large` | 1024 | Open source, strong |

### 4. Retrieval Configuration

```python
# Hybrid search (dense + sparse) — better than dense alone
results = qdrant.search(
    collection_name="docs",
    query_vector=query_embedding,
    limit=10,           # retrieve more than you'll use
    score_threshold=0.7,  # filter low-relevance
    with_payload=True,
)

# Re-rank with cross-encoder for better precision
from sentence_transformers import CrossEncoder
reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')
scores = reranker.predict([(query, result.payload['text']) for result in results])
# Sort by reranker score and take top 5
top_results = [results[i] for i in sorted(range(len(scores)), key=lambda i: scores[i], reverse=True)][:5]
```

### 5. Prompt Engineering for RAG

```python
SYSTEM_PROMPT = """You are a helpful assistant. Answer questions ONLY based on the provided context.
If the context doesn't contain enough information to answer the question, say:
"I don't have enough information to answer that question based on the available documents."

Rules:
- Never make up information not present in the context
- Cite sources when possible
- If uncertain, express that uncertainty
"""

def build_rag_prompt(question: str, chunks: list[dict]) -> str:
    context = "\n\n---\n\n".join([
        f"[Source: {c['source']}, Page: {c.get('page', 'N/A')}]\n{c['text']}"
        for c in chunks
    ])
    return f"Context:\n{context}\n\nQuestion: {question}"
```

## RAGAS Evaluation

```python
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_precision,
    context_recall,
)
from datasets import Dataset

# Build evaluation dataset
eval_data = {
    "question": ["What is the return policy?", ...],
    "answer": [rag_pipeline.query(q) for q in questions],
    "contexts": [rag_pipeline.retrieve(q) for q in questions],
    "ground_truth": ["30-day return policy for unused items", ...],
}

dataset = Dataset.from_dict(eval_data)
results = evaluate(dataset, metrics=[faithfulness, answer_relevancy, context_precision, context_recall])
print(results)
# Expected: faithfulness ≥ 0.8, answer_relevancy ≥ 0.7
```

## Production Checklist

```
[ ] Chunking strategy matches document structure
[ ] Embeddings stored with rich metadata (source, date, section)
[ ] Collection has appropriate indexes for filtered search
[ ] Retrieval tested against real user queries
[ ] Evaluation dataset of 20+ Q&A pairs for regression
[ ] Hallucination rate < 10% (faithfulness metric)
[ ] Answer relevancy > 0.7 (RAGAS)
[ ] Latency p99 < 2s (retrieval + generation)
[ ] Observability: traces in Langfuse
[ ] Stale document refresh pipeline (re-embed when docs change)
```
