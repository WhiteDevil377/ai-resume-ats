# AI Resume ATS System

A web app that compares a resume with a job description and returns an application-specific score and actionable feedback. Built with FastAPI + Streamlit, using spaCy and Sentence Transformers for NLP and the Groq API for structured resume and job-description parsing.

## What it does

1. Upload a resume (PDF / DOC / DOCX) and paste a job description.
2. The backend parses the resume, extracts skills and experience, and compares them to the JD using semantic similarity.
3. You get a score, a breakdown by category (formatting, keywords, content, skill validation, ATS compatibility), and rule-based suggestions for what to improve.
4. Past analyses are saved to your account so you can revisit them.

## Tech stack

- **Frontend:** Streamlit
- **Backend:** FastAPI (Python)
- **NLP:** spaCy (`en_core_web_md`), Sentence Transformers (`all-MiniLM-L6-v2`)
- **LLM parsing:** Groq API (`llama-3.3-70b-versatile` in the supplied code)
- **Auth + Database:** Supabase (email/password and Google OAuth)
- **PDF report export:** WeasyPrint + Jinja2

## Project structure

```
ai-resume-ats/
├── backend/              FastAPI app, NLP services, API routes
├── frontend/             Streamlit app, views, components
├── jupyter notebooks/    Research and dataset prep (not used at runtime)
├── requirements.txt      Combined backend + frontend dependencies
├── .env.example          Template for environment variables
└── .streamlit/           Optional frontend secrets example
```

## Setup

### 1. Clone and create a virtual environment

```bash
git clone https://github.com/WhiteDevil377/ai-resume-ats.git
cd ai-resume-ats
python3.12 -m venv .venv
source .venv/bin/activate         # Windows: .venv\Scripts\activate
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
python -m spacy download en_core_web_md
python -m spacy download en_core_web_sm
```

The smaller spaCy model is used as a fallback. WeasyPrint needs platform-specific native libraries; the supplied Linux setup commands are:

```bash
# Fedora
sudo dnf install -y cairo pango gdk-pixbuf2 libffi

# Debian / Ubuntu
sudo apt install -y libcairo2 libpango-1.0-0 libpangoft2-1.0-0 libffi-dev
```

### 3. Configure environment variables

Copy the template and fill in your keys:

```bash
cp .env.example .env
```

You need:

- A **Supabase** project — grab `SUPABASE_URL`, `SUPABASE_KEY` (service role), and `SUPABASE_ANON_KEY` from Project Settings → API.
- A **Groq** API key from [console.groq.com](https://console.groq.com).
- (Optional) Google OAuth set up in the Supabase dashboard if you want Google sign-in.

The backend verifies Supabase access tokens using your project's JWKS endpoint for ES256/RS256 tokens. Set `SUPABASE_JWT_SECRET` for projects issuing HS256 tokens. Keep `SUPABASE_KEY` and the JWT secret on the backend.

When launching Streamlit from the repository root, optional frontend settings belong in `.streamlit/secrets.toml`. Copy the included example if you need to configure a different backend URL or prefer Streamlit secrets for frontend authentication:

```bash
cp .streamlit/secrets.toml.example .streamlit/secrets.toml
```

Use only the anonymous key in the frontend secrets file.

### Analysis history table

The backend expects an `analyses` table in Supabase. The supplied project does not include a database migration. Create a table with a generated `id` and the fields used by `backend/database/supabase_db.py`: `user_id` (Supabase user UUID), `filename`, `ats_score`, `keyword_match`, `missing_keywords` (JSON array), `created_at` (timestamp), and `analysis_result` (JSON object). History writes and reads use the backend service-role key and filter by the verified user ID.

### 4. Run the backend

From the project root:

```bash
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

The API is now at `http://localhost:8000`.

### 5. Run the frontend

In a new terminal (with the venv activated):

```bash
streamlit run frontend/streamlit_app.py
```

The app opens at `http://localhost:8501`.

Use email/password registration or configure Google OAuth in your own Supabase project. Configure the allowed redirect URL to match `AUTH_REDIRECT_URL`.

## Notes

- **Never commit `.env` or `secrets.toml`** — they hold API keys. Both are in `.gitignore`; check before you push.
- The backend downloads the configured Sentence Transformer model on first use and caches it locally.
- A valid Groq key is required by the main analysis pipeline, including structured resume parsing.
- `jupyter notebooks/` contains research and dataset preparation work and is not required at runtime. Saved notebook outputs have been cleared for publication; datasets and exported model artifacts are not bundled.
- Resume analysis requires your own configured Supabase and Groq services. API credentials and uploaded resumes are excluded from the published repository.
