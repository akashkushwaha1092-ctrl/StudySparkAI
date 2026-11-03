# StudySpark AI

> Learn Smarter, Not Harder.

StudySpark AI is a student-focused AI study companion built with a Vite + React frontend, a FastAPI backend, and Gemma 4 via the Gemini API. It helps students turn a topic, a set of class notes, or a deadline into a clear explanation, a summary, a quiz, a study plan, or a personalized adaptive practice session.

## Why this project

StudySpark AI is designed to make studying more actionable without login flows, a database, or heavy infrastructure. The app keeps results in the browser with `localStorage`, uses the backend only for model-driven generation, and stays easy to demo locally.

## Feature highlights

- Explain a Topic — clear explanations tailored to beginner, intermediate, or advanced learners.
- Summarize Notes — turn notes into a concise summary with key points and important terms.
- Generate Quiz — create a five-question MCQ quiz with answer explanations.
- Generate Study Plan — build a daily study plan based on subjects, time, and deadlines.
- Adaptive Quiz — the flagship feature. It creates a personalized quiz session that changes difficulty after each answer, revisits weak concepts, and ends with a session summary instead of a fixed set of questions.

### Adaptive Quiz (featured)

The Adaptive Quiz is the smart revision loop in the app:

- Starts from the selected difficulty and topic or notes.
- Raises difficulty after correct answers and lowers it after mistakes.
- Reuses targeted questions to reinforce weak concepts.
- Tracks score, difficulty history, weak concepts, and completion reason.
- Saves only a concise summary to history rather than raw question content or answers.

This makes it ideal for self-testing and revision without forcing a one-size-fits-all quiz flow.

## Screenshots

| Desktop Dashboard | Mobile View |
|---|---|
| ![Desktop Dashboard](docs/assets/screenshots/01-home.png) | ![Mobile View](docs/assets/screenshots/03-mobile.png) |

## Tech stack

- Frontend: React, Vite, Tailwind CSS, React Router, and lucide-react
- Backend: FastAPI, Pydantic v2, and slowapi
- AI: Gemma 4 through Google’s `google-genai` SDK
- Storage: browser `localStorage` only; no accounts and no application database

## Architecture

```text
Browser (Vite + React)
  ├─ UI, theme, language, and local history
  └─ Calls the backend through frontend/src/services/api.js
          │
          ▼
FastAPI backend
  ├─ request validation
  ├─ rate limiting and CORS
  ├─ prompt assembly
  ├─ provider retries and error handling
  └─ output validation and response envelopes
          │
          ▼
Gemini API / Gemma 4
```

## Local setup

Use two terminals from the repository root.

### 1) Start the backend

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
```

Set `GEMINI_API_KEY` in `backend\.env`, then run:

```powershell
uvicorn main:app --reload --port 8000
```

Open the health check at http://localhost:8000/api/health and the API docs at http://localhost:8000/docs.

### 2) Start the frontend

```powershell
cd frontend
npm install
Copy-Item .env.example .env
npm run dev
```

Open http://localhost:5173.

The default frontend env template uses mock data (`VITE_USE_MOCKS=true`), which is useful for UI demos and screenshots. To use live AI responses, set `VITE_USE_MOCKS=false` in `frontend/.env` and ensure `backend/.env` contains a valid `GEMINI_API_KEY`.

## Environment variables

### Backend (`backend/.env`)

| Variable | Default | Description |
|---|---|---|
| `GEMINI_API_KEY` | empty | Gemini API key for live responses |
| `GEMMA_MODEL` | `gemma-4-26b-a4b-it` | Gemma model ID |
| `INFERENCE_PROVIDER` | `gemini_api` | AI provider |
| `ALLOWED_ORIGINS` | `http://localhost:5173` | Frontend origins allowed by CORS |
| `RATE_LIMIT` | `20/minute` | Request limit per client |
| `MODEL_TIMEOUT_SECONDS` | `30` | Model call timeout |
| `LOG_LEVEL` | `INFO` | Backend log level |
| `APP_ENV` | `development` | Development or production mode |

### Frontend (`frontend/.env`)

| Variable | Default | Description |
|---|---|---|
| `VITE_API_BASE_URL` | `http://localhost:8000` | Backend URL |
| `VITE_USE_MOCKS` | `true` | Toggle mock mode for the UI |

Important: keep `GEMINI_API_KEY` only in `backend/.env`. Never expose it in a frontend variable or source file.

## API overview

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/api/health` | Check backend and AI status |
| `POST` | `/api/explain` | Explain a topic |
| `POST` | `/api/summarize` | Summarize notes |
| `POST` | `/api/quiz` | Generate a fixed five-question quiz |
| `POST` | `/api/study-plan` | Generate a study plan |
| `POST` | `/api/adaptive-quiz/start` | Start an adaptive quiz session |
| `POST` | `/api/adaptive-quiz/answer` | Submit an answer and continue the adaptive flow |

All AI responses follow the project envelope format: `{ ok, data, meta }` on success and `{ ok: false, error }` on failure.

## Project layout

```text
StudySparkAI/
├── backend/
│   ├── main.py
│   ├── schemas.py
│   ├── prompts.py
│   ├── gemma_service.py
│   ├── requirements.txt
│   └── .env.example
├── frontend/
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── .env.example
├── docs/
│   └── spec/
├── README.md
├── .gitignore
└── package-lock.json
```

## Development notes

- The frontend talks to the backend only through `frontend/src/services/api.js`.
- Model output is validated and parsed before it is returned to the UI.
- The app supports both English and Hindi responses, plus light and dark themes.
- Mock mode is available for frontend-only demos and screenshot work.

## Verification

Run backend tests from the `backend` directory:

```powershell
cd backend
python -m pytest
```

Build the frontend from the `frontend` directory:

```powershell
cd frontend
npm run build
```

This project is a local MVP intended for demo and validation, and it should be reviewed before production deployment.
