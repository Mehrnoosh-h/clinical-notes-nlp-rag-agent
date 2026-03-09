# clinical-notes-nlp-rag-agent

# 🏥 Clinical Notes NLP Pipeline

An end-to-end NLP pipeline for parsing, structuring, analysing, and querying free-text synthetic clinical notes in JSONL format — from raw unstructured text to a fully agentic RAG assistant.

> ⚠️ All patient data used in this project is entirely **synthetic and fictional**. Do not use for clinical decision-making.

---

## 📌 What This Project Demonstrates

| Skill | How it shows up |
|-------|----------------|
| NLP text parsing | Regex-based section header detection across 17 clinical note fields |
| Information extraction | Structured key-value parsing of LABS, demographics, and negation patterns |
| Data engineering | Note-level → patient-level aggregation with Pandas |
| Semantic search | FAISS vector index over 3,308 section-level chunks |
| RAG | GPT-4o grounded Q&A with structured JSON evidence citations |
| Agentic AI | Multi-tool GPT-4o-mini agent with decision-making across 3 retrieval strategies |

---

## 🏗️ Pipeline Overview

```
JSONL → Parse → Extract Sections → Parse Labs → Feature Engineering
                                                        ↓
                                              Patient-Level Summary
                                                        ↓
                                              Negation Detection
                                                        ↓
                                         Chunk → Embed → FAISS Index
                                                        ↓
                                         RAG (GPT-4o) + Agent (GPT-4o-mini)
```

| Step | Description |
|------|-------------|
| 1. Data Loading | Read 364 clinical notes from JSONL into a Pandas DataFrame |
| 2. Section Extraction | Regex parser detects 17 unique section headers (HPI, LABS, PMH, ROS…) and extracts each into a column |
| 3. Lab Parsing | Parse the LABS section into structured `lab_*` columns (HbA1c, WBC, Creatinine, HIV Ag/Ab…) |
| 4. Feature Engineering | Extract age, sex from the PATIENT field; normalise lab values to numeric |
| 5. Patient-Level Summary | Aggregate 364 notes → 60 patients: note count, last visit, max HbA1c, most recent HIV result |
| 6. Negation Detection | Rule-based regex flags (denies fever, no chest pain, denies SOB, denies dysuria) with unit tests |
| 7. RAG Pipeline | 3,308 section chunks embedded with MiniLM-L6-v2, stored in FAISS, answered with GPT-4o |
| 8. Agentic RAG | GPT-4o-mini tool-calling agent with 3 tools and multi-round reasoning |

---

## 📊 Dataset

- **364 clinical notes** across **60 patients**
- **Note types**: Discharge Summary, ED Note, Progress Note, and more
- **17 section headers** detected: `HPI`, `LABS`, `PMH`, `ROS`, `PE`, `ALLERGIES`, `Assessment/Plan`, `Chief Complaint`, `Counseling`, `Narrative`, `PATIENT`, and others
- **6 lab columns** extracted: HbA1c, WBC, Creatinine, HIV Ag/Ab, Gonorrhea NAAT, Chlamydia NAAT

---

## 🔍 Section Extraction

Clinical notes are unstructured free text. The parser automatically discovers all section headers across the dataset and extracts their content into columns using a regex lookahead that stops at the next known header:

```
ALLERGIES: shellfish (anaphylaxis)
HPI: Presents for routine follow-up. Reports variable medication adherence.
LABS:
  - HbA1c: 6.4%
  - WBC: 12.2 K/uL
  - HIV Ag/Ab: pending
```
→ becomes structured columns `allergies`, `hpi`, `labs`, etc.

---

## 🤖 Agentic RAG

The agent has access to **3 tools** and decides which to use per question:

| Tool | When the agent uses it |
|------|----------------------|
| `search_notes` | Semantic search across all 3,308 chunks — best for finding specific clinical facts |
| `get_patient_summary` | Structured patient-level data — best for aggregate questions (max HbA1c, note count) |
| `get_all_sections_for_patient` | Full note history for one patient — best for comprehensive patient summaries |

Multi-round loop (`max_tool_rounds=3`) allows the agent to call multiple tools before committing to an answer.

---

## 🛠️ Tech Stack

- **Data** — `pandas`, `numpy`
- **NLP parsing** — `re` (standard library regex)
- **Embeddings** — `sentence-transformers` (`all-MiniLM-L6-v2`, 384-dim)
- **Vector search** — `faiss-cpu` (IndexFlatIP / cosine similarity)
- **LLM** — OpenAI `gpt-4o` (RAG generation) + `gpt-4o-mini` (agent)
- **No LangChain** — everything built from scratch

---

## 📁 Repository Structure

```
clinical-notes-nlp/
│
├── notebook/
│   └── clinical_notes_nlp.ipynb    # Main notebook (full pipeline)
├── data/
│   └── clinical_notes.jsonl        # Synthetic clinical notes (not committed — see below)
├── .gitignore
├── LICENSE
└── README.md
```

> **Note:** `rag_chunks.index` and `rag_chunks.json` are generated automatically when you run the notebook. They are excluded from the repo (see `.gitignore`).

---

## 🚀 Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/Mehrnoosh-h/clinical-notes-nlp.git
cd clinical-notes-nlp
```

### 2. Install dependencies

```bash
pip install pandas numpy faiss-cpu sentence-transformers openai python-dotenv
```

### 3. Set your OpenAI API key

Create a `.env` file in the project root:

```
OPENAI_API_KEY=sk-...
```

### 4. Add the data file

Place your JSONL file at:

```
data/clinical_notes.jsonl
```

Or update `DATA_PATH` in the notebook.

### 5. Run the notebook

```bash
jupyter notebook notebook/clinical_notes_nlp.ipynb
```

Run all cells top to bottom. The FAISS index and chunk records will be generated automatically on first run.

---

## ⚙️ Configuration

| Constant | Default | Description |
|----------|---------|-------------|
| `EMBED_MODEL_NAME` | `all-MiniLM-L6-v2` | Sentence embedding model |
| `INDEX_PATH` | `rag_chunks.index` | FAISS index save path |
| `CHUNKS_PATH` | `rag_chunks.json` | Chunk records save path |
| `k` | `5` | Chunks retrieved per query |
| `max_tool_rounds` | `3` | Max agent tool-call rounds |

---

## 🔒 .gitignore

```
.env
rag_chunks.index
rag_chunks.json
data/
__pycache__/
.ipynb_checkpoints/
```

---

## 📄 License

MIT License. All clinical data is entirely synthetic.
