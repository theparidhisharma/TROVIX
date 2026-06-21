<div align="center">

# TROVIX

### **A Modern Information Retrieval Engine Built From Scratch**

*Building the core of a production-grade search engine through classical Information Retrieval, scalable indexing, and intelligent ranking.*

---

![Python](https://img.shields.io/badge/Python-3.12+-3776AB?style=for-the-badge\&logo=python\&logoColor=white)
![Status](https://img.shields.io/badge/Status-Under%20Development-success?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)
![Architecture](https://img.shields.io/badge/Architecture-Modular-orange?style=for-the-badge)
![Search](https://img.shields.io/badge/Search-BM25%20Powered-red?style=for-the-badge)

---

**TROVIX is a modular search engine developed from first principles to explore how modern information retrieval systems work.**

Instead of relying on existing search frameworks, the project focuses on implementing the fundamental building blocks that power large-scale search engines—including document parsing, tokenization, inverted indexing, posting list generation, BM25 ranking, and an end-to-end query execution pipeline.

The long-term vision is to evolve TROVIX from a classical lexical search engine into a scalable hybrid retrieval platform supporting semantic search, vector embeddings, AI-assisted retrieval, and distributed indexing.

</div>

---

# Table of Contents

* [Project Vision](#project-vision)
* [Why TROVIX?](#why-trovix)
* [Key Features](#key-features)
* [System Architecture](#system-architecture)
* [Project Structure](#project-structure)
* [Documentation](#documentation)
* [Technology Stack](#technology-stack)
* [Development Roadmap](#development-roadmap)
* [Team & Responsibilities](#team--responsibilities)
* [Getting Started](#getting-started)
* [Performance Goals](#performance-goals)
* [Contributing](#contributing)
* [Acknowledgements](#acknowledgements)

---

# Project Vision

Search engines are among the most influential software systems ever built.

Every search request made on the internet passes through complex pipelines involving indexing, ranking, query optimization, distributed systems, and increasingly, artificial intelligence.

TROVIX exists to understand and implement these systems from the ground up.

Rather than using existing search libraries as a black box, every major subsystem is designed and implemented independently, allowing contributors to understand not only **how** search engines work, but **why** they are designed the way they are.

The project combines concepts from:

* Information Retrieval
* Data Structures
* Algorithms
* Software Architecture
* Distributed Systems
* Databases
* Machine Learning
* Artificial Intelligence

The result is a modular platform that can continuously evolve as new retrieval techniques emerge.

---

# Why TROVIX?

Most developers interact with search engines through tools such as Elasticsearch or OpenSearch.

While these platforms are incredibly powerful, they abstract away the internal algorithms that make efficient search possible.

TROVIX takes the opposite approach.

Every subsystem is implemented deliberately, documented extensively, and designed to be understandable.

The project prioritizes engineering clarity over hidden abstractions.

This repository serves as both a working search engine and an educational reference for Information Retrieval systems.

---

# Core Objectives

The project has five primary objectives.

## Build a Search Engine From Scratch

Implement every major retrieval component without relying on external search frameworks.

---

## Learn Information Retrieval

Understand the algorithms behind indexing, ranking, and search.

---

## Design Production-Quality Software

Follow modular architecture, documentation-first development, automated testing, and performance benchmarking.

---

## Create an Extensible Platform

Ensure future ranking algorithms and retrieval techniques can be integrated without redesigning the entire system.

---

## Bridge Classical IR and Modern AI

Combine traditional lexical search with future semantic retrieval, hybrid search, and Retrieval-Augmented Generation (RAG).

---

# Key Features

## Current Features

* Custom Document Parser
* Modular Tokenization Pipeline
* Inverted Index Construction
* Efficient Posting List Design
* BM25 Ranking Algorithm
* Query Execution Pipeline
* Automated Testing Strategy
* Performance Benchmark Framework
* Comprehensive Technical Documentation

---

## Planned Features

* Phrase Search
* Boolean Queries
* Fuzzy Search
* Autocomplete
* Query Suggestions
* Persistent Index Storage
* Incremental Indexing
* Semantic Search
* Vector Embeddings
* Hybrid BM25 + Dense Retrieval
* Neural Re-Ranking
* Distributed Indexing
* Retrieval-Augmented Generation (RAG)

---

# Design Philosophy

TROVIX is built around several engineering principles.

* Documentation First
* Modular Architecture
* Deterministic Behaviour
* Test-Driven Development
* Performance-Oriented Design
* Extensibility
* Clear Separation of Responsibilities

Every subsystem has a single responsibility and communicates through well-defined interfaces.

This minimizes coupling and allows individual components to evolve independently.

---
# System Architecture

TROVIX follows a modular architecture where each subsystem has a clearly defined responsibility.

Rather than tightly coupling indexing, retrieval, ranking, and query execution, every module operates independently through well-defined interfaces. This separation simplifies development, testing, debugging, and future scalability.

The complete lifecycle of a search request is illustrated below.

```text
                           DOCUMENT INGESTION

 Raw Documents
        │
        ▼
┌──────────────────────────────┐
│      Document Parser         │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│   Tokenization Pipeline      │
│                              │
│ • Lowercasing                │
│ • Punctuation Removal        │
│ • Stopword Removal           │
│ • Stemming                   │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│       Index Builder          │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│      Inverted Index          │
│                              │
│ Vocabulary                   │
│ Posting Lists                │
│ Document Statistics          │
└──────────────────────────────┘
        │
═══════════════════════════════════════════════════════════
                    INDEX READY
═══════════════════════════════════════════════════════════
        │
        ▼

                      QUERY EXECUTION

 User Query
        │
        ▼
┌──────────────────────────────┐
│      Query Parser            │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ Query Tokenization Pipeline  │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ Candidate Retrieval          │
│ (Posting List Traversal)     │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│       BM25 Ranking           │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ Sort & Return Top Results    │
└──────────────────────────────┘
        │
        ▼

   Ranked Search Results
```

---

# Architecture Principles

The architecture is designed around a set of core engineering principles.

| Principle           | Description                                                                         |
| ------------------- | ----------------------------------------------------------------------------------- |
| **Modularity**      | Every subsystem performs one well-defined responsibility.                           |
| **Loose Coupling**  | Components communicate through clean interfaces rather than implementation details. |
| **Extensibility**   | Future algorithms can replace existing ones without redesigning the system.         |
| **Scalability**     | The architecture supports future distributed indexing and retrieval.                |
| **Determinism**     | Identical inputs always produce identical outputs.                                  |
| **Maintainability** | Independent modules simplify testing, debugging, and future development.            |

---

# Repository Structure

```text
TROVIX
│
├── docs/
│   ├── architecture/
│   │   ├── 01_inverted_index_architecture.md
│   │   ├── 02_posting_list_design.md
│   │   ├── 03_document_parser.md
│   │   ├── 04_tokenization_pipeline.md
│   │   ├── 05_index_builder.md
│   │   ├── 06_bm25_ranking.md
│   │   └── 07_query_execution_pipeline.md
│   │
│   ├── roadmap/
│   │   └── 01_future_improvements.md
│   │
│   └── testing/
│       ├── 01_testing_strategy.md
│       └── 02_performance_benchmark.md
│
├── src/
│   ├── parser/
│   ├── tokenizer/
│   ├── indexing/
│   ├── ranking/
│   ├── query_engine/
│   ├── crawler/
│   ├── api/
│   └── utils/
│
├── tests/
│
├── datasets/
│
├── benchmarks/
│
├── requirements.txt
│
└── README.md
```

---

# Documentation

The TROVIX documentation is intentionally extensive.

Every major subsystem is documented before implementation, ensuring architectural decisions are well understood and maintainable.

| Document                            | Purpose                                                            |
| ----------------------------------- | ------------------------------------------------------------------ |
| **01. Inverted Index Architecture** | High-level indexing architecture and data flow.                    |
| **02. Posting List Design**         | Internal storage structures for efficient retrieval.               |
| **03. Document Parser**             | Raw document ingestion and parsing workflow.                       |
| **04. Tokenization Pipeline**       | Text preprocessing, normalization, stopword removal, and stemming. |
| **05. Index Builder**               | Construction of the inverted index and vocabulary.                 |
| **06. BM25 Ranking**                | Mathematical foundations of lexical ranking.                       |
| **07. Query Execution Pipeline**    | End-to-end search request lifecycle.                               |
| **Testing Strategy**                | Validation methodology for every subsystem.                        |
| **Performance Benchmark**           | Performance evaluation and scalability metrics.                    |
| **Future Improvements**             | Product roadmap and long-term technical vision.                    |

---

# Technology Stack

| Category                 | Technologies                 |
| ------------------------ | ---------------------------- |
| **Programming Language** | Python 3.12+                 |
| **Database**             | PostgreSQL *(planned)*       |
| **Containerization**     | Docker *(planned)*           |
| **Ranking Algorithm**    | BM25                         |
| **Index Structure**      | Custom Inverted Index        |
| **Text Processing**      | Custom Tokenization Pipeline |
| **Testing**              | Pytest                       |
| **Benchmarking**         | Custom Benchmark Framework   |
| **Documentation**        | Markdown + Mermaid           |

---

# Information Retrieval Pipeline

TROVIX implements a classical Information Retrieval pipeline inspired by modern search systems.

```text
Raw Documents

↓

Document Parsing

↓

Tokenization

↓

Vocabulary Construction

↓

Posting List Generation

↓

Inverted Index

↓

Query Processing

↓

Candidate Retrieval

↓

BM25 Ranking

↓

Top-K Results
```

Every stage has been designed to remain independent, testable, and replaceable.

This modular design allows future integration of semantic retrieval, vector search, and neural reranking without modifying the existing indexing infrastructure.

---
# Team & Responsibilities

TROVIX is being developed as a collaborative engineering project, with each member owning a well-defined subsystem of the search engine.

Rather than dividing work arbitrarily, responsibilities are assigned according to the architecture of the system. Every major module has a dedicated owner responsible for its design, implementation, testing, and documentation.

This ownership model promotes accountability, modular development, and parallel implementation while minimizing integration conflicts.

---

## Project Leadership

### **Paridhi Sharma**

**Project Lead • Search Infrastructure & Information Retrieval Engineer**

Paridhi leads the technical direction of TROVIX and is responsible for designing the core architecture of the search engine.

Primary responsibilities include:

* Overall system architecture
* Technical planning and project coordination
* Information Retrieval research
* Custom Inverted Index design
* Posting List architecture
* Tokenization Pipeline
* Document Parser
* Index Builder
* BM25 Ranking implementation
* Query Execution Pipeline
* Technical documentation
* Testing strategy
* Performance benchmarking
* Repository organization
* Development standards and engineering practices

As Project Lead, Paridhi ensures that every subsystem integrates into a cohesive and scalable architecture while maintaining consistency across the codebase and documentation.

---

## Web Crawling & Data Collection

### **Alfaiz**

**Data Collection & Crawling Engineer**

Alfaiz is responsible for designing and implementing the document acquisition pipeline.

Primary responsibilities include:

* Web crawler architecture
* Website traversal strategies
* URL discovery
* Data collection pipeline
* HTML extraction
* Content cleaning
* Duplicate document detection
* Crawl scheduling
* Dataset generation
* Crawling performance optimization
* Integration with the indexing pipeline

The crawler serves as the primary source of documents entering the TROVIX indexing system.

---

## Backend & Infrastructure

### **Johan**

**Backend Infrastructure Engineer**

Johan is responsible for building the backend services that expose TROVIX to external applications.

Primary responsibilities include:

* REST API development
* Search API endpoints
* PostgreSQL schema design
* Database integration
* Docker containerization
* Backend deployment
* API documentation
* Request validation
* Response serialization
* Infrastructure configuration

These components provide the interface through which users and applications interact with the search engine.

---

## Ranking Research

### **Lakshay Yadav**

**Ranking & Retrieval Engineer**

Lakshay focuses on improving document ranking quality through research and experimentation.

Primary responsibilities include:

* Ranking algorithm research
* PageRank investigation
* Retrieval quality evaluation
* Ranking signal analysis
* Hybrid ranking exploration
* Benchmark comparison
* Relevance evaluation
* Future ranking improvements

His work establishes the foundation for future ranking systems that extend beyond classical BM25.

---

# Responsibility Matrix

| Component                | Owner              |
| ------------------------ | ------------------ |
| Project Planning         | **Paridhi Sharma** |
| System Architecture      | **Paridhi Sharma** |
| Repository Management    | **Paridhi Sharma** |
| Technical Documentation  | **Paridhi Sharma** |
| Document Parser          | **Paridhi Sharma** |
| Tokenization Pipeline    | **Paridhi Sharma** |
| Index Builder            | **Paridhi Sharma** |
| Inverted Index           | **Paridhi Sharma** |
| Posting Lists            | **Paridhi Sharma** |
| BM25 Ranking             | **Paridhi Sharma** |
| Query Execution Engine   | **Paridhi Sharma** |
| Testing Strategy         | **Paridhi Sharma** |
| Performance Benchmarking | **Paridhi Sharma** |
| Web Crawler              | **Alfaiz**         |
| Dataset Collection       | **Alfaiz**         |
| REST API                 | **Johan**          |
| PostgreSQL Integration   | **Johan**          |
| Docker & Deployment      | **Johan**          |
| Ranking Research         | **Lakshay Yadav**  |
| PageRank Research        | **Lakshay Yadav**  |

---

# Development Workflow

The project follows a modular development strategy.

```text
                    Architecture

                          │

        ┌─────────────────┼─────────────────┐

        ▼                 ▼                 ▼

 Indexing Team      Backend Team     Crawling Team

        │                 │                 │

        └───────────┬─────┴─────────────────┘

                    ▼

             Integration Testing

                    ▼

          Performance Benchmarking

                    ▼

              Production Release
```

Each subsystem is developed independently, validated through unit and integration testing, and then merged into the main project after review.

---

# Collaboration Philosophy

The development of TROVIX is guided by several collaborative engineering principles.

* Every subsystem has a clearly defined owner.
* Architectural decisions are documented before implementation.
* Code reviews ensure consistency across modules.
* Documentation evolves alongside implementation.
* Testing is considered part of development rather than a separate phase.
* Performance is continuously measured using reproducible benchmarks.

This workflow enables multiple contributors to work simultaneously while maintaining a unified architectural vision.

---

# Contributions

Although responsibilities are distributed across different modules, TROVIX is developed as a collaborative engineering effort.

Major architectural decisions, implementation discussions, testing strategies, and roadmap planning are shared among the team, with each contributor providing expertise within their respective domain.

The modular structure of the project ensures that every contribution strengthens the overall platform while remaining independently maintainable and extensible.

---
# Development Roadmap

TROVIX is being developed incrementally, with each phase introducing a new layer of functionality while preserving the modular architecture established in previous iterations.

Instead of implementing every feature simultaneously, development follows a structured roadmap that prioritizes correctness, maintainability, and extensibility.

Each milestone represents a production-ready checkpoint, ensuring that every subsystem is validated before new capabilities are introduced.

---

# Phase I — Core Information Retrieval Engine

**Status:** ✅ Completed (Architecture & Design)

The first phase establishes the foundation of the search engine.

Major deliverables include:

* Document Parser
* Tokenization Pipeline
* Vocabulary Construction
* Posting List Generation
* Custom Inverted Index
* Index Builder
* BM25 Ranking
* Query Execution Pipeline
* Technical Documentation
* Testing Strategy
* Performance Benchmark Framework

At the completion of this phase, TROVIX possesses a fully documented lexical search architecture ready for implementation.

---

# Phase II — Core Implementation

**Status:** 🚧 In Progress

The objective of this phase is to translate the architectural specifications into production-quality code.

Primary deliverables include:

### Indexing Engine

* Document Parser implementation
* Tokenization Pipeline
* Vocabulary generation
* Posting List construction
* Inverted Index implementation
* Index serialization

---

### Query Engine

* Query parser
* Candidate retrieval
* BM25 implementation
* Result ranking
* Search response generation

---

### Testing

* Unit testing
* Integration testing
* End-to-end testing
* Regression testing

---

### Benchmarking

* Index construction benchmarks
* Query latency benchmarks
* Memory profiling
* Scalability testing

Expected outcome

```text
Raw Documents

↓

Index Construction

↓

Search Queries

↓

Ranked Results
```

---

# Phase III — Infrastructure & Integration

Once the core retrieval engine is stable, the focus shifts toward exposing TROVIX as a usable search platform.

Major deliverables

## Backend Services

* REST API
* Request validation
* Response serialization
* API documentation

---

## Database

* PostgreSQL integration
* Metadata storage
* Configuration management

---

## Crawling

* Website crawling
* Content extraction
* Dataset generation
* Duplicate detection
* Crawl scheduling

---

## Deployment

* Docker support
* Environment configuration
* Production deployment
* Logging

At the end of this phase, TROVIX becomes a complete searchable platform rather than only a retrieval engine.

---

# Phase IV — Search Experience

This phase focuses on improving retrieval quality and user interaction.

Planned capabilities include

* Phrase Search
* Boolean Queries
* Fuzzy Matching
* Autocomplete
* Query Suggestions
* Search Highlighting
* Incremental Indexing
* Persistent Index Storage
* Query Caching
* Search Analytics

These features significantly improve usability while preserving the existing indexing architecture.

---

# Phase V — Intelligent Retrieval

The fifth phase introduces semantic understanding and artificial intelligence.

Future capabilities include

* Dense Embeddings
* Vector Search
* Hybrid Retrieval
* Neural Re-ranking
* Metadata Filtering
* Faceted Search
* Retrieval-Augmented Generation (RAG)

Instead of relying exclusively on lexical matching, TROVIX begins understanding the semantic meaning of documents and user queries.

---

# Phase VI — Distributed Search Platform

The final architectural milestone focuses on scalability.

Major objectives include

* Distributed indexing
* Sharding
* Replication
* Cluster management
* Load balancing
* Distributed caching
* Fault tolerance
* Cloud deployment

These capabilities transform TROVIX into a distributed search infrastructure capable of supporting enterprise-scale workloads.

---

# Milestone Overview

| Phase     | Objective                   | Status         |
| --------- | --------------------------- | -------------- |
| Phase I   | Architecture & Design       | ✅ Completed    |
| Phase II  | Core Engine Implementation  | 🚧 In Progress |
| Phase III | Backend & Infrastructure    | 📅 Planned     |
| Phase IV  | Advanced Search Features    | 📅 Planned     |
| Phase V   | AI & Semantic Retrieval     | 📅 Planned     |
| Phase VI  | Distributed Search Platform | 📅 Planned     |

---

# Project Timeline

```text
                    TROVIX ROADMAP

Architecture
██████████████████████████████████████████ 100%

Implementation
██████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 25%

Backend Infrastructure
░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 0%

Advanced Search
░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 0%

AI Retrieval
░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 0%

Distributed Platform
░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 0%
```

---

# Performance Goals

The current engineering targets for Version 1 are outlined below.

| Metric                 | Target       |
| ---------------------- | ------------ |
| Indexed Documents      | 50,000+      |
| Query Latency          | < 10 ms      |
| Vocabulary Lookup      | O(1) Average |
| Candidate Retrieval    | O(P)         |
| BM25 Ranking           | O(C × Q)     |
| Test Coverage          | 90%+         |
| Modular Components     | 100%         |
| Documentation Coverage | 100%         |

These targets serve as measurable engineering objectives throughout development.

---

# Getting Started

## Clone the Repository

```bash
git clone https://github.com/<organization-or-user>/trovix.git
cd trovix
```

---

## Create a Virtual Environment

```bash
python -m venv .venv
```

Activate the environment.

Linux / macOS

```bash
source .venv/bin/activate
```

Windows

```bash
.venv\Scripts\activate
```

---

## Install Dependencies

```bash
pip install -r requirements.txt
```

---

## Run the Test Suite

```bash
pytest
```

---

## Build the Index

```bash
python src/indexing/build_index.py
```

---

## Start the Search Engine

```bash
python src/query_engine/search.py
```

---

# Contributing

Contributions are welcome.

When contributing to TROVIX, contributors are encouraged to:

* Follow the established project architecture.
* Keep modules independent and loosely coupled.
* Add tests alongside new functionality.
* Update documentation for architectural changes.
* Benchmark significant performance optimizations.
* Maintain consistent coding standards.

Large architectural changes should be discussed before implementation to preserve the long-term vision of the project.

---

# Acknowledgements

TROVIX draws inspiration from decades of research in Information Retrieval and Search Systems.

The project has been influenced by the architectural principles and research behind:

* Apache Lucene
* Elasticsearch
* OpenSearch
* Meilisearch
* FAISS
* BM25 and the Okapi Retrieval Model
* *Introduction to Information Retrieval* by Manning, Raghavan, and Schütze

While inspired by these systems, TROVIX is implemented independently as an educational and engineering-focused project aimed at understanding and building modern search infrastructure from first principles.

---

<div align="center">

## Built with curiosity, engineering discipline, and a passion for Information Retrieval.

**TROVIX** • *From Documents to Discovery.*

</div>
# Why Build TROVIX?

Search engines are often treated as black boxes.

Developers interact with search APIs every day, yet relatively few have the opportunity to understand the algorithms and engineering decisions that power them.

TROVIX was created to bridge that gap.

Instead of abstracting away the complexity, the project embraces it.

Every architectural decision—from tokenization and inverted indexing to BM25 ranking and query execution—has been documented, justified, and implemented with the goal of understanding modern Information Retrieval systems from first principles.

This repository is therefore more than a software project.

It is a study of search engine architecture, systems engineering, and Information Retrieval.

---

# Engineering Principles

Throughout development, every major technical decision is evaluated against the following principles.

### Simplicity

Complexity should exist only when it provides measurable value.

Simple, well-designed systems are easier to understand, test, and maintain.

---

### Modularity

Each subsystem should perform one responsibility exceptionally well.

Modules communicate through interfaces rather than implementation details, allowing individual components to evolve independently.

---

### Performance

Efficiency is treated as a feature rather than an afterthought.

Algorithmic complexity, memory usage, and latency are considered throughout the design process.

---

### Documentation

Every subsystem should be documented before implementation.

Architecture should never exist only in source code.

Documentation is treated as a first-class engineering artifact.

---

### Correctness

Before optimizing performance, every component must produce deterministic and verifiable results.

Reliability always takes precedence over premature optimization.

---

### Continuous Improvement

The project is designed to evolve.

Every version introduces measurable improvements while preserving the architectural foundation established in previous releases.

---

# Repository Statistics

| Category                   | Value                 |
| -------------------------- | --------------------- |
| Documentation Modules      | **10**                |
| Planned Development Phases | **6**                 |
| Core Search Components     | **7**                 |
| Testing Documents          | **2**                 |
| Roadmap Documents          | **1**                 |
| Target Dataset Size        | **50,000+ Documents** |
| Target Query Latency       | **< 10 ms**           |
| Primary Ranking Algorithm  | **BM25**              |
| Architecture Style         | **Modular**           |

---

# Repository Goals

The long-term objective is to transform TROVIX into a complete Information Retrieval platform featuring:

* Classical Lexical Search
* Semantic Search
* Hybrid Retrieval
* AI-Assisted Question Answering
* Distributed Indexing
* High Availability
* Production-Scale Infrastructure

Each milestone builds upon the previous one, ensuring that the project remains educational, extensible, and grounded in sound engineering principles.

---

# License

This project is released under the **MIT License**.

You are free to use, modify, distribute, and build upon this work in accordance with the terms of the license.

See the `LICENSE` file for additional details.

---

# Acknowledgements

TROVIX draws inspiration from decades of research and engineering in Information Retrieval.

The project acknowledges the contributions of the open-source community and the researchers whose work has shaped modern search systems.

Special thanks to the communities and projects behind:

* Apache Lucene
* Elasticsearch
* OpenSearch
* Meilisearch
* FAISS
* BM25 and the Okapi Retrieval Model
* *Introduction to Information Retrieval* by Manning, Raghavan & Schütze
* *Managing Gigabytes* by Witten, Moffat & Bell

Their research and engineering excellence have significantly influenced the design philosophy of TROVIX.

---

<div align="center">

# TROVIX

### *Engineering Search. Understanding Retrieval. Building the Future.*

---

**Designed and Developed by**

### **Paridhi Sharma • Lakshay Yadav • Johan • Alfaiz**

---

*Version 1.0 • Information Retrieval Engine*

*"From Documents to Discovery."*

</div>
