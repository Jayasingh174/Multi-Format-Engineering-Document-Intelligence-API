# 🤖 RFQ Intelligence Platform

### Engineering Document Intelligence, RAG & Cross-Document Conflict Detection

An AI-powered engineering document intelligence platform that processes **RFQ documents, BOQs, specifications, tables, and CAD files** using document extraction, hybrid retrieval, LLM-based structured analysis, and cross-document conflict detection.

The platform converts heterogeneous engineering documents into searchable and structured information, enables **document-grounded RAG question answering**, and compares normalized engineering entities across sources to identify **quantity inconsistencies and conflicts**.

---

## 🚀 Key Features

### 📄 Multi-Format Engineering Document Processing

Supports multiple engineering and business document formats:

* PDF
* DOCX
* XLSX / XLS
* CSV
* TXT
* DWG
* DXF

Documents are detected, extracted, normalized, chunked, and prepared for downstream retrieval and analysis.

---

### 🧠 RAG-Based Document Question Answering

Users can ask questions directly against uploaded engineering documents.

Example:

```text
What is the fire pump capacity?

What quantity of valves is required?

Which document contains this specification?

Summarize all uploaded RFQ documents.
```

The system retrieves relevant document context and passes it to the OpenAI LLM to generate document-grounded answers.

The generation workflow is designed to:

* Use supplied document context
* Avoid unsupported assumptions
* Return precise values where available
* Indicate when information is unavailable

---

### 🔎 Hybrid Retrieval — FAISS + BM25

The platform combines two retrieval approaches:

**Semantic Retrieval**

* OpenAI embeddings
* FAISS vector index
* `IndexFlatIP`
* L2-normalized embeddings

**Keyword Retrieval**

* BM25
* Token-based keyword matching

The results from FAISS and BM25 are combined to improve retrieval coverage across both semantic and exact engineering terminology.

> The current implementation uses result merging rather than weighted score fusion and does not implement CrossEncoder reranking.

---

### 📊 BOQ Intelligence

Excel-based BOQs are converted into searchable and structured information.

Typical fields include:

```text
Item
Description
Quantity
Unit
```

This allows users to query engineering quantities directly from uploaded BOQ documents.

---

### 🧾 Structured Engineering Extraction

The system can convert extracted document content into structured engineering entities.

Example:

```json
{
  "name": "Fire Pump",
  "qty": 2,
  "specification": "500 GPM"
}
```

This structured representation is used by downstream engineering analysis and conflict detection workflows.

---

### 📋 BOM & Specification Extraction

The platform extracts engineering information such as:

* BOM items
* Parts
* Materials
* Quantities
* Material specifications
* Tolerance
* Surface finish
* Coating
* Heat treatment
* Table information

---

### 📐 CAD Intelligence

The platform supports engineering CAD files:

```text
DWG
 ↓
ODA File Converter
 ↓
DXF
 ↓
ezdxf
 ↓
CAD Entities / Blocks
```

Supported CAD entities include:

* LINE
* CIRCLE
* ARC
* DIMENSION
* INSERT

Named `INSERT` blocks can be converted into countable engineering entities for conflict analysis.

---

# ⚠️ Cross-Document Conflict Detection

One of the core capabilities of the platform is comparing engineering information across multiple sources.

For example:

```text
BOQ
Fire Pump → Quantity: 2

CAD
Fire Pump → Quantity: 3

        ↓

Conflict Engine

        ↓

CONFLICT DETECTED
```

### Conflict Detection Pipeline

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
Numeric Validation
        ↓
Quantity Comparison
        ↓
Conflict Detection
        ↓
JSON / CSV Report
```

The conflict engine normalizes:

* Item name
* Quantity
* Source
* Category
* File path

Similar engineering names are matched using fuzzy matching.

For example:

```text
Fire Pump
Fire-Pump
Fire Pump Assembly
```

Numeric dimensions are also validated before similar entities are merged.

For example:

```text
Pipe 10mm
Pipe 20mm
```

are treated as different entities.

The final analysis can contain:

```text
total_entities_checked
conflicts_found
conflict_details
full_matrix
```

---

# 🏗️ System Architecture

```text
                    Web Frontend
                 HTML / CSS / JS
                        │
                        ▼
                  FastAPI API
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
     Document        RAG Query      Conflict
     Processing       Pipeline       Engine
          │             │             │
          ▼             ▼             ▼
     PDF / DOCX      OpenAI        Entity
     Excel / CAD     Embeddings    Normalization
          │             │             │
          ▼             ▼             ▼
       Chunking      FAISS + BM25  Fuzzy Matching
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                   OpenAI LLM
                        │
                        ▼
               Answers / Reports
```

---

# 🔄 RAG Workflow

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
```

---

# 🧠 RAG Implementation

## Document Indexing

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

Each indexed chunk contains text and metadata such as its source document.

Example:

```json
{
  "text": "Fire Pump quantity is 2...",
  "hash": "...",
  "metadata": {
    "source": "document.pdf"
  }
}
```

FAISS uses normalized vectors with:

```text
IndexFlatIP
```

With normalized vectors, inner-product similarity behaves approximately like cosine similarity.

---

# 📁 Project Structure

```text
RFQ-Intelligence-Platform/
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
├── requirements.txt
├── .env
└── README.md
```

---

# 🔌 API Endpoints

| Method   | Endpoint             | Purpose                    |
| -------- | -------------------- | -------------------------- |
| `POST`   | `/upload/process`    | Process a document         |
| `POST`   | `/upload/bundle`     | Process multiple RFQ files |
| `POST`   | `/query/ask`         | Ask RAG questions          |
| `GET`    | `/documents`         | List uploaded documents    |
| `DELETE` | `/delete/{filename}` | Delete a document          |
| `POST`   | `/export/conflicts`  | Export conflict report     |

### Example RAG Request

```json
{
  "question": "What is the fire pump capacity?",
  "top_k": 8
}
```

---

# ⚙️ Configuration

Create a `.env` file:

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

Important application defaults include:

```text
CHUNK_SIZE = 1000
CHUNK_OVERLAP = 200
MAX_UPLOAD_SIZE_MB = 50
```

---

# 🛠️ Tech Stack

| Category          | Technology             |
| ----------------- | ---------------------- |
| Language          | Python                 |
| Backend           | FastAPI                |
| LLM               | OpenAI                 |
| Embeddings        | text-embedding-3-large |
| Vector Search     | FAISS                  |
| Keyword Retrieval | BM25                   |
| Chunking          | LangChain              |
| PDF Processing    | PyMuPDF                |
| DOCX Processing   | python-docx            |
| Excel Processing  | Pandas, OpenPyXL, xlrd |
| CAD Processing    | ezdxf                  |
| DWG Conversion    | ODA File Converter     |
| Validation        | Pydantic               |
| Frontend          | HTML, CSS, JavaScript  |
| Persistence       | FAISS + Pickle         |
| Server            | Uvicorn                |
| Configuration     | python-dotenv          |

---

# 🚀 Installation

Clone the repository:

```bash
git clone https://github.com/Jayasingh174/RFQ-Intelligence-Platform.git
cd RFQ-Intelligence-Platform
```

Create a virtual environment:

```bash
python -m venv venv
```

Activate it on Windows:

```bash
venv\Scripts\activate
```

Install dependencies:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Configure your `.env` file with the required OpenAI API key and application settings.

---

# ▶️ Run the Application

Start the FastAPI server:

```bash
uvicorn app.main:app --reload
```

Then open the application in your browser.

FastAPI automatically provides API documentation at:

```text
http://127.0.0.1:8000/docs
```

---

# 💡 Example Queries

### Information Extraction

```text
Summarize all uploaded RFQ documents.

List all equipment with quantities.

What is the fire pump capacity?

Extract fire safety requirements.

Which document contains this specification?
```

### Conflict Detection

```text
Are there inconsistencies between BOQ and drawings?

Compare M-F1 vs M-F2 drawings.

Are the quantities consistent across the uploaded documents?

Find conflicting equipment quantities.
```

---

# 📦 Engineering Deliverables

The platform can generate structured analysis outputs including:

* JSON analysis reports
* CSV conflict reports
* Document-grounded answers
* Cross-document conflict information

Processed RFQ bundles generate reports under:

```text
deliverables/
```

---

# ⚠️ Current Limitations

The current implementation has several production-readiness limitations:

* Hybrid retrieval currently merges FAISS and BM25 results rather than using weighted score fusion.
* CrossEncoder reranking is not currently implemented.
* Vector metadata is primarily based on source filename.
* DOCX tables are not explicitly extracted.
* Structured LLM extraction uses approximately 8,000 characters of context.
* DWG processing requires ODA File Converter.
* CAD geometry is not automatically interpreted as engineering quantities.
* Conflict detection focuses on entity and quantity mismatches rather than complete engineering-rule validation.
* Local FAISS + Pickle persistence is used.
* Authentication and multi-user isolation are not implemented.
* Upload filenames require stronger sanitization for production deployment.
* CORS is currently permissive.
* Configured upload and internal document-processing limits are not fully consistent.
* Some router/model/function naming inconsistencies should be reconciled before production deployment.
* Current export functionality is CSV-based.

---

# 🔐 Security & Production Considerations

Before production deployment, the following areas should be strengthened:

* Authentication and authorization
* Multi-user document isolation
* Upload filename sanitization
* Strict CORS configuration
* File validation and processing limits
* Persistent production-grade vector storage
* API rate limiting
* Secure secret management
* Robust error handling
* Consistent API/model/function naming

---

# 🎯 Project Highlights

This project demonstrates practical implementation of:

* **Generative AI**
* **Retrieval-Augmented Generation (RAG)**
* **Hybrid Search**
* **Vector Search**
* **LLM-based Structured Extraction**
* **Engineering Document Intelligence**
* **BOQ/BOM Processing**
* **CAD Document Processing**
* **Entity Normalization**
* **Fuzzy Matching**
* **Cross-Document Conflict Detection**
* **FastAPI API Development**
* **Multi-format Document Processing**

---

# 📌 System Summary

```text
                 RFQ INTELLIGENCE PLATFORM
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Documents           RAG          Engineering
      Processing         Search          Analysis
          │                │                │
          ▼                ▼                ▼
   PDF / Excel /       FAISS + BM25     BOQ / BOM /
   DOCX / CAD          + OpenAI LLM      CAD / Specs
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    Grounded Answers
                           +
                    Conflict Reports
                           +
                       JSON / CSV
```

## Core Idea

> **Convert heterogeneous engineering documents into searchable and structured information, use hybrid RAG for document-grounded question answering, and compare normalized engineering entities across multiple sources to identify quantity inconsistencies and conflicts.**
