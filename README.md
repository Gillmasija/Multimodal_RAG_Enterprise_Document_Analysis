# Multimodal RAG – Design & Enterprise Considerations

This document explains the key design decisions behind the Multimodal Retrieval-Augmented Generation (RAG) pipeline, with emphasis on chunking, retrieval, safety, scalability, and enterprise deployment.

---

## 1. Chunking Design for Tables

Tables contain dense numerical information that can easily lose meaning if split arbitrarily. To avoid this, structured data (CSV and Excel sources) is chunked using **row-group chunking**.

**Design approach:**
- Tables are divided into fixed row groups (e.g., 25–30 rows per chunk)
- Column headers are preserved in every chunk
- Each chunk is annotated with metadata such as:
  - Source file
  - Row range (e.g., rows 0–29)

**Rationale:**
- Preserves numeric relationships within rows
- Enables accurate retrieval of financial and operational metrics
- Prevents hallucination caused by partial or context-less numeric values

This approach mirrors how enterprise analytics systems treat tabular data as authoritative structured sources rather than free text.

---

## 2. Retrieval Decision Process

The system uses a **hybrid retrieval strategy** to balance precision and semantic relevance.

**Retrieval components:**
- **BM25 (keyword search):**
  - Captures exact matches for airline names, regions, currencies, and identifiers
- **Semantic search (embeddings + FAISS):**
  - Captures contextual similarity for descriptive or analytical queries

**Ranking strategy:**
- Results from both retrievers are normalized
- A weighted hybrid score is computed:
  - 70% semantic similarity
  - 30% keyword relevance
- Top-K chunks are selected based on the combined score

**Explainability:**
Each retrieved chunk includes a `why_retrieved` explanation indicating whether it was selected due to:
- High semantic similarity
- Keyword overlap
- Or both

This transparency is critical for trust and auditability in enterprise AI systems.

---

## 3. Hallucination Prevention

To ensure dependable outputs, the system enforces multiple safety checks before generating an answer.

**Safeguards include:**
- **Minimum evidence requirement:**  
  At least a fixed number of chunks must be retrieved
- **Confidence threshold:**  
  Average hybrid score must exceed a defined minimum
- **Key-term coverage:**  
  Important terms from the user query must appear in the retrieved evidence
- **Refusal logic:**  
  If any check fails, the system refuses to answer and returns an explanatory message

**Outcome:**
- Answers are generated **only from retrieved evidence**
- Unsupported or speculative questions result in a safe refusal
- Prevents confident but ungrounded responses

---

## 4. Scaling Approach for Millions of Documents

For large-scale enterprise deployments, the architecture supports horizontal scaling.

**Scalability strategies:**
- Sharding vector indexes by document type, tenant, or time range
- Incremental ingestion to avoid full re-indexing
- Metadata-based filtering to reduce retrieval scope
- Asynchronous and batched embedding generation
- Caching frequent queries and embeddings

These techniques allow the system to scale from thousands to millions of documents while maintaining retrieval performance.

---

## 5. Adjustments for On-Prem Enterprise Clients

Enterprise environments often require strict security and deployment controls.

**On-prem adaptations include:**
- Support for private or self-hosted embedding and LLM models
- Air-gapped deployment without external network access
- Role-based access control (RBAC) on documents and queries
- Audit logs for retrieved chunks and generated answers
- Encryption at rest and in transit

These adjustments ensure compliance with enterprise security, privacy, and governance requirements.

---

## Summary

This Multimodal RAG system is designed with enterprise constraints in mind:
- Tables are handled as structured, authoritative sources
- Retrieval is hybrid, explainable, and auditable
- Hallucinations are actively prevented
- The architecture scales to large document collections
- Deployment can be adapted to on-prem and regulated environments

The result is a reliable and enterprise-ready document analysis pipeline.
