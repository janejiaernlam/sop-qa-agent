# SOPQuery
 
A locally-run Retrieval-Augmented Generation (RAG) pipeline that lets pharmaceutical staff query Standard Operating Procedure (SOP) documents in natural language and get cited, source-grounded answers.
 
Built as a capstone project for the HDip in AI/ML at the National College of Ireland (2026).
 
## Problem
 
Pharmaceutical SOPs are long, dense, and version-controlled. Finding a specific procedure manually is slow and carries a compliance risk if the wrong document or section is referenced. SOPQuery lets a user type a question and get an answer pulled from the relevant SOP section, with the source document and page clearly cited, so the answer can be checked against the original text.
 
## Features
 
- PDF ingestion and chunking of SOP documents
- Local embeddings (no data leaves the machine) and a persistent vector store
- Natural language query interface with source citation (document + page)
- No-match handling for queries outside the loaded document set
- Follow-up questions within the same session (conversation history)
- Automated evaluation of answer quality against a held-out query set
## Pipeline
 
```
PDF SOPs → chunking → embedding → vector store (Chroma)
                                        │
query → retrieval ─────────────────────┘
      → prompt construction (context + history)
      → LLM generation (Groq)
      → source citation
      → answer
```
 
## Tech stack

| Component | Tool |
|---|---|
| Orchestration | LangChain |
| Embeddings | HuggingFace `all-MiniLM-L6-v2` (local) |
| Vector store | ChromaDB (local, persistent) |
| LLM | Groq API, `llama-3.3-70b-versatile` |
| Evaluation | RAGAS (faithfulness, context relevance) |
| Interface | Jupyter Notebook |
 
## Evaluation
 
Evaluated with RAGAS on a 10-query baseline set and a separate 5-query held-out set with isolated sessions per query.
 
| Set | Faithfulness | Context relevance |
|---|---|---|
| Baseline (n=10) | 0.63 | 0.90 |
| Held-out (n=5) | 0.70 | 0.65 |
 
Chunk size was tuned from 500 to 1000 characters; this gave no meaningful improvement (faithfulness 0.625, context relevance 0.90), so the 500/50 chunk size and overlap were kept.
 
### Known limitations
 
- **Retrieval ranking**: for one query the correct chunk was retrieved but not ranked first, weakening the answer.
- **Chunk boundary splitting**: a key term ("Clinical Division Director") was split across a chunk boundary, so the retrieved context was incomplete for that query.
- **Table content dilution**: a query targeting tabular content (SOPP-8217, Appendix A) suffered because table structure does not embed well as flat text.
- **LLM-as-judge disagreement**: RAGAS faithfulness scores for two queries did not match manual review of the same answers against the source text.
- RAGAS faithfulness varied between repeated runs on the same held-out set (0.56 vs 0.70) due to LLM non-determinism; context relevance was stable across runs and used as the more reliable metric.
## Data
 
Five publicly available FDA guidance/SOP documents, from [FDA Guidance Documents](https://www.fda.gov/regulatory-information/search-fda-guidance-documents):
 
- SOPP-8201: Administrative Processing of Clinical Holds for INDs
- SOPP-8212: Breakthrough Therapy Products, Designation and Management
- SOPP-8217: Administrative Processing and Review Management Procedures for IND Applications
- Guidance on Computer Software Assurance for Production and Quality System Software
- Out-of-Specification (OOS) Investigations guidance (L2-OOS)
## Project structure
 
```
sopquery/
├── data/FDA/            # source SOP/guidance PDFs
├── docs/                # requirements spec, project framework
├── notebooks/
│   └── sop_pipeline.ipynb   # full pipeline: ingestion → evaluation
└── requirements.txt
```
 
## Setup
 
```bash
git clone https://github.com/janejiaernlam/sopquery.git
cd sopquery
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
```
 
Create a `.env` file in the project root with a Groq API key:
 
```
GROQ_API_KEY=your_key_here
```
 
Open `notebooks/sop_pipeline.ipynb` and run the cells in order. Each stage (ingestion, chunking, embedding, retrieval, generation, citation, conversation history, evaluation) is documented inline with its own test log.
 
## Notes
 
- Runs fully locally aside from the Groq API call for generation; the embedding model itself runs on-device once downloaded.
- Downloading the embedding model requires an unrestricted connection to HuggingFace Hub on first run.
- No web interface: this is a notebook-based pipeline, by design (out of scope for this iteration — see `docs/SOPQuery_SRS.docx`).
- Academic project, not intended for production or clinical use.