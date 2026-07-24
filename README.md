# Compliance-Preserving AI Retrieval System for Aircraft Maintenance Manuals

An AI-powered semantic retrieval system that helps Aircraft Maintenance Technicians (AMTs) quickly locate certified maintenance procedures using natural-language queries while preserving regulatory compliance.

Instead of replacing OEM maintenance manuals or generating maintenance instructions, the system acts as an intelligent search layer that retrieves, ranks, and previews the most relevant maintenance tasks before opening the official certified documentation.

---

## Overview

Aircraft maintenance manuals contain thousands of highly structured procedures organized using Air Transport Association (ATA) chapters. Technicians often spend significant time locating the correct task because procedures are deeply nested and many have similar names.

This project improves maintenance task retrieval by combining semantic search, multilingual embeddings, and LLM-based re-ranking while ensuring technicians always reference the original certified manual.

---

## Problem

Aircraft technicians frequently encounter challenges such as:

- Large maintenance manuals containing thousands of tasks
- Deep ATA chapter hierarchies
- Similar task names across aircraft systems
- Traditional keyword search missing relevant procedures
- Strict aviation regulations preventing AI-generated maintenance instructions

The objective was to improve search efficiency without modifying certified OEM documentation.

---

# Solution

Built a compliance-preserving retrieval system that:

- Converts aircraft maintenance manuals into structured task knowledge
- Creates revision-robust semantic embeddings using ATA hierarchy and task metadata
- Performs dense semantic retrieval
- Uses an LLM only for candidate re-ranking
- Opens the selected task directly in the certified OEM viewer
- Preserves full traceability and regulatory compliance

---

# System Architecture

```
                OFFLINE PIPELINE

Maintenance Manuals (PDF)
          │
          ▼
OCR & Layout Extraction
          │
          ▼
Vision Language Parsing
          │
          ▼
Task Knowledge Structuring
          │
          ▼
Embedding Generation
          │
          ▼
Task Knowledge Database



                ONLINE PIPELINE

Technician Query
        │
        ▼
Semantic Retrieval
        │
        ▼
Top-50 Candidate Tasks
        │
        ▼
LLM Re-ranking
        │
        ▼
Top Ranked Tasks
        │
        ▼
Task Preview
        │
        ▼
Certified OEM Viewer
```

---

# Key Features

- Semantic search for aircraft maintenance tasks
- Revision-robust embeddings using ATA metadata
- Natural-language search
- Multilingual retrieval (English & Korean)
- LLM-assisted ranking
- Compliance-preserving architecture
- Certified OEM viewer integration
- Rule-based document structuring
- Vision-Language document parsing
- Fail-safe dense retrieval fallback
- Task preview before document navigation

---

# Tech Stack

### Programming

- Python

### AI / Machine Learning

- Sentence Transformers
- BGE-M3 Multilingual Embeddings
- Qwen3
- Llama 3.3
- Qwen2.5-VL
- Dense Vector Retrieval

### NLP

- Semantic Search
- Embedding Models
- LLM Re-ranking
- Cross-lingual Retrieval

### Computer Vision

- Vision Language Models
- OCR
- Layout Extraction

### Data

- Aircraft Maintenance Manuals (AMM)
- Fault Isolation Manuals (FIM)
- ATA Hierarchy

---

# How Retrieval Works

1. Technician enters a natural-language maintenance problem.

Example:

```
Landing gear is not retracting properly
```

2. The embedding engine retrieves the most relevant maintenance tasks.

3. The LLM re-ranks candidate tasks using only:

- ATA IDs
- Task titles
- ATA hierarchy

No maintenance procedures are provided to the LLM.

4. The technician reviews the ranked task list.

5. The selected task opens directly inside the certified OEM maintenance manual.

---

# Safety & Compliance

Unlike traditional Retrieval-Augmented Generation (RAG) systems, this project never generates maintenance procedures.

The LLM:

- does not rewrite manuals
- does not summarize procedures
- does not modify certified documentation

Instead, it performs semantic ranking only, ensuring technicians always verify work using the official certified maintenance manuals.

---

# Project Results

## Synthetic Benchmark

- 8,229 maintenance tasks
- 49,643 evaluation queries
- 91.64% Hit@5 retrieval accuracy
- Robust against spelling mistakes
- Consistent >90% Hit@5 using compact LLMs

---

## Human Evaluation

- 10 licensed Aircraft Maintenance Technicians
- English and Korean queries
- 90.9% Top-10 retrieval success
- Average lookup time: **18 seconds**

---

## Operational Impact

Compared with traditional manual lookup:

- Lookup time reduced from 6–15 minutes to approximately 18 seconds
- More than 95% reduction in search time
- 96.6% of successful searches completed within one minute

---

# Challenges

Some retrieval failures occurred due to:

- Similar maintenance task titles
- Context-dependent procedures
- Aviation terminology ambiguity
- Cross-lingual translation differences

Future improvements include incorporating procedural context and richer multilingual support.

---

# Future Improvements

- Context-aware retrieval
- Multimodal maintenance queries
- Cross-document linking
- Technician feedback learning
- Interactive maintenance assistant

---

# Repository Structure

```
Maintenance-Task-Search-System/

├── data/
├── embeddings/
├── parser/
├── retrieval/
├── reranker/
├── evaluation/
├── notebooks/
├── app/
├── docs/
└── README.md
```

---

# Resume Summary

Built an AI-powered semantic retrieval system for aircraft maintenance technicians using multilingual embeddings, dense vector search, Vision-Language document parsing, and LLM-based re-ranking to accelerate certified maintenance task discovery while preserving regulatory compliance.
