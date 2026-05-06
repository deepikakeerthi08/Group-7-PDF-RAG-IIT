Setup & Installation
1. Clone the repository
bashgit clone https://github.com/deepikakeerthi08/Group-7-PDF-RAG-IIT.git
cd Group-7-PDF-RAG-IIT
2. Create and activate a virtual environment
python -m venv venv

# Windows
venv\Scripts\activate

# Mac/Linux
source venv/bin/activate

3. Install dependencies
pip install fastapi uvicorn chromadb sentence-transformers rank-bm25 nltk openai python-dotenv pymupdf

5. Create a .env file
OPEN_API_KEY=your_azure_openai_key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=o4-mini

7. Add the PDF
Place IIT Graduate Catalog 2024-2025_final.pdf in the project root.

9. Run the ingestion pipeline (once only)
python pipeline.py
This takes ~34 seconds and stores 2,494 chunks in a local chroma_db/ folder.

11. Start the application
uvicorn app:app --reload
Open your browser at http://localhost:8000

How It Works
Ingestion Pipeline 
PDF → pageSplitter → Classifier → Chunker → Metadata Extractor → ChromaDB (2,494 chunks)

pageSplitter loads and cleans each of the 618 pages
Classifier labels each page: skip / toc / descriptive / curriculum / prose / mixed
Chunker applies type-specific strategy:

Descriptive: regex splits on course code boundaries (1 chunk = 1 course)
Curriculum: whole page retained as 1 chunk (preserves requirement tables)
Prose/Mixed: semantic chunking via sentence similarity (NLTK + all-MiniLM-L6-v2)

Metadata extractor enriches chunks with course code, credits, department, program name
All chunks stored in ChromaDB + BM25 index built in memory

Query Pipeline 
User Query → Router → Retriever (BM25 + Semantic + RRF) → Cross-Encoder Reranker → o4-mini → Answer + Citations

Router detects query intent (Topic Search / Program Requirements / Policy / Admission / Financial / General)
Retriever runs semantic search (top-15) + BM25 (top-15), fused via RRF (k=60)
Cross-encoder reranks top-15 fused results
Generator (Azure OpenAI o4-mini) produces grounded answer with page citations

Pipeline Latency
StageTimePDF ingestion + chunking (one-time) 34,000 msChromaDB semantic query 95 msBM25 lexical query 18 msCross-encoder reranking (top-15) 310 msLLM generation (o4-mini) 2,400 msEnd-to-end retrieval~1,260 ms

Dataset Statistics
AttributeValueTotal PDF Pages618Pages Retained601Total Chunks Indexed2,494Unique Courses1,328Unique Departments34Unique Graduate Programs169Mean Chunk Length808.5 charsChromaDB Storage~120 MB

Notes
chroma_db/ is created after running pipeline.py and persists across runs — do not upload to GitHub
.env contains API keys — never upload to GitHub
The HF Hub warning about HF_TOKEN during startup is harmless and can be ignored
Re-running pipeline.py will add duplicate chunks — delete chroma_db/ folder first if re-ingesting
