# TROVIX V1 Module Interfaces

**Status**: FROZEN - DO NOT CHANGE WITHOUT TEAM DISCUSSION  
**Last Updated**: Phase 1  
**Owner**: Paridhi

---

## Overview

These are the 8 public interfaces for TROVIX V1. Everything else is implementation detail.

This document defines the contract between:
- **Alfaiz** (Crawler) → produces `list[Document]`
- **Paridhi** (Search Engine) → owns everything inside
- **Johan** (API) → calls `SearchEngine.search()`
- **Lakshay** (Evaluation) → consumes `SearchResult`

---

## Interface 1: Document

**Owner**: Paridhi  
**Used By**: Everyone  
**Responsibility**: Universal data model for all content in TROVIX

```python
from dataclasses import dataclass, field
from typing import Any

@dataclass
class Document:
    id: int
    title: str
    content: str
    url: str | None = None
    metadata: dict[str, Any] = field(default_factory=dict)
```

### Example

```python
Document(
    id=1,
    title="Machine Learning Basics",
    content="Machine learning is a subset of AI...",
    url="https://example.com/ml",
    metadata={
        "author": "Google",
        "language": "en",
        "crawled_at": "2024-01-15"
    }
)
```

### Rules

- Every document inside TROVIX becomes this object
- Parser creates it
- Crawler outputs it
- IndexBuilder receives it
- SearchResult contains it

---

## Interface 2: SearchResult

**Owner**: Paridhi  
**Used By**: Johan (API), Lakshay (Evaluation)  
**Responsibility**: Standardized ranking output

```python
from dataclasses import dataclass

@dataclass
class SearchResult:
    document: Document
    score: float
```

### Example

```python
SearchResult(
    document=Document(
        id=12,
        title="Machine Learning Basics",
        content="...",
        url="https://example.com/ml",
        metadata={}
    ),
    score=8.44
)
```

### Why Document Instead of document_id?

- Johan can immediately expose `title`, `url`, `snippet` without asking you
- No extra lookups needed
- Lakshay can evaluate ranking quality with full context
- Future UI changes are non-breaking

---

## Interface 3: DocumentParser

**Owner**: Paridhi  
**Used By**: Internal (IndexBuilder initialization)  
**Responsibility**: Convert file → Document

```python
from pathlib import Path

class DocumentParser:
    def parse(self, path: Path) -> Document:
        """
        Convert a file at path into a Document object.
        
        Args:
            path: Path to document file (txt, json, html, pdf, etc)
        
        Returns:
            Document object with id, title, content, url, metadata
        
        Raises:
            FileNotFoundError: If path does not exist
            ParsingError: If file cannot be parsed
        """
        ...
```

### What's Private

Do NOT expose these:
- `parse_txt()`
- `parse_json()`
- `parse_pdf()`
- `parse_html()`

These stay internal. You can add them anytime without breaking anyone.

### Example Usage

```python
parser = DocumentParser()
doc = parser.parse(Path("data/document.txt"))
```

---

## Interface 4: IndexBuilder

**Owner**: Paridhi  
**Used By**: Alfaiz (integration), internal initialization  
**Responsibility**: Convert list[Document] → inverted index

```python
class IndexBuilder:
    def index(self, documents: list[Document]) -> None:
        """
        Build the inverted index from documents.
        
        Internally handles:
        - Parsing (if needed)
        - Tokenization
        - Normalization
        - Vocabulary building
        - Posting list creation
        
        Args:
            documents: List of Document objects to index
        
        Returns:
            None (modifies internal state)
        """
        ...
```

### What's Hidden

Nobody except you touches:
- Tokenizer
- Vocabulary
- Posting Lists
- Stemmer
- Stopwords
- Candidate Retrieval

### Example Usage

```python
builder = IndexBuilder()
crawler_output = [doc1, doc2, doc3, ...]
builder.index(crawler_output)
```

---

## Interface 5: SearchEngine

**Owner**: Paridhi  
**Used By**: Johan (API), Lakshay (Evaluation)  
**Responsibility**: THE public interface for searching

```python
class SearchEngine:
    def search(
        self, 
        query: str, 
        top_k: int = 10
    ) -> list[SearchResult]:
        """
        Execute a search query and return top-ranked documents.
        
        Internally handles:
        - Query tokenization
        - Candidate document retrieval
        - BM25 ranking
        - Sorting and truncation
        
        Args:
            query: Search query string
            top_k: Number of top results to return (default: 10)
        
        Returns:
            List of SearchResult objects, sorted by score descending
        
        Raises:
            ValueError: If query is empty
            SearchEngineError: If index not initialized
        """
        ...

    def get_statistics(self) -> IndexStatistics:
        """
        Return index statistics.
        
        Returns:
            IndexStatistics object
        """
        ...

    def get_index_info(self) -> IndexInfo:
        """
        Return index metadata.
        
        Returns:
            IndexInfo object
        """
        ...
```

### Example Usage

```python
engine = SearchEngine()
results = engine.search("machine learning", top_k=5)

for result in results:
    print(f"{result.document.title}: {result.score}")
```

### What's Hidden

Nobody calls these directly:
- `tokenize()`
- `rank_with_bm25()`
- `get_candidate_docs()`
- `normalize_text()`

You decide the implementation.

---

## Interface 6: IndexStatistics

**Owner**: Paridhi  
**Used By**: Johan (API)  
**Responsibility**: Index metrics

```python
from dataclasses import dataclass

@dataclass
class IndexStatistics:
    document_count: int
    vocabulary_size: int
    average_document_length: float
    posting_count: int
```

### Example

```python
IndexStatistics(
    document_count=52342,
    vocabulary_size=183211,
    average_document_length=312.5,
    posting_count=183211
)
```

### Johan Uses This For

```
GET /stats

{
  "documents": 52342,
  "vocabulary": 183211,
  "posting_lists": 183211,
  "average_document_length": 312.5
}
```

---

## Interface 7: IndexInfo

**Owner**: Paridhi  
**Used By**: Johan (API)  
**Responsibility**: Index metadata

```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class IndexInfo:
    indexed: bool
    build_time_seconds: float
    created_at: datetime
    version: str
```

### Example

```python
IndexInfo(
    indexed=True,
    build_time_seconds=32.1,
    created_at=datetime(2024, 1, 15, 14, 30, 0),
    version="1.0.0"
)
```

### Johan Uses This For

```
GET /index/status

{
  "indexed": true,
  "documents": 52342,
  "build_time": "32.1s",
  "created_at": "2024-01-15T14:30:00"
}
```

---

## Interface 8: Crawler Output

**Owner**: Alfaiz  
**Used By**: Paridhi  
**Responsibility**: Produce searchable documents

**Contract**: You MUST output

```python
list[Document]
```

Nothing else.

### Example

```python
documents = [
    Document(
        id=1,
        title="Python Tutorial",
        content="Python is a programming language...",
        url="https://example.com/python",
        metadata={"source": "web", "crawled_at": "2024-01-15"}
    ),
    Document(
        id=2,
        title="Web Scraping Guide",
        content="Web scraping is a technique...",
        url="https://example.com/scraping",
        metadata={"source": "web", "crawled_at": "2024-01-15"}
    ),
]

return documents
```

### What You DON'T Do

❌ Don't tokenize  
❌ Don't build vocabulary  
❌ Don't create posting lists  
❌ Don't rank  
❌ Don't parse

Just produce `Document` objects.

---

## Dependency Graph

```
         Alfaiz
        Crawler
           │
           ▼
       Document[]
           │
           ▼
      IndexBuilder
      (Paridhi's Internal)
           │
           ▼
      SearchEngine
      (Paridhi owns)
           │
           ▼
       SearchResult
           │
      ┌────┼────┐
      ▼    ▼    ▼
    Johan Lakshay UI
```

---

## Folder Structure (Reflects Public vs Private)

```
src/
│
├── models/
│   ├── document.py              (PUBLIC)
│   ├── search_result.py         (PUBLIC)
│   ├── statistics.py            (PUBLIC)
│   └── index_info.py            (PUBLIC)
│
├── parser/
│   └── document_parser.py       (PUBLIC)
│
├── indexing/
│   └── index_builder.py         (PUBLIC)
│
├── query/
│   └── search_engine.py         (PUBLIC)
│
└── internal/
    ├── tokenizer.py            (PRIVATE - do not import outside)
    ├── postings.py             (PRIVATE)
    ├── vocabulary.py           (PRIVATE)
    ├── bm25.py                 (PRIVATE)
    ├── candidate_retrieval.py  (PRIVATE)
    ├── stemmer.py              (PRIVATE)
    └── stopwords.py            (PRIVATE)
```

---

## Import Rules

### Alfaiz (Crawler)

```python
from models.document import Document

# That's it.
```

### Johan (API)

```python
from query.search_engine import SearchEngine
from models.search_result import SearchResult
from models.statistics import IndexStatistics
from models.index_info import IndexInfo

# Nothing from internal/
```

### Lakshay (Evaluation)

```python
from query.search_engine import SearchEngine
from models.search_result import SearchResult

# That's it.
```

### Paridhi (You)

```python
from models.document import Document
from models.search_result import SearchResult
from parser.document_parser import DocumentParser
from indexing.index_builder import IndexBuilder
from query.search_engine import SearchEngine

# You can import from internal/
from internal.tokenizer import Tokenizer
from internal.postings import Posting, Vocabulary
from internal.bm25 import BM25Ranker
```

---

## Change Policy

### ✅ You CAN change without breaking anyone

- Add private methods to Tokenizer
- Change BM25 implementation
- Add internal helper classes
- Change internal data structures
- Optimize candidate retrieval
- Modify posting list representation

**Why?** Your internal changes don't affect the public interfaces.

### ❌ You CANNOT change these without team discussion

- `Document` fields
- `SearchResult` structure
- `SearchEngine.search()` signature
- `IndexBuilder.index()` signature
- `DocumentParser.parse()` signature

**Why?** Everyone else's code depends on these.

### Example Scenario

**Month 1**: You use BM25

```python
class SearchEngine:
    def search(self, query, top_k):
        tokens = self.tokenizer.tokenize(query)
        candidates = self.bm25.get_candidates(tokens)
        results = self.bm25.rank(candidates)
        return results[:top_k]
```

**Month 3**: You want to switch to Neural Ranking

```python
class SearchEngine:
    def search(self, query, top_k):
        tokens = self.tokenizer.tokenize(query)
        candidates = self.neural_retriever.get_candidates(tokens)
        results = self.neural_ranker.rank(candidates)
        return results[:top_k]
```

**Result**: Johan's API doesn't change. Alfaiz's crawler doesn't change. Lakshay's evaluation code doesn't change.

Only your internal code changed.

---

## Summary: What Each Person Depends On

| Person | Input | Output | Depends On |
|--------|-------|--------|-----------|
| **Alfaiz** | Raw web data | `list[Document]` | Nothing (he's first) |
| **Paridhi** | `list[Document]` | `SearchResult` via `SearchEngine` | Only `Document` (public) |
| **Johan** | User query | JSON response | Only `SearchEngine` (public) |
| **Lakshay** | Query + Results | Evaluation metrics | Only `SearchResult` (public) |

---

## Frozen ✓

These 8 interfaces are locked for V1. Do not change them without calling a team sync.

1. ✓ Document
2. ✓ SearchResult
3. ✓ DocumentParser
4. ✓ IndexBuilder
5. ✓ SearchEngine
6. ✓ IndexStatistics
7. ✓ IndexInfo
8. ✓ Crawler Output (list[Document])

Everything else is yours to optimize, rewrite, and improve.

---

## Questions?

If anyone asks:

- "Can I call BM25 directly?" → No, go through `SearchEngine.search()`
- "Can I import from internal/?" → Only if you're Paridhi
- "Can I change Document fields?" → Not without team sync
- "Can I add helper methods to Tokenizer?" → Yes, it's private

---

**Created**: Phase 1  
**Next Step**: Create implementation files for each interface in this structure