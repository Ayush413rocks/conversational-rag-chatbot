# Conversational RAG Chatbot

A production-ready conversational AI chatbot built with **FastAPI**, **LangChain**, **ChromaDB**, and **Streamlit** that enables intelligent document Q&A with multi-turn conversational memory.

## Overview

This project implements a **Retrieval-Augmented Generation (RAG)** pipeline that allows users to upload documents (PDF, DOCX, HTML) and interact with their content through a conversational interface. Unlike standard one-shot Q&A systems, this chatbot maintains conversation history across follow-up questions, enabling natural multi-turn dialogue.

The system uses **HuggingFace sentence transformers** for generating semantic embeddings, **ChromaDB** as the vector store for efficient similarity search, and **Groq's Llama 3.3** as the LLM for generating context-aware responses.

## Architecture

```
┌─────────────────┐     HTTP      ┌─────────────────────────────────┐
│   Streamlit UI  │ ────────────► │        FastAPI Backend          │
│   (app/)        │               │        (api/)                   │
└─────────────────┘               │                                 │
                                  │  ┌─────────────┐               │
                                  │  │  LangChain  │               │
                                  │  │  RAG Chain  │               │
                                  │  └──────┬──────┘               │
                                  │         │                       │
                                  │  ┌──────▼──────┐  ┌─────────┐  │
                                  │  │  ChromaDB   │  │  Groq   │  │
                                  │  │  Vector DB  │  │  LLM    │  │
                                  │  └─────────────┘  └─────────┘  │
                                  └─────────────────────────────────┘
```

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Streamlit |
| Backend | FastAPI + Uvicorn |
| RAG Pipeline | LangChain |
| Vector Store | ChromaDB |
| Embeddings | HuggingFace `all-MiniLM-L6-v2` |
| LLM | Groq `llama-3.3-70b-versatile` |
| Document Parsing | PyPDF, Docx2txt |

## Key Features

- **Multi-turn conversational memory** — maintains context across follow-up questions using LangChain's history-aware retriever
- **Multi-format document support** — upload and index PDF, DOCX, and HTML files
- **Semantic search** — uses sentence transformer embeddings for meaning-based retrieval rather than keyword matching
- **RESTful API** — FastAPI backend with full Swagger documentation at `/docs`
- **Real-time document management** — upload, list, and delete documents from the vector store

## Why These Technology Choices

- **Groq over OpenAI** — Groq provides free, ultra-fast LLM inference using custom LPU hardware, making it ideal for development and deployment without API costs
- **HuggingFace embeddings over OpenAI embeddings** — `all-MiniLM-L6-v2` runs locally, eliminating embedding API costs while maintaining strong semantic search quality
- **ChromaDB over FAISS** — ChromaDB provides persistent storage with metadata filtering, enabling document-level deletion which FAISS doesn't support natively
- **FastAPI over Flask** — async support, automatic OpenAPI documentation, and Pydantic validation make FastAPI better suited for production AI APIs

## Project Structure

```
conversational-rag-chatbot/
├── api/
│   ├── main.py              # FastAPI app and route handlers
│   ├── langchain_utils.py   # RAG chain with conversational memory
│   ├── chroma_utils.py      # Document ingestion and vector store
│   ├── db_utils.py          # SQLite session management
│   └── pydantic_models.py   # Request/response schemas
├── app/
│   ├── streamlit_app.py     # Main Streamlit entry point
│   ├── chat_interface.py    # Chat UI components
│   ├── sidebar.py           # Document upload and model selection
│   └── api_utils.py         # FastAPI client calls
├── .env.example
└── requirements.txt
```

## Setup and Installation

### Prerequisites
- Python 3.10+
- Groq API key (free at [console.groq.com](https://console.groq.com))

### Installation

```bash
# Clone the repository
git clone https://github.com/Ayush413rocks/conversational-rag-chatbot.git
cd conversational-rag-chatbot

# Create virtual environment
python -m venv venv
venv\Scripts\activate  # Windows
source venv/bin/activate  # Mac/Linux

# Install dependencies
pip install -r requirements.txt
pip install langchain-groq langchain-huggingface sentence-transformers
```

### Configuration

```bash
cp .env.example .env
```

Edit `.env` and add your Groq API key:

```
GROQ_API_KEY=your_groq_api_key_here
LANGCHAIN_TRACING_V2=false
```

### Running the Application

**Terminal 1 — Start the FastAPI backend:**
```bash
cd api
uvicorn main:app --reload
```

**Terminal 2 — Start the Streamlit frontend:**
```bash
cd app
streamlit run streamlit_app.py
```

Open `http://localhost:8501` in your browser.

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/chat` | Send a message and get a RAG response |
| POST | `/upload-doc` | Upload and index a document |
| GET | `/list-docs` | List all indexed documents |
| POST | `/delete-doc` | Remove a document from the vector store |

Full API documentation available at `http://localhost:8000/docs`

## How It Works

1. **Document Ingestion** — uploaded files are parsed, split into chunks (1000 tokens, 200 overlap), and converted to embeddings using `all-MiniLM-L6-v2`
2. **Vector Storage** — embeddings are stored in ChromaDB with file metadata for efficient retrieval and deletion
3. **Query Processing** — user questions are reformulated using chat history via a history-aware retriever, then used to fetch the top-k most semantically similar chunks
4. **Response Generation** — retrieved chunks are passed as context to Groq's Llama 3.3, which generates a grounded, context-aware response
5. **Memory** — conversation history is maintained per session, enabling natural follow-up questions

## License

MIT
