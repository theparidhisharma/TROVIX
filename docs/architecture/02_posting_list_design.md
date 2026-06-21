# 02. Posting List Design

**Project:** TROVIX  
**Module:** Indexing Engine  
**Version:** 1.0  
**Author:** Pari (Indexing Lead)

---

# Table of Contents

1. Introduction
2. Problem Statement
3. Why Posting Lists?
4. Design Goals
5. Anatomy of an Inverted Index
6. Posting Structure
7. Posting List Structure
8. Vocabulary Design
9. Inverted Index Organization
10. Storage Layout
11. Index Construction
12. Lookup Process
13. Updating Posting Lists
14. Design Decisions
15. Trade-offs
16. Complexity Analysis
17. Future Improvements
18. Conclusion

---

# Introduction

The Posting List is the most fundamental data structure in TROVIX.

Every search query, regardless of its complexity, ultimately retrieves one or more posting lists before ranking documents.

Without posting lists, an inverted index cannot function.

This document specifies the internal representation, organization, and lifecycle of posting lists within TROVIX.

The goal is to define a data structure that is:

- Memory efficient
- Easy to update
- Fast to search
- Compatible with BM25
- Extensible for future search features

This document serves as the implementation specification for the indexing subsystem.

---

# Problem Statement

Suppose the search engine has indexed the following documents.

```
Document 1

Machine Learning Basics

----------------------------

Document 2

Python for Data Science

----------------------------

Document 3

Machine Vision Systems
```

Now suppose a user searches

```
machine
```

How does the search engine know which documents contain that word?

Scanning every document every time is inefficient.

Instead,

the search engine stores a direct mapping between

```
Term

↓

Documents
```

This mapping is called a Posting List.

The challenge is designing this structure so that

- retrieval is fast,
- memory usage remains reasonable,
- and ranking algorithms receive all required statistics.

---

# Why Posting Lists?

The inverted index consists of two primary components.

```
Vocabulary

↓

Posting Lists
```

The vocabulary tells us

> "Does this word exist?"

The posting list tells us

> "Where does this word exist?"

Without posting lists,

the inverted index would merely be a dictionary of words.

It would provide no information about document locations.

For example,

```
machine
```

must eventually answer

```
machine

↓

Document 3

Document 8

Document 17

Document 25
```

Every search query in TROVIX ultimately begins with retrieving one or more posting lists.

Therefore,

posting list design has a direct impact on

- query latency,
- memory consumption,
- ranking performance,
- scalability.

A poor posting list design negatively affects every other subsystem.

---

# Design Goals

The posting list implementation should satisfy the following objectives.

## Fast Retrieval

Given a term,

the posting list should be retrievable in approximately constant time through the vocabulary.

---

## Compact Storage

Posting lists should avoid storing unnecessary information.

Duplicate document contents should never be stored inside the index.

Only metadata required for retrieval and ranking should be maintained.

---

## Efficient Updates

During indexing,

new postings should be inserted with minimal overhead.

Index construction should remain close to linear time.

---

## Ranking Compatibility

The posting list must expose all information required by BM25.

This includes

- Document ID
- Term Frequency

Future versions may include

- Positions
- Field Information
- Payloads

without redesigning the overall architecture.

---

## Extensibility

Future retrieval techniques such as

- Phrase Search
- Boolean Retrieval
- Proximity Search

should require extending the posting structure rather than replacing it.

This minimizes future refactoring and preserves compatibility with existing indexes.

---

# Anatomy of an Inverted Index

The inverted index is not a single data structure. Instead, it is composed of several smaller structures that work together to support efficient document retrieval.

At the highest level, the indexing system consists of three primary components:

1. Vocabulary
2. Posting Lists
3. Postings

These components form a hierarchy.

```
                 Inverted Index
                       │
                       ▼
                 Vocabulary
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
     machine       learning      python
         │             │             │
         ▼             ▼             ▼
   Posting List   Posting List  Posting List
         │             │             │
         ▼             ▼             ▼
     Posting       Posting      Posting
```

Every search query eventually navigates through this hierarchy.

---

# Vocabulary

The vocabulary is the entry point into the inverted index.

Its responsibility is to maintain every unique searchable term that exists in the corpus.

Conceptually,

```
Vocabulary

↓

machine

learning

python

database

docker

network
```

Each term acts as a key.

The corresponding value is a posting list.

Example

```python
{
    "machine": PostingList,
    "learning": PostingList,
    "python": PostingList
}
```

Notice that the vocabulary does **not** store documents.

It only stores references to posting lists.

This separation keeps the architecture modular.

---

# Posting Lists

Each vocabulary term owns exactly one posting list.

For example,

```
machine

↓

Posting List
```

The posting list contains every document that includes the word **machine**.

Example

```
machine

↓

Document 3

Document 7

Document 15

Document 28
```

Internally,

this becomes

```python
[
    Posting(...),
    Posting(...),
    Posting(...),
    Posting(...)
]
```

Every posting corresponds to one document.

---

# Postings

A posting is the smallest searchable unit inside the inverted index.

One posting represents

> One term appearing inside one document.

For example,

```
Document 15

Machine Learning improves search systems.
```

produces

```python
Posting(
    document_id=15,
    term_frequency=1
)
```

If

```
Machine Machine Machine
```

appears,

the posting becomes

```python
Posting(
    document_id=15,
    term_frequency=3
)
```

Notice that the document only appears **once** inside the posting list.

Only the frequency changes.

This significantly reduces storage.

---

# Relationship Between Components

Suppose the corpus contains

```
Document 1

Machine Learning

------------------------

Document 2

Python Machine

------------------------

Document 3

Machine Vision
```

The vocabulary becomes

```text
Vocabulary

machine
learning
python
vision
```

The posting lists become

```text
machine

↓

Doc1

Doc2

Doc3

-------------------

learning

↓

Doc1

-------------------

python

↓

Doc2

-------------------

vision

↓

Doc3
```

Finally,

every document inside each posting list stores metadata.

```
machine

↓

Doc1 (TF=1)

Doc2 (TF=1)

Doc3 (TF=1)
```

This organization allows TROVIX to answer the question

> "Which documents contain this term?"

without scanning the corpus.

---

# Why Separate Vocabulary and Posting Lists?

An alternative design would store all information inside one giant structure.

Example

```python
{
    "machine": {
        "documents":[...]
    }
}
```

Although possible,

this makes extending the index more difficult.

Instead,

TROVIX separates responsibilities.

The vocabulary is responsible for

- locating terms

The posting list is responsible for

- storing document occurrences

The posting is responsible for

- storing document-specific statistics

This separation follows the Single Responsibility Principle and simplifies maintenance.

---

# Example Walkthrough

Suppose the following document enters the indexing pipeline.

```
Document 17

Machine Learning with Python
```

After tokenization,

```
machine

learning

python
```

The index builder performs three updates.

Step 1

```
Vocabulary

↓

machine

↓

Posting List

↓

Add Posting
```

Step 2

```
Vocabulary

↓

learning

↓

Posting List

↓

Add Posting
```

Step 3

```
Vocabulary

↓

python

↓

Posting List

↓

Add Posting
```

The final index becomes

```python
{
    "machine": [
        Posting(17,1)
    ],

    "learning": [
        Posting(17,1)
    ],

    "python": [
        Posting(17,1)
    ]
}
```

As more documents are indexed, additional postings are appended to these posting lists.

The vocabulary itself grows only when a previously unseen term is encountered.

---

# Why This Design Scales

This hierarchical organization provides several advantages.

- Vocabulary lookup remains approximately O(1).
- Posting lists grow independently.
- Common terms do not affect uncommon terms.
- Documents are never duplicated.
- Ranking statistics remain localized within postings.
- Future metadata can be added without redesigning the entire index.

This modular structure forms the backbone of the TROVIX indexing engine and directly supports efficient query processing and BM25 ranking.

---

# Posting Design

The **Posting** is the smallest unit of information stored inside the inverted index.

A posting represents the occurrence of **one term in one document**.

For every document that contains a particular term, exactly one posting is created.

For example, consider the following document.

```
Document 42

Machine learning is transforming software engineering.
```

The term

```
machine
```

appears once.

Therefore, one posting is created.

```
Posting

↓

Document ID = 42

Term Frequency = 1
```

If another document contains the same term,

another posting is created.

```
machine

↓

Posting

↓

Document 51

Term Frequency = 3
```

The posting list simply stores these postings together.

---

# Responsibilities

A Posting is responsible for storing information required to rank a document.

It should **not**

- store the document text
- store document metadata
- store unrelated statistics

Its purpose is to represent one relationship:

```
Term

↓

Document
```

---

# Initial Posting Structure

The first implementation of TROVIX keeps postings intentionally lightweight.

```python
class Posting:
    document_id: int
    term_frequency: int
```

Example

```python
Posting(
    document_id=15,
    term_frequency=4
)
```

Meaning

- Document 15 contains the current term.
- The term appears four times.

Nothing more is required for BM25.

---

# Why Store Document IDs?

Instead of storing

```
Machine Learning Basics.pdf
```

the posting stores

```
15
```

Advantages

- Smaller memory footprint
- Faster comparisons
- Faster serialization
- Easier indexing

Document IDs are immutable.

Once assigned, they never change.

---

# Why Store Term Frequency?

Term Frequency (TF) measures how many times a word appears inside a document.

Example

```
Machine

Machine

Machine

Learning
```

Frequency Table

| Term | Frequency |
|------|-----------|
| machine | 3 |
| learning | 1 |

BM25 uses TF to estimate relevance.

A document containing the search term multiple times is often more relevant than one containing it only once.

Without TF, BM25 cannot be computed.

---

# Why Not Store the Entire Document?

Suppose every posting stored

```
Document

↓

Entire Text
```

The index would duplicate document contents for every unique term.

Example

```
Document

↓

Machine

Learning

Python

Database

Cloud
```

The same document would be repeated five times.

This is extremely wasteful.

Instead,

the posting stores only

```
Document ID
```

The document itself remains in the document store.

---

# Future Posting Structure

As TROVIX evolves,

additional information may be stored inside each posting.

Example

```python
class Posting:
    document_id: int
    term_frequency: int
    positions: list[int]
    field: str
    payload: dict
```

---

## Positions

```
Machine Learning is Machine Learning

↓

machine

↓

Position 1

Position 4
```

Positions enable

- Phrase Search
- Proximity Search
- Highlighting
- Snippet Generation

---

## Field Information

Search engines often rank terms differently depending on where they appear.

Example

```
Title

↓

Machine Learning

--------------------

Body

↓

Machine Learning
```

A match inside the title is usually more important.

Future postings may therefore include

```
Field = Title
```

or

```
Field = Body
```

---

## Payload

Payloads allow arbitrary metadata to be attached to postings.

Examples

- Confidence Score
- Language
- Category
- Custom Ranking Signals

Version 1 of TROVIX does not require payloads.

---

# Posting Lifecycle

Every posting follows the same lifecycle.

```text
Token

↓

Find Vocabulary Entry

↓

Locate Posting List

↓

Create Posting

↓

Append to Posting List

↓

Persist Index
```

Once created,

a posting should never reference another posting.

Each posting remains independent.

---

# Memory Layout

Conceptually,

a posting occupies

```
Posting

├── Document ID

└── Term Frequency
```

Future versions

```
Posting

├── Document ID

├── Term Frequency

├── Positions

├── Field

└── Payload
```

Keeping the initial structure compact improves cache locality and reduces memory consumption.

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| Integer Document IDs | Lower memory usage |
| Store Term Frequency | Required for BM25 |
| Do Not Store Text | Avoid duplication |
| Immutable Posting | Simpler implementation |
| Lightweight Structure | Faster traversal |

---

# Summary

A Posting is the smallest building block of the inverted index.

It represents the relationship between a single term and a single document while storing only the information required for ranking.

By keeping postings compact and immutable, TROVIX minimizes memory usage and enables efficient traversal during query processing.

Future versions can extend postings with positional information, field weights, and custom payloads without changing the overall architecture of the indexing system.

---

# Posting List Design

A **Posting List** is an ordered collection of postings associated with a single term.

While a posting represents one occurrence of a term in one document, the posting list represents **all occurrences of that term across the entire document collection.**

Conceptually,

```
Vocabulary

↓

machine

↓

Posting List

↓

Posting

Posting

Posting

Posting
```

Every searchable term inside TROVIX owns exactly one posting list.

---

# Why Do We Need Posting Lists?

Suppose our corpus contains the following documents.

```
Document 1

Machine Learning Basics

--------------------------

Document 2

Python for Machine Learning

--------------------------

Document 3

Computer Vision
```

A user searches

```
machine
```

Without posting lists,

the search engine must inspect every document individually.

```
Document 1

↓

Contains machine?

↓

Yes

-----------------

Document 2

↓

Contains machine?

↓

Yes

-----------------

Document 3

↓

Contains machine?

↓

No
```

This approach becomes prohibitively expensive as the corpus grows.

Instead,

the posting list already stores the answer.

```
machine

↓

Doc 1

Doc 2
```

The search engine immediately knows which documents should be considered.

---

# Structure of a Posting List

A posting list is simply a collection of Posting objects.

Example

```python
PostingList = [

    Posting(document_id=1, term_frequency=2),

    Posting(document_id=7, term_frequency=4),

    Posting(document_id=13, term_frequency=1)

]
```

Each element represents one document containing the term.

Notice that

- One document appears only once.
- The frequency captures repeated occurrences.

---

# Internal Representation

The first implementation of TROVIX represents posting lists as Python lists.

Example

```python
[
    Posting(...),
    Posting(...),
    Posting(...),
    Posting(...)
]
```

Reasons

- Simple implementation
- Fast iteration
- Efficient appends during indexing
- Excellent compatibility with BM25

Alternative data structures such as linked lists or balanced trees were considered but rejected due to increased complexity and limited practical benefit for the initial implementation.

---

# Ordering of Postings

Posting lists should always remain ordered by Document ID.

Example

Correct

```
Doc 1

Doc 4

Doc 8

Doc 13

Doc 21
```

Incorrect

```
Doc 13

Doc 2

Doc 18

Doc 4
```

Maintaining sorted posting lists provides several advantages.

- Faster intersection
- Easier merging
- Better compression
- Deterministic output

Many search algorithms assume posting lists are sorted.

---

# Why Sort by Document ID?

Sorting by document ID enables efficient operations during query processing.

Suppose a user searches

```
machine AND learning
```

Posting Lists

```
machine

↓

1

4

8

11

19
```

```
learning

↓

2

4

8

15

19
```

Instead of comparing every document against every other document,

both lists can be traversed simultaneously.

```
1   4   8   11   19
    ↑

2   4   8   15   19
    ↑
```

This allows Boolean retrieval to execute in linear time.

---

# Posting List Lifecycle

Each posting list follows the same lifecycle.

```
Term Created

↓

Posting List Created

↓

New Posting Added

↓

Additional Postings Added

↓

Used During Search

↓

Serialized to Disk
```

A posting list grows only when new documents containing the corresponding term are indexed.

---

# Example Construction

Suppose the following three documents are indexed.

```
Document 1

Machine Learning

------------------

Document 2

Machine Vision

------------------

Document 3

Deep Learning
```

The resulting posting lists become

```
machine

↓

Doc1

Doc2

---------------------

learning

↓

Doc1

Doc3

---------------------

vision

↓

Doc2

---------------------

deep

↓

Doc3
```

Internally,

```python
{
    "machine": [

        Posting(1,1),

        Posting(2,1)

    ],

    "learning": [

        Posting(1,1),

        Posting(3,1)

    ]
}
```

---

# Insertion Strategy

Whenever a token is encountered,

the Index Builder performs the following steps.

```
Locate Vocabulary Entry

↓

Retrieve Posting List

↓

Create Posting

↓

Append Posting
```

If the posting list does not exist,

it is created automatically.

```
database

↓

New Posting List

↓

Add Posting
```

---

# Traversal During Search

Suppose a user searches

```
machine
```

The search engine performs

```
Vocabulary Lookup

↓

Retrieve Posting List

↓

Iterate Through Postings

↓

Candidate Documents
```

The posting list itself does not perform ranking.

It simply supplies candidate documents.

Ranking is delegated to the BM25 module.

This separation keeps the architecture modular.

---

# Memory Considerations

Posting lists represent the largest component of the inverted index.

For frequently occurring terms,

posting lists may contain thousands of postings.

Example

```
software

↓

25,000 Postings
```

To keep memory usage manageable,

posting lists should

- Store only essential information
- Avoid duplicate entries
- Remain contiguous in memory
- Support future compression

---

# Future Optimizations

Several improvements can enhance posting list performance.

## Skip Pointers

Allow traversal to jump over sections of a posting list.

Benefits

- Faster Boolean retrieval
- Reduced comparisons

---

## Compression

Compress document IDs using techniques such as

- Delta Encoding
- Variable Byte Encoding
- Elias Gamma Coding

Benefits

- Smaller indexes
- Faster loading
- Lower memory usage

---

## Block Storage

Very large posting lists may be divided into blocks.

Benefits

- Better cache locality
- Faster disk access
- Parallel processing

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| Python List | Simple and efficient |
| Sorted by Document ID | Enables fast intersections |
| One Posting per Document | Eliminates duplication |
| Append During Indexing | Efficient construction |
| Separate Ranking Logic | Cleaner architecture |

---

# Summary

A Posting List is the core retrieval structure of TROVIX.

It groups all postings associated with a single vocabulary term and provides the search engine with an efficient way to identify candidate documents.

By maintaining sorted, lightweight posting lists, TROVIX supports efficient indexing, fast query processing, and seamless integration with ranking algorithms such as BM25.

Future enhancements—including compression, skip pointers, and block-based storage—can be introduced without changing the overall architecture of the indexing subsystem.

---

# Vocabulary Design

The Vocabulary serves as the entry point into the inverted index.

Every search query begins by consulting the vocabulary to determine whether a term exists within the indexed corpus.

If the vocabulary cannot locate a term, no further processing is required.

For this reason, the vocabulary is designed to provide extremely fast lookup while maintaining a compact representation of every unique term discovered during indexing.

---

# What is a Vocabulary?

A vocabulary is the collection of all unique searchable terms present in the corpus.

Consider the following three documents.

```
Document 1

Machine Learning with Python

-------------------------

Document 2

Machine Vision Systems

-------------------------

Document 3

Python for Data Science
```

After tokenization,

the combined set of unique terms becomes

```
machine

learning

python

vision

systems

data

science
```

These unique terms form the vocabulary.

Notice that

```
machine
```

appears in multiple documents,

yet it appears **only once** inside the vocabulary.

---

# Responsibilities

The vocabulary has only one responsibility:

> Map a search term to its corresponding posting list.

Conceptually,

```
Vocabulary

↓

machine

↓

Posting List
```

It does **not**

- store documents
- perform ranking
- compute BM25
- tokenize text

It simply connects words to posting lists.

---

# Internal Representation

For Version 1,

the vocabulary will be implemented using Python's built-in dictionary.

```python
vocabulary = {
    "machine": PostingList(...),
    "learning": PostingList(...),
    "python": PostingList(...)
}
```

Keys

```
str
```

Values

```
PostingList
```

This representation provides average-case constant-time lookup.

---

# Why Use a Dictionary?

The vocabulary is one of the most frequently accessed components in the search engine.

Every query term performs one lookup.

Example

```
machine learning python
```

performs

```
Vocabulary["machine"]

Vocabulary["learning"]

Vocabulary["python"]
```

Using Python dictionaries allows these operations to execute in approximately

```
O(1)
```

average time.

---

# Vocabulary Lifecycle

The vocabulary evolves during indexing.

Initially,

```
{}
```

After indexing the first document,

```
{
    "machine": PostingList(...),

    "learning": PostingList(...)
}
```

After additional documents,

```
{
    "machine": PostingList(...),

    "learning": PostingList(...),

    "python": PostingList(...),

    "database": PostingList(...)
}
```

The vocabulary only grows when a previously unseen term is encountered.

---

# Vocabulary Update Algorithm

Whenever the Index Builder encounters a token,

it follows the following procedure.

```
Token

↓

Exists in Vocabulary?

↓

Yes -----------------> Retrieve Posting List

↓

No

↓

Create Posting List

↓

Insert into Vocabulary

↓

Append Posting
```

This process ensures that every unique term owns exactly one posting list.

---

# Example Walkthrough

Suppose the first indexed document is

```
Machine Learning
```

Vocabulary before indexing

```python
{}
```

After tokenization

```
machine

learning
```

Vocabulary becomes

```python
{
    "machine": PostingList(),

    "learning": PostingList()
}
```

Now suppose the second document is

```
Machine Vision
```

The term

```
machine
```

already exists.

Therefore,

only

```
vision
```

is added.

Final vocabulary

```python
{
    "machine": PostingList(),

    "learning": PostingList(),

    "vision": PostingList()
}
```

---

# Memory Considerations

The vocabulary stores only

- unique terms
- references to posting lists

It does **not** duplicate postings or documents.

This minimizes memory usage.

Conceptually,

```
Vocabulary

↓

machine

↓

Reference

↓

Posting List
```

rather than

```
Vocabulary

↓

Entire Posting List

↓

Entire Documents
```

---

# Future Improvements

The initial implementation prioritizes simplicity.

Future versions may replace or augment the dictionary with more advanced data structures.

## Trie

Supports prefix search and autocomplete.

Example

```
mach

↓

machine

machine learning
```

---

## Finite State Transducer (FST)

Useful for

- autocomplete
- compressed dictionaries
- efficient prefix matching

---

## Persistent Vocabulary

Instead of rebuilding the vocabulary after every restart,

future versions may serialize it to disk.

Benefits

- Faster startup
- Persistent indexes
- Reduced initialization time

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| Dictionary-Based Vocabulary | Constant-time lookup |
| One Entry Per Unique Term | Avoid duplication |
| References to Posting Lists | Lower memory usage |
| Shared Across Entire Index | Single source of truth |
| Independent of Ranking | Separation of concerns |

---

# Summary

The vocabulary acts as the gateway to the inverted index.

It maintains every unique searchable term while providing fast access to the corresponding posting lists.

By using a dictionary-based implementation, TROVIX achieves efficient lookup performance while keeping the architecture modular and extensible.

Future versions may introduce Trie-based structures or compressed dictionaries to support advanced search capabilities such as autocomplete and prefix matching.

---

# Inverted Index Design

The **Inverted Index** is the primary data structure of TROVIX.

While the vocabulary provides access to searchable terms and posting lists store document occurrences, the Inverted Index acts as the central object responsible for managing the entire indexing subsystem.

It combines the vocabulary, document statistics, corpus statistics, and indexing metadata into a single searchable structure.

Every indexing operation updates the Inverted Index, and every search query reads from it.

---

# Responsibilities

The Inverted Index is responsible for:

- Maintaining the vocabulary
- Storing posting lists
- Tracking document statistics
- Tracking corpus statistics
- Supporting efficient lookup
- Providing data required by BM25

The Inverted Index is **not** responsible for:

- Parsing documents
- Tokenizing text
- Ranking documents
- Executing search queries

Those responsibilities belong to other modules.

---

# High-Level Structure

Conceptually, the Inverted Index contains four major components.

```
                 Inverted Index
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
 Vocabulary      Document Stats   Corpus Stats
       │
       ▼
 Posting Lists
       │
       ▼
 Postings
```

Each component serves a specific purpose.

---

# Internal Representation

The first version of TROVIX will represent the inverted index using a Python class.

```python
class InvertedIndex:

    vocabulary: Dict[str, PostingList]

    document_lengths: Dict[int, int]

    total_documents: int

    average_document_length: float
```

Each field stores information required during indexing and retrieval.

---

# Vocabulary

Stores every unique searchable term.

Example

```python
{
    "machine": PostingList(...),
    "learning": PostingList(...),
    "python": PostingList(...)
}
```

Purpose

- Fast lookup
- Posting list retrieval

---

# Document Statistics

Certain ranking algorithms require document-specific information.

The most important statistic is document length.

Example

```python
document_lengths = {

    1: 243,

    2: 517,

    3: 102

}
```

The key represents

```
Document ID
```

The value represents

```
Number of Tokens
```

BM25 uses document length to normalize scores.

---

# Corpus Statistics

The index also stores statistics describing the entire document collection.

Example

```python
total_documents = 50000

average_document_length = 287.3
```

These values are updated whenever indexing completes.

They are required for BM25 calculations.

---

# Relationship Between Components

Suppose the term

```
machine
```

appears inside

```
Document 1

Document 5

Document 17
```

The Inverted Index stores

```text
Inverted Index

│

├── Vocabulary

│      │

│      ▼

│   machine

│      │

│      ▼

│   Posting List

│      │

│      ├── Posting(1,3)

│      ├── Posting(5,1)

│      └── Posting(17,4)

│

├── Document Lengths

│      ├── 1 → 248

│      ├── 5 → 173

│      └── 17 → 412

│

└── Corpus Statistics

       ├── Total Documents

       └── Average Length
```

Everything required for retrieval is accessible through this single object.

---

# Index Construction Workflow

Every indexed document follows the same process.

```text
Document

↓

Tokenizer

↓

Unique Tokens

↓

Vocabulary Lookup

↓

Posting List

↓

Posting

↓

Update Statistics

↓

Index Complete
```

After the workflow finishes,

the inverted index is immediately ready to answer search queries.

---

# Lookup Workflow

Suppose the user searches

```
machine learning
```

The search engine performs the following operations.

Step 1

```
Lookup

↓

machine

↓

Posting List
```

Step 2

```
Lookup

↓

learning

↓

Posting List
```

Step 3

```
Merge Candidate Documents
```

Step 4

```
Compute BM25
```

Step 5

```
Return Ranked Results
```

Notice that

the search engine never scans documents directly.

Everything begins with the Inverted Index.

---

# Why Centralize Everything?

One possible design would store

- vocabulary
- document statistics
- corpus statistics

inside independent global objects.

Instead,

TROVIX groups these structures inside a single InvertedIndex class.

Advantages

- Easier serialization
- Cleaner API
- Better encapsulation
- Simpler testing
- Future persistence support

Instead of passing multiple objects between modules,

only one object needs to be shared.

---

# Public Operations

The Inverted Index should expose only a small set of operations.

| Method | Purpose |
|---------|---------|
| `add_document()` | Index a new document |
| `get_posting_list(term)` | Retrieve posting list |
| `contains(term)` | Check if a term exists |
| `document_length(doc_id)` | Retrieve document length |
| `vocabulary_size()` | Number of indexed terms |
| `total_documents()` | Number of indexed documents |

Keeping the public interface small reduces coupling between modules.

---

# Future Extensions

As TROVIX evolves,

the Inverted Index may additionally maintain

- Positional indexes
- Field indexes
- Compressed posting lists
- Persistent storage
- Cache layer
- Incremental updates
- Sharding metadata
- Query statistics

The architecture presented here allows these capabilities to be integrated without redesigning the existing indexing subsystem.

---

# Design Decisions

| Decision | Reason |
|----------|--------|
| Single InvertedIndex object | Centralized state management |
| Separate Vocabulary | Faster lookup and cleaner organization |
| Separate Document Statistics | Required by BM25 |
| Separate Corpus Statistics | Supports ranking algorithms |
| Small Public API | Easier maintenance and testing |

---

# Summary

The Inverted Index is the central data structure of TROVIX.

Rather than storing only posting lists, it encapsulates every structure required for efficient indexing and retrieval, including the vocabulary, document statistics, and corpus-level statistics.

By organizing these components into a single cohesive object, TROVIX achieves a modular architecture that is easier to maintain, extend, and optimize as the search engine evolves.

---

# Index Construction Algorithm

Once the internal data structures have been defined, the next step is to describe how they are populated.

The Index Construction Algorithm is responsible for transforming a collection of raw documents into a fully functional inverted index.

Unlike query processing, index construction is an offline operation. It is typically executed once when documents are added to the corpus, after which the generated index is used to serve search requests.

The algorithm follows a deterministic sequence of operations, ensuring that every document is processed consistently.

---

# Overall Workflow

The complete indexing workflow is illustrated below.

```mermaid
flowchart LR

A[Raw Document]
--> B[Document Parser]
--> C[Tokenizer]
--> D[Count Term Frequencies]
--> E[Update Vocabulary]
--> F[Update Posting Lists]
--> G[Update Statistics]
--> H[Index Complete]
```

Each stage performs one clearly defined responsibility.

---

# Step 1 — Parse the Document

The parser reads a raw document and converts it into a structured `Document` object.

Input

```
Machine Learning with Python
```

Output

```python
Document(
    id=17,
    title="...",
    body="Machine Learning with Python"
)
```

No indexing occurs during this stage.

Its only purpose is to prepare structured input for later stages.

---

# Step 2 — Tokenize the Document

The tokenizer converts raw text into normalized search terms.

Pipeline

```
Lowercase

↓

Remove punctuation

↓

Split into tokens

↓

Remove stopwords

↓

Stem
```

Example

Input

```
Machine Learning with Python
```

Output

```
machine

learn

python
```

These normalized tokens become the input for index construction.

---

# Step 3 — Count Term Frequencies

Before updating the inverted index, the algorithm determines how many times each term appears within the document.

Example

```
machine

machine

learning

python

machine
```

Frequency table

| Term | Frequency |
|------|-----------|
| machine | 3 |
| learning | 1 |
| python | 1 |

This prevents duplicate postings from being created for the same document.

Each document contributes **exactly one posting per unique term**.

---

# Step 4 — Update the Vocabulary

For every unique token,

the algorithm checks whether the vocabulary already contains the term.

Workflow

```
Token

↓

Exists?

↓

Yes -----------> Retrieve Posting List

↓

No

↓

Create Posting List

↓

Insert into Vocabulary
```

After this step,

every term is guaranteed to have a corresponding posting list.

---

# Step 5 — Create the Posting

The algorithm creates one posting for each unique term.

Example

```python
Posting(
    document_id=17,
    term_frequency=3
)
```

Notice that

```
machine
```

appears three times,

yet only one posting is created.

The frequency captures repeated occurrences.

---

# Step 6 — Append to the Posting List

The newly created posting is appended to the posting list belonging to that term.

Before

```python
machine

↓

[
    Posting(1,2),

    Posting(8,1)
]
```

After

```python
machine

↓

[
    Posting(1,2),

    Posting(8,1),

    Posting(17,3)
]
```

The posting list now reflects the updated corpus.

---

# Step 7 — Update Document Statistics

After indexing completes,

document-level statistics are recorded.

Example

```python
document_lengths[17] = 241
```

These statistics are later required by BM25.

---

# Step 8 — Update Corpus Statistics

Finally,

global statistics are updated.

Example

```python
total_documents += 1

average_document_length = ...
```

These values are maintained separately from the posting lists because they describe the corpus as a whole rather than individual documents.

---

# Complete Algorithm

The following pseudocode summarizes the indexing process.

```text
for each document

    parse document

    tokenize text

    count term frequencies

    for each unique term

        if term not in vocabulary

            create posting list

        create posting

        append posting

    update document statistics

update corpus statistics
```

The algorithm guarantees that:

- Every unique term has one posting list.
- Every document contributes at most one posting per term.
- Corpus statistics remain synchronized with the index.

---

# Why This Algorithm?

Several alternative approaches were considered.

For example,

creating a posting every time a token appears.

```
machine

↓

Posting

↓

Posting

↓

Posting
```

This would create multiple postings for the same document and waste memory.

Instead,

term frequencies are computed first,

allowing the algorithm to create only one posting per document.

This significantly reduces index size while preserving all information required for ranking.

---

# Complexity

For a document containing **t** tokens,

- Tokenization: **O(t)**
- Frequency Counting: **O(t)**
- Vocabulary Updates: **O(u)**

where **u** is the number of unique terms.

Since **u ≤ t**,

overall complexity remains linear.

```
O(t)
```

for each document.

Across the entire corpus,

```
O(T)
```

where **T** is the total number of tokens.

---

# Summary

The Index Construction Algorithm transforms raw documents into a structured inverted index through a sequence of deterministic operations.

By separating parsing, tokenization, frequency counting, vocabulary updates, and statistics collection, the algorithm remains modular, efficient, and easy to maintain.

The resulting index becomes immediately available for query processing and BM25 ranking without requiring additional preprocessing.

---

# Lookup Algorithm

Once the inverted index has been constructed, its primary responsibility is to answer search queries efficiently.

Unlike the indexing process, which is performed offline, the lookup algorithm executes every time a user performs a search.

Its objective is simple:

> Given one or more search terms, retrieve the relevant posting lists with minimal computation.

Since the vocabulary provides constant-time lookup on average, query execution begins by consulting the vocabulary rather than scanning documents.

---

# Query Execution Workflow

Every search query follows the same sequence of operations.

```mermaid
flowchart LR

A[User Query]
--> B[Tokenizer]
--> C[Vocabulary Lookup]
--> D[Retrieve Posting Lists]
--> E[Generate Candidate Documents]
--> F[BM25 Ranking]
--> G[Sort Results]
--> H[Return Results]
```

Notice that the search engine never opens every document.

Instead, it immediately retrieves only the posting lists corresponding to the query terms.

---

# Step 1 — Receive the Query

Suppose the user enters

```
machine learning
```

At this stage the query is simply raw text.

No searching has yet occurred.

---

# Step 2 — Normalize the Query

The same tokenizer used during indexing must also process search queries.

Pipeline

```
Lowercase

↓

Remove punctuation

↓

Split words

↓

Remove stopwords

↓

Stem
```

Input

```
Machine Learning
```

Output

```
machine

learn
```

Using the identical preprocessing pipeline guarantees consistency between indexed documents and search queries.

---

# Step 3 — Vocabulary Lookup

Each normalized term is searched inside the vocabulary.

Example

```python
Vocabulary["machine"]

Vocabulary["learn"]
```

Possible outcomes

### Term Exists

```
machine

↓

Posting List
```

Continue processing.

---

### Term Does Not Exist

```
quantumteleportation

↓

Not Found
```

Return an empty posting list and continue processing the remaining terms.

The search engine should never throw an error simply because a term is unknown.

---

# Step 4 — Retrieve Posting Lists

Every successful lookup returns one posting list.

Example

```
machine

↓

Posting List

↓

Doc1

Doc7

Doc18

Doc25
```

```
learn

↓

Posting List

↓

Doc1

Doc11

Doc18
```

These posting lists become the candidate set for ranking.

---

# Step 5 — Candidate Generation

For multi-term queries,

candidate documents are generated from the retrieved posting lists.

Example

Query

```
machine learning
```

Posting Lists

```
machine

↓

1

7

18

25
```

```
learning

↓

1

11

18
```

Combined candidate documents

```
1

7

11

18

25
```

The ranking module will later determine which of these documents are most relevant.

---

# Step 6 — Ranking

The lookup algorithm does not compute document relevance.

Instead,

it passes the candidate documents and query terms to the BM25 ranking engine.

Example

```text
Candidates

↓

BM25

↓

Scores

↓

Sorted Results
```

Separating retrieval from ranking keeps the architecture modular and allows different ranking algorithms to be introduced in the future.

---

# Lookup Pseudocode

```text
Receive Query

↓

Normalize Query

↓

For Each Term

    Lookup Vocabulary

    Retrieve Posting List

Merge Candidate Documents

Pass Candidates to BM25

Sort Results

Return Results
```

---

# Lookup Example

Assume the following vocabulary.

```python
{
    "machine": [

        Posting(1,2),

        Posting(4,1),

        Posting(8,3)

    ],

    "python": [

        Posting(2,1),

        Posting(8,2)

    ]
}
```

Query

```
machine python
```

Retrieved posting lists

```
machine

↓

1

4

8
```

```
python

↓

2

8
```

Candidate documents

```
1

2

4

8
```

BM25 will then calculate a relevance score for each candidate before returning the final ranked results.

---

# Performance Considerations

Efficient lookup depends on several factors.

- Constant-time vocabulary lookup.
- Efficient traversal of posting lists.
- Compact memory layout.
- Minimal object allocation during queries.

The lookup algorithm intentionally avoids unnecessary work.

Documents that do not contain any query terms are never examined.

---

# Complexity

For a query containing **q** terms,

Vocabulary Lookup

```
O(q)
```

Posting List Retrieval

```
O(k)
```

where

```
k
```

represents the total size of the retrieved posting lists.

Overall,

the lookup process scales with the number of matching documents rather than the total number of indexed documents.

This is the primary performance advantage of an inverted index.

---

# Summary

The Lookup Algorithm transforms a user query into a set of candidate documents through efficient vocabulary lookups and posting list retrieval.

By leveraging the inverted index, TROVIX avoids scanning the entire corpus and retrieves only the documents relevant to the query.

This separation of retrieval and ranking provides a clean, modular architecture capable of supporting increasingly sophisticated ranking strategies in future versions.

---

# Complexity Analysis

The efficiency of the posting list design directly determines the performance of the search engine.

This section analyzes the computational complexity of the major operations performed on the inverted index.

Throughout this analysis, the following notation is used.

| Symbol | Description |
|----------|-------------|
| N | Total number of documents |
| T | Total number of indexed tokens |
| V | Vocabulary size |
| U | Unique terms in a document |
| P | Size of a posting list |
| Q | Number of query terms |

---

# Vocabulary Operations

## Insert New Term

During indexing, every unique token is inserted into the vocabulary if it does not already exist.

Implementation

```python
vocabulary[token] = PostingList()
```

Average Complexity

```
O(1)
```

Worst Case

```
O(V)
```

Although hash collisions are theoretically possible, Python dictionaries provide near constant-time performance in practice.

---

## Lookup Term

Searching begins by locating a term inside the vocabulary.

Example

```python
posting_list = vocabulary["machine"]
```

Average Complexity

```
O(1)
```

This is one of the primary reasons for using a dictionary-based vocabulary.

---

# Posting List Operations

## Append Posting

Whenever a new document containing a term is indexed, a posting is appended.

```python
posting_list.append(posting)
```

Complexity

```
O(1)
```

Amortized.

---

## Traverse Posting List

Suppose

```
machine
```

appears in

```
8,000 documents.
```

The posting list must be traversed once.

Complexity

```
O(P)
```

where

```
P = Posting List Length
```

---

## Boolean Intersection

Suppose

```
machine

AND

learning
```

Posting Lists

```
Machine

↓

1

4

8

13

21
```

```
Learning

↓

2

4

8

17

21
```

Both sorted lists can be traversed simultaneously.

Complexity

```
O(P1 + P2)
```

instead of

```
O(P1 × P2)
```

This is one of the biggest advantages of maintaining sorted posting lists.

---

# Index Construction

For one document,

the algorithm performs

- Tokenization
- Frequency Counting
- Vocabulary Updates
- Posting Creation

Overall

```
O(t)
```

where

```
t
```

represents the number of tokens inside the document.

Across the complete corpus,

```
O(T)
```

---

# Query Processing

Suppose a query contains

```
Q
```

terms.

Operations include

- Tokenization
- Vocabulary Lookup
- Posting Retrieval
- Candidate Generation

Overall complexity

```
O(Q + P)
```

where

```
P
```

represents the total size of all retrieved posting lists.

Notice that

```
P << N
```

for most queries.

---

# Space Complexity

The inverted index stores

- Vocabulary
- Posting Lists
- Document Statistics
- Corpus Statistics

Overall space complexity

```
O(T)
```

because every indexed token contributes to exactly one posting.

The vocabulary itself grows much more slowly than the total number of tokens.

---

# Complexity Summary

| Operation | Average Complexity |
|-----------|-------------------|
| Vocabulary Lookup | O(1) |
| Vocabulary Insert | O(1) |
| Posting Append | O(1) |
| Posting Traversal | O(P) |
| Boolean Intersection | O(P₁ + P₂) |
| Index Construction | O(T) |
| Query Processing | O(Q + P) |

---

# Performance Expectations

The initial implementation of TROVIX is designed around the following performance targets.

| Metric | Target |
|---------|--------|
| Indexed Documents | 50,000+ |
| Vocabulary Size | 100,000+ |
| Posting Append | O(1) |
| Vocabulary Lookup | O(1) |
| Query Latency | <10 ms |
| Index Construction | Linear |

These targets will be validated later during benchmarking.

---

# Future Improvements

Although the current posting list design satisfies the requirements of Version 1, several improvements can further enhance efficiency and scalability.

## Positional Posting Lists

Store token positions.

Example

```python
Posting(

    document_id=17,

    term_frequency=4,

    positions=[8,19,33,61]

)
```

Benefits

- Phrase Search
- Proximity Search
- Better snippets

---

## Skip Pointers

Large posting lists become expensive to traverse.

Skip pointers allow the search engine to jump across sections of a posting list.

Benefits

- Faster Boolean Retrieval
- Lower traversal cost

---

## Posting Compression

Compress document IDs before storage.

Potential techniques

- Delta Encoding
- Variable Byte Encoding
- Elias Gamma Coding

Benefits

- Smaller indexes
- Lower RAM usage
- Faster disk loading

---

## Persistent Indexes

Rather than rebuilding the index after every restart,

future versions will serialize posting lists to disk.

Benefits

- Faster startup
- Persistent search indexes

---

## Distributed Posting Lists

Large indexes can be partitioned.

```
Vocabulary

↓

Shard 1

Shard 2

Shard 3
```

Benefits

- Horizontal scalability
- Parallel search
- Better fault tolerance

---

# Key Design Principles

Throughout TROVIX, the posting list implementation follows several principles.

- One posting per document.
- One posting list per term.
- One vocabulary entry per unique token.
- Sorted posting lists.
- Lightweight posting objects.
- Separation of indexing and ranking.

These principles simplify implementation while maintaining strong performance characteristics.

---

# Conclusion

Posting Lists form the backbone of the TROVIX indexing engine.

They provide an efficient mapping between searchable terms and the documents that contain them, allowing the search engine to retrieve candidate documents without scanning the entire corpus.

This document defined:

- The role of postings and posting lists.
- Vocabulary organization.
- Internal structure of the inverted index.
- Index construction workflow.
- Lookup algorithm.
- Complexity analysis.
- Future optimization opportunities.

Together, these design decisions establish the implementation blueprint for the core indexing data structures.

The next document focuses on the **Document Parser**, which is responsible for transforming raw files into structured `Document` objects that can enter the indexing pipeline.

---

# References

## Books

- *Introduction to Information Retrieval* — Manning, Raghavan & Schütze
- *Managing Gigabytes* — Witten, Moffat & Bell
- *Search Engines: Information Retrieval in Practice* — Croft, Metzler & Strohman

---

## Documentation

- Apache Lucene Documentation
- Elasticsearch Guide
- OpenSearch Documentation

---

## Research

- Robertson & Zaragoza — *The Probabilistic Relevance Framework: BM25 and Beyond*
- Dean & Ghemawat — *MapReduce: Simplified Data Processing on Large Clusters*