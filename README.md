Absolutely. I’ll keep the content **exactly as you provided** and only format it as a clean `README.md`.

````markdown
# Multi-Format Engineering Document Intelligence API

An AI-powered engineering document intelligence system that processes RFQ documents, BOQs, CAD files, and other engineering documents using RAG, vector search, LLM-based extraction, and cross-document conflict detection.

---

## 1. Project Overview

The system allows users to:

- Upload engineering/RFQ documents.
- Extract text and structured information.
- Index documents using OpenAI Embeddings + FAISS + BM25.
- Ask document-grounded questions using RAG.
- Extract BOQ, BOM, specifications, tables, and CAD information.
- Compare entities and quantities across multiple documents.
- Detect cross-document quantity conflicts.
- Generate JSON analysis reports and CSV conflict reports.

### Supported files

`PDF | DOCX | XLSX | XLS | CSV | TXT | DWG | DXF`

---

## 2. Main Workflow

### RAG Workflow

```text
Upload Document
      ↓
File-Type Detection
      ↓
Text / BOQ / CAD Extraction
      ↓
Recursive Chunking
      ↓
OpenAI Embeddings
      ↓
FAISS + BM25
      ↓
Hybrid Retrieval
      ↓
Context Construction
      ↓
OpenAI LLM
      ↓
Grounded Answer
````

### Conflict Analysis Workflow

```text
Multiple RFQ Documents
        ↓
Individual Processing
        ↓
Entity Extraction
        ↓
Entity Normalization
        ↓
Deduplication
        ↓
Fuzzy Matching
        ↓
Quantity Comparison
        ↓
Conflict Detection
        ↓
JSON / CSV Report
```

---

## 3. Key Features

### Multi-Format Processing

Supports:

* PDF
* DOCX
* Excel BOQ
* CSV
* TXT
* DWG
* DXF

### RAG Question Answering

Users can ask questions such as:

* What is the fire pump capacity?
* What quantity of valves is required?
* Which document contains this specification?

Answers are generated from retrieved document context.

### BOQ Processing

Excel files are processed to identify fields such as:

* Item
* Description
* Quantity
* Unit

Broad BOQ questions can bypass normal top-K retrieval and use the complete BOQ text.

### Structured Extraction

LLM-based extraction converts document content into structured engineering items:

```json
{
  "name": "Fire Pump",
  "qty": 2,
  "specification": "500 GPM"
}
```

### BOM / Specification / Table Extraction

The system extracts:

* BOM items
* Material
* Quantity
* Material specifications
* Tolerance
* Surface finish
* Coating
* Heat treatment
* Pipe-delimited tables

### CAD Processing

DWG files are converted to DXF using ODA File Converter and parsed using ezdxf.

Supported CAD entities include:

* LINE
* CIRCLE
* ARC
* DIMENSION
* INSERT

---

## 4. Architecture

```text
                Web Frontend
              HTML / CSS / JS
                     │
                     ▼
                FastAPI API
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
 Document         RAG Query     Conflict
 Processing       Pipeline      Engine
       │             │             │
       ▼             ▼             ▼
 PDF/DOCX/       OpenAI        Entity
 Excel/CAD       Embedding     Normalization
 Extraction          │             │
       │             ▼             ▼
       ▼         FAISS + BM25  Fuzzy Matching
   Chunking           │             │
       │              ▼             ▼
       └──────────► OpenAI LLM ◄────┘
                       │
                       ▼
                 Reports / Answer
```

---

## 5. Project Structure

```text
project/
│
├── app/
│   ├── main.py
│   ├── config.py
│   │
│   ├── brain/
│   │   ├── vector_service.py
│   │   ├── embedding_service.py
│   │   ├── llm_service.py
│   │   ├── document_service.py
│   │   ├── chunk_service.py
│   │   └── conflict_engine.py
│   │
│   ├── extraction/
│   │   ├── bom_extractor.py
│   │   ├── spec_extractor.py
│   │   └── table_extractor.py
│   │
│   ├── services/
│   │   ├── cad_service.py
│   │   ├── intelligence_service.py
│   │   ├── excel_service.py
│   │   └── export_service.py
│   │
│   ├── pipeline/
│   │   └── query_pipeline.py
│   │
│   ├── routers/
│   │   ├── upload_router.py
│   │   ├── query_router.py
│   │   └── document_router.py
│   │
│   └── web/
│       ├── index.html
│       ├── app.js
│       └── style.css
│
├── uploads/
├── vectorstore/
├── temp_dxf/
├── deliverables/
└── .env
```

---

## 6. RAG Logic

### Document Indexing

```text
Document
   ↓
Extraction
   ↓
Chunking
   ↓
OpenAI Embedding
   ↓
L2 Normalization
   ↓
FAISS
   +
BM25
```

Each indexed chunk stores:

```json
{
    "text": "...",
    "hash": "...",
    "metadata": {
        "source": "document.pdf"
    }
}
```

### Semantic Search

FAISS uses normalized vectors with:

```text
IndexFlatIP
```

Normalized inner product behaves approximately like cosine similarity.

### Keyword Search

BM25 performs keyword-based retrieval over document tokens.

### Hybrid Search

The implementation combines:

```text
FAISS results
+
BM25 results
```

and removes duplicate documents.

> Note: this implementation does not use a 70/30 weighted score or CrossEncoder reranking.

### LLM Generation

The retrieved chunks are converted into context and passed to the OpenAI LLM.

The prompt instructs the model to:

* use only supplied context
* avoid outside assumptions
* provide precise values
* state when information is unavailable

---

## 7. Conflict Detection / Main Core Logic

The conflict engine compares engineering entities across multiple sources.

### Step 1 — Entity Normalization

Item Name
Quantity
Source
Category
File Path

are converted into a common structure.

### Step 2 — Deduplication

Entities are grouped using:

```text
Normalized Item
+
Numeric Dimensions
+
Source
```

Quantities are aggregated.

### Step 3 — Fuzzy Matching

Similar names are matched using difflib.

Example:

```text
Fire Pump
Fire-Pump
Fire Pump Assembly
```

### Step 4 — Numeric Validation

Numeric values must match before similar entities are merged.

For example:

```text
Pipe 10mm
Pipe 20mm
```

are treated as different entities.

### Step 5 — Quantity Comparison

Example:

```text
BOQ → Fire Pump → 2

CAD → Fire Pump → 3
```

Result:

```text
CONFLICT
```

The final analysis contains:

```text
total_entities_checked
conflicts_found
conflict_details
full_matrix
```

---

## 8. Project-Specific Engineering Intelligence Logic

### BOQ Intelligence

Excel BOQs are converted into structured/searchable information.

```text
Excel
 ↓
Sheets
 ↓
Rows
 ↓
Item / Description / Quantity
 ↓
Normalized Entity
```

### BOM Extraction

Pipe-delimited BOM rows are converted into:

```text
Part
Material
Quantity
```

### Specification Extraction

The system extracts:

* Material
* Tolerance
* Surface Finish
* Coating
* Heat Treatment

### CAD Intelligence

```text
DWG
 ↓
ODA Converter
 ↓
DXF
 ↓
ezdxf
 ↓
CAD Entities + Blocks
```

Named INSERT blocks can become countable entities for conflict analysis.

### Structured LLM Extraction

The system sends extracted document content to the LLM and requests structured JSON containing:

```text
Project
Items
Quantity
Specification
```

The extraction context is limited to approximately 8,000 characters.

### Engineering Reports

Processed RFQ bundles generate JSON reports under:

```text
deliverables/
```

Conflict analysis can also be exported as CSV.

---

## 9. API Endpoints

| Method | Endpoint             | Purpose                    |
| ------ | -------------------- | -------------------------- |
| POST   | `/upload/process`    | Process a document         |
| POST   | `/upload/bundle`     | Process multiple RFQ files |
| POST   | `/query/ask`         | Ask RAG questions          |
| GET    | `/documents`         | List uploaded documents    |
| DELETE | `/delete/{filename}` | Delete document            |
| POST   | `/export/conflicts`  | Export conflict report     |

### Example RAG request

```json
{
  "question": "What is the fire pump capacity?",
  "top_k": 8
}
```

---

## 10. Setup & Run

### Install

```bash
pip install -r requirements.txt
```

### Configure `.env`

```env
OPENAI_API_KEY=your_api_key

OPENAI_MODEL=gpt-4o-mini
EMBEDDING_MODEL=text-embedding-3-large
EMBEDDING_DIMENSION=3072

CHUNK_SIZE=1000
CHUNK_OVERLAP=200

TOP_K=8
MAX_CONTEXT_CHARS=12000

UPLOAD_DIR=uploads
SAVE_DIR=vectorstore

ODA_PATH=C:\path\to\ODAFileConverter.exe
```

### Run

```bash
uvicorn app.main:app --reload
```

Then open the FastAPI application in the browser.

---

## 11. Current Limitations

* Hybrid retrieval is result merging rather than weighted score fusion.
* No CrossEncoder reranking is implemented.
* Vector metadata mainly contains source filename.
* DOCX tables are not explicitly extracted.
* Structured LLM extraction uses approximately 8,000 characters of context.
* DWG processing requires ODA File Converter.
* CAD geometry is not automatically interpreted as engineering quantities.
* Conflict detection focuses on entity/quantity mismatches rather than full engineering rule validation.
* Local FAISS + pickle persistence is used.
* Authentication and multi-user isolation are not implemented.
* Upload filenames require stronger sanitization for production use.
* CORS is currently permissive.
* The configured upload limit and internal document-processing limit are not fully consistent.
* Some router/model/function naming inconsistencies should be reconciled before production deployment.
* Current export functionality is CSV-based.

---

## 12. Tech Stack

| Category          | Technology             |
| ----------------- | ---------------------- |
| Language          | Python                 |
| Backend           | FastAPI                |
| LLM               | OpenAI                 |
| Embeddings        | text-embedding-3-large |
| Vector Database   | FAISS                  |
| Keyword Retrieval | BM25                   |
| Chunking          | LangChain              |
| PDF               | PyMuPDF                |
| DOCX              | python-docx            |
| Excel             | Pandas, OpenPyXL, xlrd |
| CAD               | ezdxf                  |
| DWG Conversion    | ODA File Converter     |
| Validation        | Pydantic               |
| Frontend          | HTML, CSS, JavaScript  |
| Persistence       | FAISS + Pickle         |
| Server            | Uvicorn                |
| Configuration     | python-dotenv          |

---

# System Summary

```text
               RFQ AI SYSTEM
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   Documents        RAG        Engineering
   Processing      Search       Analysis
        │             │             │
        ▼             ▼             ▼
 PDF / Excel /   FAISS + BM25   BOQ / BOM /
 DOCX / CAD      + OpenAI LLM   CAD / Specs
        │             │             │
        └─────────────┼─────────────┘
                      ▼
             Grounded Answers
                      +
             Conflict Reports
                      +
               JSON / CSV
```

**Core idea:** convert heterogeneous engineering documents into searchable and structured information, use RAG for document-grounded Q&A, and compare normalized entities across sources to identify quantity inconsistencies.

```
```
