---
name: rag
description: Build a RAG pipeline via ai-ml-engineer. Ingestion, retrieval, generation, eval.
allowed-tools: ["Read", "Write", "Bash", "Glob", "WebFetch"]
argument-hint: "<describe the RAG use case: documents, questions, output format>"
---

<objective>
Build a RAG pipeline for: $ARGUMENTS

Deliver: ingestion pipeline, vector store setup, retrieval logic, generation, RAGAS eval.
</objective>

<execution_context>
@agents/tier2/ai-ml-engineer.md
@skills/rag-pipeline.md
@skills/research-first.md
</execution_context>

<context>
**RAG Use Case**: $ARGUMENTS
</context>

<process>
1. Research existing AI/LLM setup in codebase
2. Design pipeline: document types → chunking → embedding → vector DB → retrieval → generation
3. Implement pipeline
4. Set up evaluation with DeepEval/RAGAS
5. Document usage
</process>
