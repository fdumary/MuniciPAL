# MuniciPAL-CAP6951-Team13
*AI-Powered Policy and Ordinance Assistant*

## Project Overview
MuniciPAL is a Retrieval-Augmented Generation (RAG) system designed to help city staff navigate complex municipal ordinances and grant policies. Documents are chunked, embedded, and indexed in a local ChromaDB vector store; at query time the most relevant passages are retrieved and passed to Google Gemini to produce accurate, grounded responses to regulatory queries.

## Getting Started
### Prerequisites
- Python 3.10+
- Google AI Studio API Key (optional — the engine falls back to Mock Mode without one)

### Security
This project uses a `.env` file for API key management.
1. Create a `.env` file in the root directory.
2. Add your key: `GOOGLE_API_KEY=your_key_here`

### Installation
```bash
git clone https://github.com/allisoncerna/MuniciPAL-CAP6951-Team13.git
cd MuniciPAL-CAP6951-Team13
python -m venv .venv
.venv\Scripts\activate        # Windows (use `source .venv/bin/activate` on macOS/Linux)
pip install -r requirements.txt
```

### Building the retrieval index
```bash
python -m src.build_index
```
This extracts text from every PDF under `data/raw/` (via `src/ingestion.py`), splits it into 512-token chunks with 64-token overlap, embeds each chunk with all-MiniLM-L6-v2, and stores everything in a persistent ChromaDB collection at `data/chroma_db/`.

### Testing the pipeline end-to-end
```bash
python run_pipeline.py          # retrieval -> Gemini smoke test
python -m src.evaluate_retrieval  # retrieval precision metrics (Objective 1)
```

### Running the API backend
```bash
uvicorn src.api:app --reload --port 8000
```
Endpoints (interactive docs at http://localhost:8000/docs):

| Method | Route | Purpose |
| --- | --- | --- |
| GET | `/health` | Index stats and LLM mode |
| POST | `/api/query` | Question -> grounded answer + source chunks |
| POST | `/api/generate` | Wizard answers -> drafted document with citations |
| GET | `/api/documents` | List indexed documents |
| POST | `/api/documents/upload` | Upload a PDF; it is extracted, chunked, and indexed |
| DELETE | `/api/documents/{filename}` | Remove a document from the index |

## Architecture
```
data/raw/*.pdf --> src/ingestion.py --> data/manifest.csv
                       |
                       v
              src/build_index.py  (src/chunking.py: 512-token chunks, 64 overlap)
                       |
                       v
              data/chroma_db/  (ChromaDB, all-MiniLM-L6-v2 embeddings)
                       |
   user query --> src/retrieval.py (top-k semantic search)
                       |
                       v
              src/llm_engine.py (Gemini 2.0 Flash, grounded prompt)
                       ^
              src/api.py (FastAPI) <--> frontend/ (Next.js)
```

## Project Roadmap
    [x] Data Ingestion: Automated PDF text extraction from nested directory structures.
    [x] LLM Generation: Secure integration with Google Gemini 2.0 Flash.
    [x] Vectorization: ChromaDB indexing with all-MiniLM-L6-v2 embeddings.
    [x] Retrieval: Top-k semantic search grounding all LLM outputs.
    [x] Backend API: FastAPI endpoints for query, generation, and document management.
    [x] Frontend: Connect the Next.js interface to the API.

## Data Inventory
The system consumes documents from the City of Delray Beach. Our pipeline processes 68+ ordinances and policies stored in `data/raw/`.

## Data Manifest
The system automatically generates `data/manifest.csv` via `src/ingestion.py`. This manifest acts as the source of truth for the retrieval pipeline.
