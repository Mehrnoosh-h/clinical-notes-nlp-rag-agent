## Core Components

### 1) Data Loading
Synthetic clinical notes are loaded from a JSONL file into a Pandas DataFrame for downstream processing.

### 2) Section Extraction
Clinical notes are parsed into structured sections using regex-based header detection.  
Examples of extracted sections include:

- `HPI`
- `LABS`
- `PMH`
- `ROS`
- `PE`
- `ALLERGIES`
- `Assessment/Plan`
- `Chief Complaint`
- `Counseling`
- `Narrative`
- `PATIENT`

This converts messy note text into clean section-level columns.

### 3) Lab Parsing
The `LABS` section is parsed into structured lab fields such as:

- HbA1c
- WBC
- Creatinine
- HIV Ag/Ab
- Gonorrhea NAAT
- Chlamydia NAAT

### 4) Feature Engineering
Additional structured features are extracted from text, including:

- Age
- Sex
- Numeric lab values
- Standardized note-level fields

### 5) Patient-Level Aggregation
Note-level data is aggregated into patient-level summaries for downstream querying and analysis.

### 6) Negation Detection
Rule-based regex logic identifies common negated findings, such as:

- denies fever
- no chest pain
- denies shortness of breath
- denies dysuria

### 7) RAG Pipeline
Section-level chunks are embedded using a sentence-transformer model and stored in a FAISS index for fast semantic retrieval.

### 8) Agentic RAG
A GPT-4o-mini tool-calling agent decides which retrieval path to use before answering a user question.

---

## Example of Section Parsing

```text
ALLERGIES: shellfish (anaphylaxis)
HPI: Presents for routine follow-up. Reports variable medication adherence.
LABS:
  - HbA1c: 6.4%
  - WBC: 12.2 K/uL
  - HIV Ag/Ab: pending
```

This is transformed into structured fields such as:

- `allergies`
- `hpi`
- `labs`

---

## Agentic Retrieval Design

The agent uses three tools depending on the question:

| Tool | Purpose |
|------|---------|
| `search_notes` | Semantic search across chunked note sections |
| `get_patient_summary` | Structured patient-level summary lookup |
| `get_all_sections_for_patient` | Full section history for a selected patient |

This allows the system to combine retrieval, structured lookup, and patient-specific context in a multi-step reasoning loop.

---

## Tech Stack

- **Python**
- **pandas**
- **numpy**
- **re** (standard library regex)
- **sentence-transformers**
- **FAISS**
- **OpenAI GPT-4o**
- **OpenAI GPT-4o-mini**
- **Jupyter Notebook**

> Built from scratch without LangChain.

---

## Repository Structure

```text
clinical-notes-nlp-rag-agent/
│
├── data/
│   └── clinical_notes.jsonl
├── notebooks/
│   └── clinical_notes_nlp.ipynb
├── .gitignore
├── LICENSE
└── README.md
```

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/Mehrnoosh-h/clinical-notes-nlp-rag-agent.git
cd clinical-notes-nlp-rag-agent
```

### 2. Install dependencies

```bash
pip install pandas numpy faiss-cpu sentence-transformers openai python-dotenv jupyter
```

### 3. Set your OpenAI API key

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_api_key_here
```

### 4. Verify the data path

Make sure your notebook points to:

```python
DATA_PATH = Path("data/clinical_notes.jsonl")
```

### 5. Run the notebook

```bash
jupyter notebook notebooks/clinical_notes_nlp.ipynb
```

Run all cells from top to bottom.

---

## Key Outputs

Running the notebook generates the main artifacts needed for retrieval and QA, including:

- Parsed note sections
- Structured lab and demographic features
- Patient-level summary tables
- Chunked retrieval records
- FAISS vector index files

If generated locally, retrieval artifacts such as FAISS indexes and chunk records can be excluded with `.gitignore`.

---

## Why This Project Matters

This project shows how NLP, information extraction, vector search, and LLM-based reasoning can be combined into a practical pipeline for working with unstructured clinical text.

It is especially relevant for roles in:

- Data Science
- NLP / LLM Engineering
- Applied Machine Learning
- Healthcare AI
- AI Product Prototyping

---

## Notes

- All clinical notes in this repository are synthetic.
- This project is intended for demonstration and learning.
- The pipeline emphasizes transparency by combining structured extraction with retrieval-grounded generation.

---

## License

This project is licensed under the **MIT License**.
