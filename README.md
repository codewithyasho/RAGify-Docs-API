# RAGify Docs API 📚🤖

A comprehensive API for scraping documentation from any URL and asking intelligent questions about it using **Retrieval-Augmented Generation (RAG)** powered by LangChain and Groq.

---

## 🎯 Overview

**RAGify Docs API** is a developer-friendly tool that automates the process of extracting knowledge from documentation websites. Instead of manually reading through documentation, you can now:

- 🔗 Provide a documentation URL
- 🤔 Ask any question about the content
- 🧠 Get intelligent, sourced answers powered by AI

Built with **LangChain**, **FastAPI**, and **Groq's powerful LLM**, this tool handles the heavy lifting of document loading, chunking, embedding, retrieval, and generation—all with caching for performance.

---

## 🚀 Quick Start

### Prerequisites

- Python 3.12 or higher
- Groq API key (free tier available at [console.groq.com](https://console.groq.com))

### Installation

1. **Clone or navigate to the project directory:**

   ```bash
   cd RAGify-Docs-API
   ```

2. **Create a virtual environment:**

   ```bash
   python -m venv .venv
   .venv\Scripts\activate  # On Windows
   # or
   source .venv/bin/activate  # On macOS/Linux
   ```

3. **Install dependencies:**

   ```bash
   pip install -r requirements.txt
   # or
   pip install -e .
   ```

4. **Set up environment variables:**
   Create a `.env` file in the project root:

   ```env
   GROQ_API_KEY=your_groq_api_key_here
   ```

---

## 📖 Usage

### Option 1: FastAPI Server

Start the API server:

```bash
uvicorn app:app --reload
```

The API will be available at `http://localhost:8000`

**API Documentation:**

- Interactive Docs: `http://localhost:8000/docs` (Swagger UI)
- Alternative Docs: `http://localhost:8000/redoc`

### Option 2: Interactive CLI

Run the interactive mode:

```bash
python main.py
```

You'll be prompted to:

1. Enter a documentation URL (e.g., `https://docs.langchain.com/oss/python/langchain/overview`)
2. Ask questions about the scraped documentation
3. View answers with source URLs

---

## 🔌 API Endpoints

### `GET /`

Welcome endpoint with basic information.

**Response:**

```json
{
  "message": "Welcome to the RAGify Docs API. Use the /ragify endpoint to ask questions about documentation from a given URL."
}
```

### `POST /ragify`

Main endpoint for asking questions about documentation.

**Request Body:**

```json
{
  "url": "https://docs.langchain.com/oss/python/langchain/overview",
  "query": "What is LangChain?"
}
```

**Response:**

```json
{
  "answer": "LangChain is a framework for developing applications powered by language models...",
  "sources": [
    "https://docs.langchain.com/oss/python/langchain/overview",
    "https://docs.langchain.com/oss/python/langchain/guides"
  ]
}
```

**Status Codes:**

- `200` — Success
- `500` — Error (RAG initialization or chain invocation failed)

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│             FastAPI Server                   │
│          (app.py with CORS)                 │
└────────────┬────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│      URL Caching Layer                      │
│   (Avoid re-processing same URLs)           │
└────────────┬────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────┐
│         RAG Pipeline (main.py)              │
├─────────────────────────────────────────────┤
│  1. RecursiveUrlLoader → Scrape docs        │
│  2. RecursiveCharacterTextSplitter          │
│  3. HuggingFace Embeddings                  │
│  4. InMemoryVectorStore                     │
│  5. MMR Retriever (k=5, diverse results)    │
│  6. ChatGroq LLM (openai/gpt-oss-120b)      │
│  7. LangChain RAG Chain                     │
└─────────────────────────────────────────────┘
```

### Component Details

| Component | Purpose |
|-----------|---------|
| **RecursiveUrlLoader** | Crawls documentation pages recursively with custom HTML extraction |
| **RecursiveCharacterTextSplitter** | Splits documents (1000 chars/chunk, 200 char overlap) |
| **HuggingFace Embeddings** | Converts text to vectors using `sentence-transformers/all-MiniLM-L6-v2` |
| **InMemoryVectorStore** | Stores embeddings for semantic search |
| **MMR Retriever** | Returns diverse, relevant documents (5 best from 10 candidates) |
| **ChatGroq** | LLM for generating answers (model: `openai/gpt-oss-120b`, temp: 0.2) |

---

## 💻 Example Usage

### Using cURL

```bash
curl -X POST "http://localhost:8000/ragify" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://docs.langchain.com/oss/python/langchain/overview",
    "query": "What is the purpose of LangChain?"
  }'
```

### Using Python Requests

```python
import requests

response = requests.post(
    "http://localhost:8000/ragify",
    json={
        "url": "https://docs.langchain.com/oss/python/langchain/overview",
        "query": "How do I create a RAG chain?"
    }
)

print(response.json())
```

### Using JavaScript/Fetch

```javascript
fetch("http://localhost:8000/ragify", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    url: "https://docs.langchain.com/oss/python/langchain/overview",
    query: "What are chains in LangChain?"
  })
})
.then(res => res.json())
.then(data => console.log(data));
```

---

## ⚙️ Configuration

### Environment Variables

Create a `.env` file to configure the application:

```env
# Groq API Key (required)
GROQ_API_KEY=your_api_key_here

# Optional: Customize embedding model
# EMBEDDING_MODEL=sentence-transformers/all-MiniLM-L6-v2

# Optional: Customize LLM model
# LLM_MODEL=openai/gpt-oss-120b
```

### Tunable Parameters (in [main.py](main.py))

```python
# Chunk size and overlap
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,      # Increase for longer context
    chunk_overlap=200     # Increase for better continuity
)

# Retriever settings
retriever = vector_store.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 5,           # Number of results to return
        "fetch_k": 10,    # Candidates to consider
        "lambda_mult": 0.5  # 1.0 = relevance, 0.0 = diversity
    }
)

# LLM settings
llm = ChatGroq(
    model="openai/gpt-oss-120b",
    temperature=0.2      # Lower = factual, Higher = creative
)
```

---

## 🛠️ Development & Debugging

### Run with hot-reload

```bash
uvicorn app:app --reload
```

### Run tests (if added)

```bash
pytest
```

### Check for issues

```bash
# Verify dependencies
pip check

# Lint code
pylint app.py main.py
```

---

## 🎉 Happy RAGifying

Build smarter, faster, and more informed applications with RAGify Docs API.

**RAGify Docs API** — Because great documentation deserves great answers.
