---
title: Retrieval System(based on FAISS and Embedding Model)
published: 2025-04-02
tags: [LLM]
category: Arxiv
draft: false
---

# Retrieval System(based on FAISS and Embedding Model)

## FAISS Vector Search (Top-k ROR entries)

This part explains the process of building the vector index from the ROR dataset.

###  What is an Embedding Model?

Embedding models convert natural language (e.g., institution names) into numerical vectors. These vectors encode semantic similarity — so that similar names have closer vector representations.

- **Model Used**: `all-MiniLM-L6-v2`
- **Vector Size**: 384-dim

For example:

- "MIT" → `[0.1, -0.2, ..., 0.8]`
- "Massachusetts Institute of Technology" → `[0.12, -0.19, ..., 0.79]`

Even though their text is different, the vectors are close because the model understands their **semantic equivalence**.

### Why do "MIT" and "Massachusetts Institute of Technology" have similar vectors?

Because models like MiniLM have been pre-trained on massive public corpora (Wikipedia, arXiv, news, etc.).

They've seen these terms used interchangeably in similar contexts, such as:

- "MIT (Massachusetts Institute of Technology)"
- "The research was conducted at MIT."
- "Founded in 1861, the Massachusetts Institute of Technology..."

So during training, the model **learns** they represent the same concept.

> Embedding ≠ literal word matching. It's a meaning-based representation.

When we do vector search, FAISS retrieves the entries whose **embedding vectors** are closest to the input.

###  What is FAISS?

**FAISS** (Facebook AI Similarity Search) is a **high-performance vector search library** developed by Facebook, optimized for **searching through millions of dense vectors**.

Its role in a RAG system is to:

> Quickly find the **top-k most similar vectors** to a given query vector.

------

####  Example:

1. Suppose you’ve already **encoded 100,000 institution names** into vectors using a model like `MiniLM`, each vector being 384-dimensional.

2. You want to search for `"MIT"` →
    It is first embedded as:

   ```text
   [0.12, -0.33, ..., 0.28]
   ```

3. Then you query FAISS:

> “Find the top 5 most similar vectors to this one!”

------

#### FAISS returns top-k similar results:

```text
1. Massachusetts Institute of Technology — https://ror.org/012345678
2. MIT Media Lab — https://ror.org/0abcxyz12
3. MIT CSAIL — https://ror.org/08zdef123
4. Media Lab MIT — https://ror.org/01mnplv95
5. MIT Lincoln Laboratory — https://ror.org/07uvm9831
```

These are **semantically close entries** in the vector space.

### Relationship Between MiniLM and FAISS

```
    A[Input: "MIT"] --> B[MiniLM Embedding<br>→ [0.12, -0.33, ..., 0.28]]
    B --> C[FAISS Vector Search] --> [FAISS index]
    C --> D[Return Top-k Similar ROR Entries]
```

###  entity linking system based on the RAG approach

```
graph TD
    A[Input: Institution Name] --> B[FAISS Vector Search Top-k ROR entries]
    B --> C[Prompt Template + Institution + ROR Context]
    C --> D[Gemini (via VertexAI)]
    D --> E[Return Best ROR ID]
```

