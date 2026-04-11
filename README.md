# Aviation-MRO-Local-AI-Assistant-Offline-RAG-Pipeline-
Secure, 100% offline RAG pipeline for Aviation Maintenance (MRO). Uses local LLMs (Phi-3 via Ollama) and ChromaDB to search FAA technical manuals. Provides mechanics with instant, accurate procedural answers while ensuring zero data leakage and strict compliance with aerospace privacy standards.
By running entirely offline, this architecture ensures zero data leakage of proprietary or sensitive maintenance procedures, adhering to strict aerospace data security compliance.

## Tech Stack
* **Language:** Python 3.14
* **LLM Engine:** Ollama (`phi3` model)
* **Embeddings:** Hugging Face (`all-MiniLM-L6-v2`)
* **Vector Database:** ChromaDB
* **Document Processing:** LangChain (`PyPDFLoader`, `RecursiveCharacterTextSplitter`)
* **Environment:** VS Code (Jupyter Notebook interface)

---

## The End-to-End Build Process

This prototype was built in four distinct phases to ensure accurate data extraction, fast semantic searching, and secure local text generation. 

### Phase 1: Environment Setup & Data Acquisition
To ensure the system could run completely offline without relying on cloud APIs (like OpenAI), the environment was configured using local tools.
1. **Local LLM:** Installed [Ollama](https://ollama.com/) and downloaded the Phi-3 model via terminal (`ollama run phi3`).
2. **Dependencies:** Installed required Python libraries directly into the Jupyter Notebook environment:
   ```bash
   !pip install langchain langchain-community pypdf chromadb sentence-transformers
3. Data Source: Downloaded the FAA Aviation Maintenance Technician Handbook - General and extracted Chapter 8 (Cleaning and Corrosion Control) as manual.pdf.

Phase 2: Document Ingestion and Semantic Chunking
An LLM cannot process an entire 800-page manual at once. The PDF was loaded and split into overlapping chunks to preserve the context of the maintenance procedures.
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter

# Load the local manual (using raw string for Windows pathing)
loader = PyPDFLoader(r"manual.pdf")
pages = loader.load()

# Split the document into 1000-character chunks with a 100-character overlap
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000, 
    chunk_overlap=100
)
chunks = text_splitter.split_documents(pages)
print(f"Split the manual into {len(chunks)} chunks!")

Phase 3: Vector Database Creation
To make the text chunks searchable by the AI, they were converted into mathematical embeddings and stored in a local Chroma vector database.
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma

# Initialize the lightweight Hugging Face embedding model
embedding_model = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")

# Build and persist the Chroma database locally
vector_db = Chroma.from_documents(
    documents=chunks, 
    embedding=embedding_model, 
    persist_directory="./mro_chroma_db"
)
print("Vector database built successfully! Ready for queries.")

Phase 4: Custom Retrieval & Generation (RAG) Loop
To ensure maximum stability and avoid framework dependency errors, the standard LangChain QA wrappers were bypassed. A custom retrieval loop was engineered to query the Chroma database and dynamically inject the results directly into the local LLM's prompt.
from langchain_community.llms import Ollama

# Connect to the local Ollama instance
llm = Ollama(model="phi3")

# Define the mechanic's query
query = "What are the common causes of corrosion on an aircraft?"

# Perform a similarity search in the Vector DB for the top 3 most relevant chunks
retrieved_docs = vector_db.similarity_search(query, k=3)

# Combine the retrieved chunks into a single context string
context = "\n\n".join([doc.page_content for doc in retrieved_docs])

# Construct a strict prompt enforcing the LLM to use ONLY the manual excerpts
prompt = f"""You are an expert aviation mechanic. Use ONLY the following manual excerpts to answer the question. 

Manual Excerpts:
{context}

Question: {query}
Answer:"""

# Generate the response completely offline
response = llm.invoke(prompt)
print(response)
