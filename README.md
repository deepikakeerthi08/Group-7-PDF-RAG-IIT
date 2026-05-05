A Retrieval-Augmented Generation (RAG) application that lets you chat with PDF documents using semantic search, BM25, and LLM-powered answers.
Features

Hybrid Search — combines semantic vector search (ChromaDB) + BM25 keyword search + cross-encoder reranking
Smart Chunking — classifies and chunks PDF pages differently based on content type (descriptive, curriculum, general)
FastAPI Backend — REST API with a built-in chat UI
Azure OpenAI / OpenAI — generates grounded answers with page-level citations

Setup
1. Clone and open the project
bashcd PDF-RAG-main
2. Create and activate a virtual environment
bashpython -m venv venv

# Windows
venv\Scripts\activate

# Mac/Linux
source venv/bin/activate
3. Install dependencies
bashpip install fastapi uvicorn chromadb sentence-transformers rank-bm25 nltk openai python-dotenv pymupdf
4. Create a .env file
# For Azure OpenAI
OPEN_API_KEY=your_azure_openai_key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=o4-mini

# OR for regular OpenAI (requires editing generate.py)
OPENAI_API_KEY=sk-your-openai-key
5. Add your PDF
Place your PDF in the project root and update line 8 of pipeline.py:
pythonpages = load("your_file.pdf")
6. Ingest the PDF (run once)
bashpython pipeline.py
This processes the PDF and stores chunks in a local chroma_db/ folder.
7. Start the app
bashuvicorn app:app --reload
Open your browser at http://localhost:8000
How It Works

Ingestion (pipeline.py) — PDF is loaded, each page is classified, chunked, and stored in ChromaDB with metadata.
Query (vector.py) — Incoming question triggers hybrid search (semantic + BM25), fused via Reciprocal Rank Fusion (RRF), then reranked by a cross-encoder.
Generation (generate.py) — Top chunks are passed as context to the LLM, which returns a grounded answer with page citations.

Notes

The chroma_db/ folder is created after running pipeline.py and persists across runs.
Re-running pipeline.py will add duplicate chunks — clear chroma_db/ first if re-ingesting.
The HF Hub warning about HF_TOKEN is harmless and can be ignored.