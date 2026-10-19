# JobPilot — AI-Powered Job Recommendation System

JobPilot is a prototype job recommendation and resume-tailoring application built with Python, SentenceTransformers, FAISS, Streamlit, and Google Gemini. It matches a candidate's resume and preferences against a dataset of approximately 25,000 historical job postings, then filters and ranks the results.

The project demonstrates data ingestion and deduplication, text embedding generation, vector similarity search, multi-factor ranking, feedback-based re-ranking, and an interactive application.

> **Project scope:** JobPilot uses a historical job-posting dataset, primarily from 2021. Recommendations are demonstrations of matching logic, **not verified current openings**. The application is a portfolio prototype rather than a production job-search service.

## Features

- **Resume intake:** Upload a PDF resume and extract its text and recognized skills.
- **Semantic job retrieval:** Encode a candidate profile with `all-MiniLM-L6-v2` and search a FAISS index for similar job postings.
- **Filtering and ranking:** Exclude certain seniority and contract-related matches; rank remaining jobs by semantic similarity (50%), skill overlap (25%), target-role match (15%), and location match (10%).
- **Explanations:** Display matched skills, similarity scores, and ranking signals for recommended jobs.
- **Feedback-based re-ranking:** Adjust ranking weights in the current session based on Accept, Reject, or Skip feedback.
- **Resume-tailoring draft:** Optionally use Gemini 2.5 Flash to generate a draft tailored to a selected posting. This feature requires an API key.
- **Batch analytics:** Summarize skills, companies, locations, titles, and salary-data coverage in the job dataset.

## How It Works

```text
Historical job postings
       |
Ingestion, standardization, deduplication
       |
Sentence-transformer job embeddings
       |
FAISS vector index + processed job data
       |
       +-----------------------+
                               |
Uploaded PDF resume -> text and skill extraction
                               |
                      Candidate profile embedding
                               |
                       FAISS top-100 retrieval
                               |
                        Hard filtering
                               |
                   Multi-factor re-ranking
                               |
                Recommendations + explanations
                               |
                 Optional feedback / Gemini draft
```

The development notebook documents the ingestion, embedding, indexing, evaluation, and feedback-simulation workflow. The Streamlit application loads **precomputed deployment artifacts**, so it does not need to regenerate embeddings when launched.

## Technology Stack

| Area | Technologies |
| --- | --- |
| Data processing | Python, pandas, NumPy |
| NLP and embeddings | SentenceTransformers (`all-MiniLM-L6-v2`) |
| Vector search | FAISS |
| Application | Streamlit |
| PDF parsing | pypdf |
| Generative AI | Google Gemini 2.5 Flash (optional) |
| Data ingestion extension | Adzuna API (optional notebook workflow) |

## Repository Structure

```text
jobpilot-ai-recommender/
├── app.py
├── job_recommendation_pipeline.ipynb
├── README.md
├── requirements.txt
├── .gitignore
├── artifacts/
│   ├── job_embeddings_deploy.npy
│   └── job_faiss_deploy.index
└── data/
    ├── jobpilot_jobs_with_embedding_text_deploy.csv
    └── sample_resume.pdf
```

This structure describes the intended cleaned portfolio repository. Additional files may be present until cleanup is complete. The synthetic `sample_resume.pdf` is used by the notebook; the Streamlit app accepts user-uploaded PDFs. The notebook references additional raw inputs and intermediate files that are intentionally excluded from this repository.

## Run the Streamlit Application

**Requirements:** Python and the dependencies listed in `requirements.txt`.

1. Clone or download this repository.
2. From the project root, install dependencies:

   ```bash
   python3 -m pip install -r requirements.txt
   ```

3. Start the application:

   ```bash
   python3 -m streamlit run app.py
   ```

4. Upload a PDF resume, select target roles and preferred locations, and choose **Find Matching Jobs**.

The application reads these files at startup:

- `data/jobpilot_jobs_with_embedding_text_deploy.csv`
- `artifacts/job_faiss_deploy.index`

Keep the repository's relative folder structure intact. The recommendation and analytics features do not require a Gemini API key.

### Optional API Credentials

To enable Gemini resume drafting, create a local `file.env` in the project root:

```dotenv
GEMINI_API_KEY=your_key_here
```

The notebook's optional Adzuna ingestion section also reads `ADZUNA_APP_ID` and `ADZUNA_APP_KEY` from the environment. **Never commit real API credentials.** `file.env` is excluded by `.gitignore`.

## Development Notebook and Reproducibility

`job_recommendation_pipeline.ipynb` documents the experimental workflow used to develop the recommendation system, including:

1. Reading and deduplicating job postings.
2. Extracting and representing resume information.
3. Generating sentence-transformer embeddings.
4. Building and saving a FAISS index.
5. Evaluating ranking and simulating feedback adjustments.
6. Demonstrating optional live-job ingestion through Adzuna.

**Important:** The notebook's original ingestion step expects `data/techmap_jobs_sample_30000.jsonl` (approximately 382 MB), which is **not included in this repository**. It also generates intermediate files such as `data/jobpilot_jobs_snapshot.csv`, `data/jobpilot_jobs_with_embedding_text.csv`, `artifacts/job_embeddings.npy`, and `artifacts/job_faiss.index`. These development files are distinct from the `_deploy` files used by `app.py` and are intentionally excluded. The notebook therefore documents the development pipeline but **cannot be run end-to-end from a fresh clone without the original input data**. The included deployment files are intended to support the Streamlit application independently.

## Evaluation Results

The development work included exploratory evaluation of retrieval and feedback-based re-ranking:

- **Top-10 relevance:** The project technical brief reports approximately **80% relevance** among the top ten retrieved jobs based on manual review. This was a small, informal assessment, not a benchmark against a published labeled test set.
- **Feedback simulation:** In the notebook's simulated feedback experiment, the reported average rank of accepted jobs changed from **5.75 to 5.00** after two feedback iterations (approximately **13% lower**, where a lower rank number is better). This measures behavior in a simulation, **not** verified improvement with real users.

These results are exploratory and should not be interpreted as production-level recommendation accuracy.

## Data and Limitations

- The deployed dataset contains approximately **25,000 historical job postings**; these should not be treated as current openings.
- Job recommendations use heuristic filtering and weighted scoring. They do not guarantee job eligibility or suitability.
- Skill extraction relies on keyword matching, which may miss synonyms or context.
- Salary fields are incomplete in the underlying dataset.
- Feedback adjustments are session-based rather than a persistent learned model.
- Gemini-generated resume text should be reviewed for accuracy before use.
- The optional Adzuna notebook workflow does **not** make the deployed app a live-updating job board.

## Possible Improvements

- Refresh postings through a supported jobs API and track freshness/expiration.
- Automate incremental ingestion, deduplication, and index updates.
- Add systematic retrieval and ranking evaluation against labeled relevance judgments.
- Improve skill extraction and candidate seniority classification.
- Add tests, configuration validation, and reproducible data-pipeline setup.

## Project Context

Originally developed as a graduate business analytics project. This repository presents the technical implementation as a portfolio prototype.
