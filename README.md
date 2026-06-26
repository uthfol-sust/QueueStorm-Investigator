# QueueStorm Investigator

**AI-powered support copilot for digital finance complaints.**  
QueueStorm Investigator reads support tickets alongside transaction history, evaluates evidence, routes cases to the right team, and generates safe customer replies — all in under 30 seconds.

Built for the bKash SUST Hackathon 2026.

---

## Features

- **Ticket analysis** — Submit a complaint with transaction history; get structured case routing, evidence verdict, severity, and confidence score
- **AI-generated text** — Three natural-language fields: agent summary, recommended next action, and a customer-safe reply validated against safety rules
- **Safety guardrails** — Built-in checks prevent credential requests, unauthorized refund promises, and prompt injection
- **Analysis history** — Browse, filter, and review past analyses with stats aggregated by case type, severity, department, and verdict
- **JSON Studio** — Raw API testing mode for direct payload experimentation
- **Health check** — `GET /health` endpoint for monitoring

---

## Architecture

```
┌──────────────────┐     POST /analyze-ticket      ┌──────────────────┐
│                  │  ──────────────────────────►  │                  │
│  Streamlit UI    │                               │  FastAPI Backend │
│  (frontend/)     │  ◄──────────────────────────  │  (backend/)      │
│                  │     TicketResponse            │                  │
└──────────────────┘                               └────────┬─────────┘
                                                            │
                                            ┌───────────────┴───────────────┐
                                            │                               │
                                    ┌───────┴───────┐             ┌─────────┴────────┐
                                    │  Rule Engine  │             │  LLM (Gemini /   │
                                    │  (engine.py)  │             │  Groq) (llm.py)  │
                                    └───────────────┘             └──────────────────┘
```

### Backend (FastAPI)

| Component      | File              | Role                                      |
|----------------|-------------------|-------------------------------------------|
| Routes         | `backend/main.py` | FastAPI app, exception handlers, endpoints |
| Models         | `backend/models.py` | Pydantic request/response schemas        |
| Rule engine    | `backend/engine.py` | Structured field logic (case type, verdict, severity, etc.) |
| LLM integration| `backend/llm.py`  | Generates agent summary, next action, customer reply |
| Safety checks  | `backend/safety.py` | Post-processing guardrails on text fields |
| Config         | `backend/config.py` | Environment variable loading              |
| History store  | `backend/history.py` | In-memory analysis history with filtering |

### Frontend (Streamlit)

| Component         | File                     | Role                        |
|-------------------|--------------------------|-----------------------------|
| App               | `frontend/app.py`        | UI pages, forms, rendering  |
| Streamlit config  | `frontend/.streamlit/config.toml` | Theme settings       |

---

## API Endpoints

| Method | Path                | Description                        |
|--------|---------------------|------------------------------------|
| GET    | `/health`           | Backend readiness check            |
| POST   | `/analyze-ticket`   | Submit ticket for full analysis    |
| GET    | `/history`          | List past analyses (filterable)    |
| GET    | `/history/stats`    | Aggregated stats by type/severity  |
| GET    | `/history/{id}`     | Single history entry               |
| DELETE | `/history/{id}`     | Delete a history entry             |

---

## Prerequisites

- Python 3.11+
- A Gemini API key (or Groq API key if using Groq models)
- `uv` or `pip` for dependency management

---

## Local Development

### 1. Clone and set up

```bash
git clone <repo-url>
cd queuestorm-investigator
```

### 2. Environment variables

```bash
cp .env.example .env
```

Edit `.env`:

```env
GEMINI_API_KEY=your_gemini_api_key_here
MODEL_NAME=gemini-1.5-flash
PORT=8000
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Start the backend

```bash
uvicorn backend.main:app --reload --port 8000
```

### 5. Start the frontend

```bash
streamlit run frontend/app.py
```

Open http://localhost:8501 in your browser.

---

## Deployment

### Deployed backend on Render
> **Backend Live URL:** `https://<your-render-service>.onrender.com`  

### Deployed frontend on Streamlit Cloud


> **Frontend Live URL:** `https://queuestorm.streamlit.app/`  


## Environment Variables

| Variable            | Required | Default            | Description                          |
|---------------------|----------|---------------------|--------------------------------------|
| `GEMINI_API_KEY`    | No*     | —                   | Google Gemini API key                |
| `GROQ_API_KEY`      | yes*      | —                   | Groq API key (if using Groq models)  |
| `MODEL_NAME`        | No       | `gemini-1.5-flash`  | LLM model identifier                 |
| `PORT`              | No       | `8000`              | Backend server port                  |
| `QUEUESTORM_API_URL`| No       | `http://127.0.0.1:8000` | Backend URL for the frontend    |

\* At least one LLM provider key is required.

---

## Project Structure

```
queuestorm-investigator/
├── backend/
│   ├── __init__.py
│   ├── config.py            # Environment config
│   ├── engine.py            # Rule engine logic
│   ├── history.py           # Analysis history store
│   ├── llm.py               # LLM text generation
│   ├── main.py              # FastAPI app + routes
│   ├── models.py            # Pydantic schemas
│   └── safety.py            # Text safety validators
├── frontend/
│   ├── .streamlit/
│   │   └── config.toml      # Streamlit theme config
│   └── app.py               # Streamlit UI
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```

---

