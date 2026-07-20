# Maintenance Task Search System

An offline Retrieval-Augmented Generation (RAG) prototype that helps users search aviation maintenance documentation and generate context-grounded answers using locally hosted AI models.

> **Important:** This project is an educational prototype. It does not replace approved aircraft maintenance manuals, engineering instructions, regulatory requirements, or authorized maintenance decisions.

---

## Overview

Aviation maintenance professionals work with long technical manuals that contain procedures, warnings, inspection guidance, troubleshooting steps, and safety information.

Finding the right section for a specific maintenance question can require manually searching through large documents. Using public AI services may also be unsuitable when organizations need to keep internal technical documentation private.

This project demonstrates a local RAG pipeline that:

- Processes aviation maintenance documents locally
- Creates semantic embeddings without an external API
- Stores document embeddings in a local vector database
- Retrieves passages relevant to a user question
- Generates an answer using a locally running language model

---

## Problem

Aircraft maintenance manuals are detailed, structured, and procedure-heavy. Users may need to search across many pages before locating the information relevant to a specific question.

A general-purpose AI assistant may generate a plausible answer without grounding it in the approved technical document. Sending internal manuals to an external AI provider may also introduce privacy and data-control concerns.

This project addresses those problems by combining semantic retrieval with a locally hosted language model.

---

## Target Users

This prototype is relevant to:

- Aviation maintenance technicians
- Maintenance, Repair, and Overhaul teams
- Maintenance planners
- Technical publications personnel
- Engineering support teams
- Organizations exploring private document-search systems

---

## What the System Does

The system follows this workflow:

1. Loads an aviation maintenance PDF.
2. Splits the document into overlapping text chunks.
3. Converts the chunks into semantic embeddings.
4. Stores the embeddings in ChromaDB.
5. Converts the user's question into an embedding.
6. Retrieves the most relevant document passages.
7. Adds the retrieved passages to a constrained prompt.
8. Sends the prompt to a locally running Phi-3 model through Ollama.
9. Returns an answer based on the retrieved context.

---

## Architecture

```text
Aviation Maintenance Manual
            |
            v
        PDF Loader
            |
            v
  Recursive Text Splitter
            |
            v
 MiniLM Semantic Embeddings
            |
            v
   Local ChromaDB Index
            |
            v
   Top-K Similarity Search
            |
            v
 Context-Constrained Prompt
            |
            v
   Phi-3 through Ollama
            |
            v
      Generated Answer
```

---

## Technology Stack

| Component | Technology |
|---|---|
| Programming Language | Python |
| Local LLM Runtime | Ollama |
| Language Model | Phi-3 |
| Embedding Model | `all-MiniLM-L6-v2` |
| Vector Database | ChromaDB |
| PDF Loading | LangChain `PyPDFLoader` |
| Text Splitting | `RecursiveCharacterTextSplitter` |
| Development Environment | Jupyter Notebook / VS Code |

---

## Key Design Decisions

### Local Language Model

Phi-3 is executed through Ollama so that document context, user questions, and generated answers can remain on the local machine.

### Local Embeddings

The project uses the Hugging Face `all-MiniLM-L6-v2` model to generate semantic embeddings without relying on an external embedding API.

### Overlapping Text Chunks

The document is divided into chunks of approximately 1,000 characters with a 100-character overlap.

The overlap helps preserve information that may continue across chunk boundaries.

### Persistent Vector Storage

ChromaDB is used to store document embeddings locally. Persistent storage allows the existing document index to be reused instead of rebuilding it for every query.

### Custom Retrieval and Generation Loop

The project manually controls:

- Query embedding
- Similarity retrieval
- Context assembly
- Prompt construction
- Local model invocation

This provides more visibility into the RAG pipeline than relying entirely on a high-level question-answering wrapper.

### Context-Constrained Prompting

The model is instructed to answer only from the retrieved document excerpts.

This can reduce unsupported output, but it does not guarantee correctness. Formal groundedness and answer-quality evaluation remain future work.

---

## Example Query

```text
What are the common causes of corrosion on an aircraft?
```

The system retrieves the most relevant passages from the indexed maintenance document and sends those passages to the local model as supporting context.

---

## Project Structure

```text
Local_AI-Assistant_Offline_RAG_Pipeline/
├── mro_rag.ipynb
├── manual.pdf
├── README.md
├── requirements.txt
└── .gitignore
```

---

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/lakshman199/Local_AI-Assistant_Offline_RAG_Pipeline.git
cd Local_AI-Assistant_Offline_RAG_Pipeline
```

### 2. Create a Virtual Environment

#### Windows

```bash
python -m venv venv
venv\Scripts\activate
```

#### macOS or Linux

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Python Dependencies

```bash
pip install langchain langchain-community langchain-text-splitters pypdf chromadb sentence-transformers
```

You can also create a `requirements.txt` file containing:

```text
langchain
langchain-community
langchain-text-splitters
pypdf
chromadb
sentence-transformers
jupyter
```

Then install the dependencies with:

```bash
pip install -r requirements.txt
```

### 4. Install Ollama

Install Ollama for your operating system.

After installation, download the Phi-3 model:

```bash
ollama pull phi3
```

Confirm that Ollama is working:

```bash
ollama run phi3
```

### 5. Add the Technical Manual

Place the PDF file in the project directory.

Update the file path in the notebook when necessary:

```python
pdf_path = "manual.pdf"
```

---

## Running the Project

1. Start Ollama.
2. Open `mro_rag.ipynb` in Jupyter Notebook or VS Code.
3. Run the notebook cells in order.
4. Confirm that the PDF is loaded successfully.
5. Allow the embedding model to download when it runs for the first time.
6. Wait for the ChromaDB vector index to be created.
7. Replace the sample question with your own aviation maintenance question.
8. Run the retrieval and generation cells.

Example:

```python
query = "What are the common causes of corrosion on an aircraft?"
```

---

## Core Pipeline Example

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_community.llms import Ollama

# Load the PDF
loader = PyPDFLoader("manual.pdf")
documents = loader.load()

# Split the document
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=100
)

chunks = text_splitter.split_documents(documents)

# Create local embeddings
embedding_model = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)

# Store embeddings locally
vector_store = Chroma.from_documents(
    documents=chunks,
    embedding=embedding_model,
    persist_directory="./chroma_db"
)

# Load the local language model
llm = Ollama(model="phi3")

# Retrieve relevant context
query = "What are the common causes of corrosion on an aircraft?"
retrieved_documents = vector_store.similarity_search(query, k=3)

context = "\n\n".join(
    document.page_content for document in retrieved_documents
)

prompt = f"""
You are an aviation maintenance document assistant.

Answer the question using only the provided context.
If the answer is not available in the context, state that the information
was not found in the retrieved document sections.

Context:
{context}

Question:
{query}

Answer:
"""

response = llm.invoke(prompt)

print(response)
```

The exact implementation in the notebook may differ depending on the installed LangChain version.

---

## Current Features

- Local PDF ingestion
- Recursive text chunking
- Overlapping document segments
- Local semantic embeddings
- Persistent ChromaDB storage
- Top-K semantic retrieval
- Local response generation
- Context-constrained prompting
- No external LLM API required during inference

---

## Current Limitations

- The project is currently notebook-based.
- It uses a limited document collection.
- The generated response does not yet display page-level citations.
- Retrieval relevance has not been formally evaluated.
- Answer groundedness and correctness have not been formally measured.
- The system does not verify whether a manual revision is current.
- The project does not include authentication or role-based access control.
- The system has not been validated for operational aircraft maintenance use.
- A local model may still generate incomplete or unsupported information.
- Performance depends on available hardware.

---

## Planned Improvements

### Retrieval Improvements

- Add hybrid keyword and semantic search
- Add a cross-encoder reranker
- Compare different chunk sizes and overlaps
- Add metadata filtering
- Add document-type filtering
- Support multiple manuals

### Answer Quality

- Add page-number and document citations
- Add confidence-based refusal
- Add groundedness evaluation
- Add verified question-and-answer test cases
- Compare multiple local language models
- Track retrieval precision and response latency

### Application Development

- Build a FastAPI backend
- Create a technician-facing web interface
- Add document upload support
- Add query history
- Add source preview panels
- Add feedback collection

### Security and Governance

- Add role-based access control
- Add encrypted storage
- Add audit logging
- Add model-version tracking
- Add document-revision tracking
- Add access restrictions by document type

### Deployment

- Add Docker support
- Add environment configuration
- Document minimum hardware requirements
- Benchmark CPU and GPU performance
- Add automated tests

---

## Evaluation Plan

A future evaluation dataset should contain:

- Maintenance questions
- Verified reference answers
- Relevant document sections
- Correct page numbers
- Expected refusal cases

The system can then be evaluated on:

- Retrieval relevance
- Context precision
- Context recall
- Answer groundedness
- Answer correctness
- Citation accuracy
- Refusal accuracy
- Response latency

No evaluation metrics are currently claimed for this prototype.

---

## Responsible Use

This project is intended as a document-search and research prototype.

Users should always verify generated information against:

- The current approved maintenance manual
- Applicable Airworthiness Directives
- Service Bulletins
- Engineering orders
- Regulatory requirements
- Authorized organizational procedures

The assistant should not be used as the sole basis for an aircraft maintenance action, inspection decision, release-to-service decision, or safety-critical judgment.

---

## Why This Project Matters

This project demonstrates how local AI can support technical document retrieval while providing greater control over sensitive documentation.

It also shows that a useful RAG system depends on more than the language model. Document preparation, chunking, metadata, retrieval quality, citations, evaluation, access control, and responsible-use design are all important parts of the system.

---
