# 📚 LangChain RAG (Retrieval-Augmented Generation) Project

A complete Retrieval-Augmented Generation (RAG) pipeline built using **LangChain**, **ChromaDB**, **HuggingFace Embeddings**, and **Groq Llama 3.1**.

This project converts PDF documents into vector embeddings, stores them inside a Chroma vector database, and answers user queries using semantic search + LLM.

---

# Project Architecture

                    PDF Documents
                          │
                          ▼
                Directory Loader
                          │
                          ▼
                 Document Loading
                          │
                          ▼
                  Text Splitting
              (Chunking Documents)
                          │
                          ▼
             HuggingFace Embeddings
                          │
                          ▼
                 Chroma Vector Store
                          │
────────────────────────────────────────────────
                          │
                  User Question
                          │
                          ▼
                  Vector Retrieval
          (Most Relevant Document Chunks)
                          │
                          ▼
                LangChain RetrievalQA
                          │
                          ▼
                Groq Llama-3.1 LLM
                          │
                          ▼
                    Final Answer

---

# Project Structure

```
project/
│
├── docs/
│      document1.pdf
│      document2.pdf
│      ...
│
├── vector_store/
│      (Generated Chroma Database)
│
├── langchain_rag_doc_ingestion.ipynb
│
├── langchain_rag_retrevial_Generation.ipynb
│
└── README.md
```

---

# Technologies Used

- Python
- LangChain
- ChromaDB
- HuggingFace Embeddings
- Sentence Transformers
- Groq API
- Llama 3.1 8B Instant
- NLTK

---

# Project Workflow

The complete project is divided into **two independent stages.**

---

# Phase 1 : Document Ingestion

Notebook:

```
langchain_rag_doc_ingestion.ipynb
```

This notebook prepares the knowledge base.

---

## Step 1 : Import Required Libraries

The project imports:

- DirectoryLoader
- CharacterTextSplitter
- HuggingFaceEmbeddings
- Chroma Vector Store

These libraries are responsible for:

- Loading PDFs
- Splitting text
- Creating embeddings
- Saving vectors

---

## Step 2 : Configure Paths

The following paths are configured:

- PDF folder
- Vector database folder
- Collection name

Example:

```python
docs_dir_path
vector_store_path
collection_name
```

---

## Step 3 : Initialize Embedding Model

The project loads the default HuggingFace embedding model.

```
sentence-transformers/all-MiniLM-L6-v2
```

Purpose:

Convert every text chunk into a numerical vector representation.

Without embeddings, semantic search is impossible.

---

## Step 4 : Load PDF Documents

The project uses

```
DirectoryLoader
```

to automatically load every PDF inside the document directory.

Result:

```
PDF Files
      ↓
LangChain Documents
```

---

## Step 5 : Split Documents into Chunks

Large documents cannot be embedded directly.

Therefore,

```
CharacterTextSplitter
```

splits every document.

Configuration:

```
Chunk Size    : 1000 characters

Chunk Overlap : 500 characters
```

Why overlap?

Because context should not be lost between neighboring chunks.

Example:

```
Chunk 1

------------
Sentence A
Sentence B
Sentence C
------------

Chunk 2

------------
Sentence C
Sentence D
Sentence E
------------
```

Notice that Sentence C appears in both chunks.

---

## Step 6 : Generate Embeddings

Each chunk is passed through

```
HuggingFaceEmbeddings
```

Output:

```
Text Chunk

↓

Vector (Embedding)

↓

768-dimensional numerical representation
```

These vectors capture semantic meaning.

---

## Step 7 : Store Embeddings inside ChromaDB

Finally,

```
Chroma.from_documents()
```

creates a persistent vector database.

Stored information includes:

- Text chunk
- Embedding vector
- Metadata

The vector database is saved on disk.

This means embeddings do not need to be recreated every time.

---

# End of Phase 1

Output:

```
PDFs

↓

Chunks

↓

Embeddings

↓

Persistent Chroma Vector Database
```

---

# Phase 2 : Retrieval + Generation

Notebook:

```
langchain_rag_retrevial_Generation.ipynb
```

This notebook performs Question Answering.

---

## Step 1 : Load Embedding Model

Exactly the same embedding model is loaded.

This is extremely important.

Why?

Because the query embedding must be created using the same model that created document embeddings.

Otherwise similarity search will fail.

---

## Step 2 : Initialize LLM

The project uses

```
Groq
```

with

```
Llama-3.1-8B-Instant
```

Configuration:

- Temperature = 0

Meaning:

The responses are deterministic and consistent.

---

## Step 3 : Load Existing Chroma Database

Instead of recreating embeddings,

the notebook loads the existing vector database.

```
Chroma(
    persist_directory=...
)
```

Now the system already has knowledge of all PDFs.

---

## Step 4 : Create Retriever

```
retriever =
vector_store.as_retriever()
```

Purpose:

Given a user question,

retrieve the most relevant chunks.

The retriever performs semantic similarity search.

Not keyword search.

---

## Step 5 : Create RetrievalQA Chain

```
RetrievalQA.from_chain_type()
```

This chain combines:

```
Retriever

+

LLM
```

Workflow:

```
User Question

↓

Retriever

↓

Relevant Chunks

↓

LLM

↓

Answer
```

---

## Step 6 : User Query

Example:

```python
query = "What is RCCGnet?"
```

---

## Step 7 : Retrieval

The retriever searches the vector database.

Suppose thousands of chunks exist.

Only the most relevant chunks are selected.

Example:

```
10000 Chunks

↓

Top Relevant Chunks

↓

Pass to LLM
```

---

## Step 8 : Generation

The retrieved chunks become the context for the LLM.

The LLM answers only using retrieved information.

This reduces hallucination.

---

## Step 9 : Final Output

The notebook prints:

- Final Answer
- Source Documents (optional)

---

# Complete Data Flow

```
PDF Files

↓

Directory Loader

↓

Documents

↓

Text Splitter

↓

Chunks

↓

Embeddings

↓

Chroma Vector Store

====================================

User Question

↓

Embedding

↓

Similarity Search

↓

Relevant Chunks

↓

Groq Llama 3.1

↓

Generated Answer
```

---

# Components Explained

## DirectoryLoader

Loads all PDF files automatically.

---

## CharacterTextSplitter

Splits large documents into smaller chunks.

---

## HuggingFaceEmbeddings

Converts text into vector embeddings.

---

## ChromaDB

Stores embeddings efficiently.

Supports semantic search.

---

## Retriever

Finds the most relevant chunks based on vector similarity.

---

## RetrievalQA

Combines retrieval with an LLM.

Instead of asking the LLM directly,

it first retrieves relevant knowledge.

---

## Groq Llama 3.1

Responsible for generating the final natural-language response.

---

# Why RAG?

Without RAG:

```
Question

↓

LLM

↓

May Hallucinate
```

With RAG:

```
Question

↓

Retriever

↓

Relevant Context

↓

LLM

↓

Accurate Answer
```

---

# Advantages

✔ Uses your own documents

✔ Reduces hallucinations

✔ Fast semantic search

✔ Persistent vector database

✔ Scalable

✔ Easy to update documents

✔ Modular architecture

---

# Current Pipeline

```
PDFs

↓

Load

↓

Chunk

↓

Embed

↓

Store

↓

Retrieve

↓

Generate

↓

Answer
```

---

# Future Improvements

- Support DOCX
- Support TXT
- Multi-PDF collections
- Metadata filtering
- Conversational Memory
- Hybrid Search (BM25 + Vector Search)
- Streaming responses
- Web UI using Streamlit
- Citation support
- Multi-query retrieval
- Re-ranking models

---

# Summary

This project demonstrates an end-to-end Retrieval-Augmented Generation (RAG) pipeline.

The ingestion notebook is responsible for creating a searchable vector knowledge base from PDF documents. The retrieval notebook loads that knowledge base, retrieves the most relevant document chunks using semantic similarity search, and passes them to the Groq Llama 3.1 model to generate accurate, context-aware answers.

The overall workflow is:

PDF → Chunking → Embedding → ChromaDB → Retrieval → LLM → Answer