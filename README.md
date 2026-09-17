# AI Job Lister & Chatbot

An AI-powered resume parser and job recommendation system with a Retrieval-Augmented Generation (RAG) chatbot. Users upload a resume (PDF), the app extracts key information, fetches matching job listings, stores them as vector embeddings in Pinecone, and lets users query the job database conversationally via a Gemini-powered chatbot.

## Overview

The project combines several pieces into an end-to-end pipeline:

1. **Resume parsing** — extract text and key entities (skills, orgs, locations) from uploaded PDF resumes using `PyPDF2` and `spaCy`.
2. **Job fetching** — query the [Jooble API](https://jooble.org/api/about) for job listings matching the extracted resume content.
3. **Vector storage** — embed job listings with `sentence-transformers` and store/query them in a [Pinecone](https://www.pinecone.io/) vector index.
4. **Hybrid retrieval + RAG chat** — combine Pinecone semantic search with BM25 keyword search (via Reciprocal Rank Fusion) to retrieve relevant jobs, then use Google's Gemini API to generate a natural-language response summarizing them.
5. **Caching** — cache job recommendations in Redis to avoid redundant API calls for the same resume.
6. **OCR support** — optionally extract resume text from an image (e.g. `resume.jpg`) using `easyocr`.
7. **Streamlit UI** — a simple web front end for uploading resumes and browsing/querying job recommendations.

## Project Structure

| File | Description |
|---|---|
| `app.py` | Main Streamlit app: resume upload → extraction → job fetch (with Redis caching) → Pinecone storage → recommendations display. |
| `JOB.py` | Simpler/earlier standalone version: resume upload → spaCy extraction → direct Jooble job fetch and display (no Pinecone/RAG). |
| `pdf_parser.py` | PDF text extraction and resume entity extraction (`PyPDF2` + `spaCy`), with its own minimal Streamlit UI. |
| `image_parser.py` | OCR-based resume text extraction from an image file using `easyocr`. |
| `preprocess_jobs.py` | Fetches jobs from the Jooble API, cleans job text, and encodes job listings into embeddings. |
| `store_jobs_Pinecone.py` | Creates a Pinecone index (if needed) and uploads embedded job vectors to it. |
| `querying_jobs.py` | Standalone script to query the Pinecone index for jobs matching a text query. |
| `query_test.py` | Small utility script to print the total vector count stored in the Pinecone index. |
| `RAG.py` | Core RAG logic: hybrid retrieval (Pinecone + BM25 via Reciprocal Rank Fusion) and Gemini-based response generation. |
| `caching.py` | Variant of the app flow demonstrating Redis-based caching of job recommendations. |
| `jooble_api_tester.py` | Standalone script for testing the Jooble API directly. |
| `requirements.txt` | Python dependencies. |
| `resume.jpg` | Sample resume image for OCR testing. |
| `O/P/*.png` | Sample output screenshots of the application. |

## Requirements

- Python 3.10+ (recommended, for compatibility with `spacy`, `torch`, and `transformers` versions pinned in `requirements.txt`)
- Redis server (for caching in `app.py` / `caching.py`)
- Accounts/API keys for:
  - [Jooble API](https://jooble.org/api/about)
  - [Pinecone](https://www.pinecone.io/)
  - [Google Generative AI (Gemini)](https://ai.google.dev/)

### Install dependencies

```bash
pip install -r requirements.txt
python -m spacy download en_core_web_sm   # if not installed via the wheel URL in requirements.txt
```

### Start Redis (required for `app.py`)

```bash
redis-server
```

## ⚠️ Security Notice — Hardcoded API Keys

**This codebase contains hardcoded API keys and secrets committed directly in source files**, including `RAG.py`, `preprocess_jobs.py`, `querying_jobs.py`, `store_jobs_Pinecone.py`, and `query_test.py` (Pinecone, Gemini, and Jooble credentials). Before using or publishing this repository:

1. **Rotate/revoke all exposed keys immediately** in the Pinecone, Google AI Studio, and Jooble dashboards, since they are visible in the source code as committed.
2. **Move all credentials to environment variables** (e.g. via a `.env` file loaded with `python-dotenv`, or `os.environ`) rather than hardcoding them in source.
3. Add `.env` to `.gitignore` (currently only `venv/` and `__pycache__/` are ignored).

## Getting Started

> After rotating and externalizing the API keys as described above:

1. **Store job listings**: run `store_jobs_Pinecone.py` once to create the Pinecone index and populate it with job vectors (or let `app.py` populate it on first resume upload).
2. **Launch the app**:
   ```bash
   streamlit run app.py
   ```
3. Upload one or more PDF resumes. The app will:
   - Extract resume content
   - Fetch and cache matching job listings
   - Store job embeddings in Pinecone
   - Display the top 5 recommended jobs

For the simpler, non-RAG version, run:
```bash
streamlit run JOB.py
```

## Notes

- `RAG.py`'s LLaMA-2-based generation path is present but commented out; the active implementation uses Google's Gemini (`gemini-1.5-pro`) for response generation.
- Several scripts (`querying_jobs.py`, `query_test.py`, `jooble_api_tester.py`) are standalone testing/debugging utilities rather than part of the main app flow.
