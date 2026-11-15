# 📚 Machine Learning Knowledge Assistant (End-to-End RAG Pipeline)

An end-to-end **Retrieval-Augmented Generation (RAG)** application built in Python that answers technical Machine Learning and Deep Learning questions grounded strictly in a reference textbook (*Introduction to Machine Learning with Python* — 392 pages).

---

## 🚀 Overview & Key Features

* **PDF Ingestion & Cleaning (`PyMuPDF`)**: Extracts text across all 392 pages while preserving exact page metadata, filtering empty pages, and fixing line-break hyphenation artifacts.
* **Token-Aware Chunking (`LangChain` + `tiktoken`)**: Splits textbook content into **402 overlapping semantic chunks** (750-token limit, 100-token overlap, averaging ~424 tokens/chunk) using `cl100k_base` encoding.
* **Dense Vector Embeddings (`SentenceTransformers`)**: Encodes text chunks into 384-dimensional normalized vectors using `all-MiniLM-L6-v2`.
* **Fast Vector Search (`FAISS`)**: Stores embeddings in a `faiss.IndexFlatIP` index to perform exact Cosine Similarity retrieval across the book.
* **Grounded Answer Generation (`Gemini 3.5 Flash-Lite`)**: Uses low-temperature (`0.1`) prompting with strict system instructions to generate answers **solely** from retrieved textbook excerpts, complete with inline page citations `(Page X)` and hallucination refusal for out-of-domain queries.
* **Interactive Real-Time Q&A**: Includes an interactive CLI loop inside the notebook for live questioning.

---

## 🏗️ RAG Architecture Workflow

```text
┌───────────────────┐     ┌─────────────────────────┐     ┌──────────────────────────┐
│   PDF Textbook    │────▶│  PyMuPDF + Regex Clean  │────▶│  Recursive Token Splitter│
│   (392 Pages)     │     │  (Page Metadata Kept)   │     │  (750 tok / 100 overlap) │
└───────────────────┘     └─────────────────────────┘     └────────────┬─────────────┘
                                                                       │ 402 Chunks
                                                                       ▼
┌───────────────────┐     ┌─────────────────────────┐     ┌──────────────────────────┐
│  Grounded Answer  │◀────│ Gemini 3.5 Flash-Lite   │◀────│  SentenceTransformers +  │
│  + Page Citations │     │ (Temp: 0.1, Strict RAG) │     │  FAISS Vector Index      │
└───────────────────┘     └─────────────────────────┘     └──────────────────────────┘
```

---

## 🛠️ Tech Stack

| Component | Tool / Model Used | Purpose |
| :--- | :--- | :--- |
| **PDF Parsing** | `PyMuPDF` (`fitz`) | Fast page-by-page text extraction |
| **Text Splitting** | `langchain-text-splitters`, `tiktoken` | Token-measured recursive chunking |
| **Embeddings** | `sentence-transformers/all-MiniLM-L6-v2` | 384-D semantic vector representations |
| **Vector Database** | `faiss-cpu` (`IndexFlatIP`) | Exact inner-product / cosine similarity search |
| **LLM Generator** | Google `gemini-3.5-flash-lite` (`google-genai`) | Context-grounded answer synthesis |

---

## ⚙️ Setup & How to Run

### 1. Prerequisites
* Python 3.10+ (or Google Colab)
* A free **Gemini API Key** from [Google AI Studio](https://aistudio.google.com/apikey)
* The source textbook PDF (`intro-to-ml-book.pdf`)

### 2. Install Dependencies
```bash
pip install pymupdf langchain-text-splitters tiktoken sentence-transformers faiss-cpu google-genai
```

### 3. Configure API Key
Set your Gemini API key in the environment or directly inside the notebook cell:
```python
import os

os.environ["GEMINI_API_KEY"] = "YOUR_GEMINI_API_KEY_HERE"
```

### 4. Execute the Notebook
1. Upload `intro-to-ml-book.pdf` when prompted by the file upload widget.
2. Run the preprocessing and chunking cell to build the 402-chunk corpus.
3. Run the embedding cell to populate the FAISS vector index.
4. Run the RAG demonstration cell to view automated test queries and interact with the real-time Q&A prompt (type `exit` to quit).

---

## 🧪 Evaluation & Grounding Highlights

* **Top-$k$ Retrieval Tuning**: Tested $k \in \{3, 5, 7\}$. **$k = 5$** provided the best balance between multi-page conceptual completeness and minimal context noise.
* **Citation Accuracy**: Every generated claim cites its source page (e.g., explaining $k$-NN voting from *Pages 35, 49, 50* and $k$-fold cross-validation from *Pages 266, 268*).
* **Hallucination Prevention**: When tested with out-of-domain queries (e.g., *"What is the recipe for baking a chocolate cake?"*), the assistant strictly refuses with:
  > *"I cannot find the answer to this question in the provided textbook excerpts."*
