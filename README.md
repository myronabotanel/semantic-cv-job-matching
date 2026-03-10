# DevMatch 🎯
### Intelligent Talent Allocation System

> An AI-powered application that automatically matches developers to jobs (and vice versa) using a hybrid NLP approach — combining semantic embeddings with TF-IDF keyword analysis.

---

## 📌 Overview

DevMatch was built during a hackathon to solve a real problem: manually matching CVs to job descriptions is slow and inconsistent. Our system automates this by analyzing both the **semantic meaning** and **keyword relevance** of CVs and job descriptions, then ranking matches by a weighted final score.

The system supports two matching directions:
- **Job → CVs**: Upload a job description, get the top 5 matching candidates
- **CV → Jobs**: Upload a CV, find the most suitable job opening

---

## 🧠 How the Matching Works

The core algorithm combines three scoring components:

```
Final Score = 0.10 × Industry Score
            + 0.30 × Technical Skills Score
            + 0.60 × Semantic Matching Score
```

### 1. Semantic Matching (60%)
Uses **spaCy** (`en_core_web_lg`) word embeddings to compute contextual similarity between CV and job description texts. Texts are first preprocessed using lemmatization and stop-word removal, then compared using spaCy's `.similarity()` method. To handle large datasets efficiently, similarity is computed in **parallel batches** using `ThreadPoolExecutor`.

### 2. TF-IDF Keyword Matching (part of semantic score)
**TF-IDF** (Term Frequency–Inverse Document Frequency) vectorization via `sklearn` captures keyword-level overlap. Cosine similarity is computed between CV and job description vectors, then normalized with `MinMaxScaler`. The final semantic score blends both approaches:
```
Semantic Score = 0.6 × spaCy embeddings + 0.4 × TF-IDF cosine similarity
```

### 3. Technical Skills Score (30%)
Users define custom skills with weights (must sum to 100%). The system checks for exact keyword matches using regex, allowing precise control over what technical qualifications matter most for a given role.

### 4. Industry Score (10%)
CVs and job descriptions are pre-classified by industry (IT, Healthcare, Retail, etc.) using **GPT-4** and stored in a SQLite database. Only candidates from matching industries are considered, improving relevance and reducing noise.

---

## 🏗️ Architecture

```
DevMatch/
├── Home.py                  # Streamlit entry point
├── pages/
│   ├── match_job_to_cvs.py  # Job → CV matching logic
│   └── match_cv_to_job.py   # CV → Job matching logic
├── src/
│   ├── db/
│   │   ├── models.py        # SQLAlchemy ORM models
│   │   ├── queries.py       # DB query functions
│   │   └── session.py       # DB connection setup
│   └── preprocessing/
│       ├── openai_client.py     # GPT-4 industry classification
│       ├── document_processor.py # DOCX/PDF parsing
│       ├── parse_cv.py          # CV preprocessing pipeline
│       └── parse_jd.py          # Job description pipeline
├── utils/
│   ├── common_functions.py  # spaCy tokenizer, loaders
│   └── explanation.py       # GPT-3.5 match explanations
├── DataSet/
│   ├── cv/                  # CV documents (.docx)
│   └── job_descriptions/    # Job description documents (.docx)
├── data/
│   └── cvs_metadata.sqlite  # Industry scores database
└── tests/                   # Unit tests (pytest)
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Streamlit |
| NLP / Embeddings | spaCy (`en_core_web_lg`) |
| Keyword Matching | scikit-learn (TF-IDF, cosine similarity) |
| Normalization | scikit-learn (MinMaxScaler) |
| Industry Classification | OpenAI GPT-4 |
| Match Explanations | OpenAI GPT-3.5-turbo |
| Database | SQLite + SQLAlchemy ORM |
| Document Parsing | python-docx, PyPDF2 |
| Testing | pytest + unittest.mock |
| Parallelism | concurrent.futures (ThreadPoolExecutor) |

---

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- OpenAI API key

### Installation

```bash
git clone https://github.com/myronabotanel/semantic-cv-job-matching
cd semantic-cv-job-matching
pip install -r requirements.txt
python -m spacy download en_core_web_lg
```

### Configuration

Create a `.env` file in the root directory:
```
OPENAI_API_KEY=your_key_here
```

### Populate the Database

Before running the app, classify your documents by industry:
```bash
python src/db/populate_cv.py
python src/db/populate_jd.py
```

### Run the App

```bash
streamlit run Home.py
```

---

## 🧪 Testing

```bash
pytest tests/
```

Tests cover:
- Keyword matching scores (full match, no match, partial match)
- Semantic similarity computation
- Industry filtering logic
- Final score calculation
- Full end-to-end flow (mocked)
- Database query functions
- Document processor (file size, MIME type validation)

---

## 👥 Team

| Role | Contribution |
|---|---|
| **Core ML / NLP Algorithm** | TF-IDF + spaCy matching, scoring formula, industry filtering, DB queries, unit tests |
| **Frontend** | Streamlit UI, user interaction flow |
| **LLM Integration (×2)** | GPT-4 industry classification, GPT-3.5 match explanations |

---

## 📄 License

This project was developed for hackathon purposes.
