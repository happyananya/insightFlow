# InsightFlow

InsightFlow is a full-stack app that converts natural-language questions into SQL using your database schema as context.

- **Frontend:** React + Vite UI (`frontend/`)
- **Backend:** FastAPI service with schema retrieval + semantic table matching (`backend/`)
- **Infra:** Docker Compose support for backend runtime (`docker-compose.yml`)

---

## How It Works

1. You type a question in natural language in the frontend.
2. The frontend sends it to `POST /getSql`.
3. The backend:
   - loads table metadata from Supabase/Postgres (or falls back to dummy metadata),
   - ranks relevant tables with embeddings + FAISS,
   - prompts Gemini to generate SQL.
4. The generated SQL is returned and shown in the UI.

---

## Project Structure

```text
insightFlow/
├── frontend/              # React + Vite app
├── backend/               # FastAPI app
│   ├── app/main.py        # API entrypoint and SQL generation flow
│   ├── app/db.py          # Supabase/Postgres schema metadata loader
│   ├── .env.example       # Required backend env var template
│   └── Dockerfile         # Backend container image
└── docker-compose.yml     # Local Docker orchestration (backend)
```

---

## Prerequisites

Install the following on your machine:

- **Node.js** (recommended: current LTS)
- **npm** (comes with Node.js)
- **Python 3.12+**
- **pip**
- **Docker Desktop** (only required for Docker-based backend run)

You also need:

- A reachable **Postgres/Supabase connection string** for `SUPABASE_DB_URL`

---

## Environment Setup

From the repo root:

```bash
cp backend/.env.example backend/.env
```

Edit `backend/.env` and set:

```env
SUPABASE_DB_URL=postgresql://postgres:password@db.<project-ref>.supabase.co:5432/postgres
```

Notes:

- Backend checks `SUPABASE_DB_URL` first, then `DATABASE_URL`.
- If neither is set, the app uses built-in dummy metadata so the API can still run.

---

## Run Locally (Recommended: Backend in Docker + Frontend on Host)

### 1) Start backend with Docker

From repo root:

```bash
docker compose up --build backend
```

Backend URL: `http://localhost:8000`

### 2) Start frontend

In a new terminal:

```bash
cd frontend
npm install
npm run dev
```

Frontend URL: `http://localhost:5173`

Vite is configured to proxy `/getSql` requests to `http://localhost:8000`, so no extra frontend env var is required for local dev.

### 3) Stop services

- Stop frontend with `Ctrl+C`
- Stop backend with `Ctrl+C`, then optionally:

```bash
docker compose down
```

---

## Run Fully Local (No Docker)

### 1) Start backend natively

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
export SUPABASE_DB_URL="postgresql://postgres:password@db.<project-ref>.supabase.co:5432/postgres"
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### 2) Start frontend

In another terminal:

```bash
cd frontend
npm install
npm run dev
```

---

## API Endpoints

- `POST /getSql`
  - Request body:
    ```json
    {
      "query": "Show weekly revenue for the last 6 weeks",
      "top_k": 2
    }
    ```
  - Response: plain text SQL string
- `GET /debug/metadata`
  - Returns currently loaded table metadata

---

## Troubleshooting

- **Frontend loads but SQL generation fails**
  - Confirm backend is running on `http://localhost:8000`.
  - Check backend logs for request/runtime errors.

- **Database metadata not loading**
  - Verify `SUPABASE_DB_URL` is valid and reachable from your machine/container.
  - Confirm the database user has permissions to read schema metadata.

- **First backend start is slow**
  - The sentence-transformer model may download on first run.
  - This is expected; subsequent starts are faster once cached.

- **Port conflicts**
  - Backend expects `8000`, frontend expects `5173`.
  - Free these ports or update config accordingly.

---

## Development Notes

- Frontend dev server script: `npm run dev`
- Frontend production build: `npm run build`
- Frontend preview build: `npm run preview`
- Backend default launch command (Docker): `uvicorn main:app --host 0.0.0.0 --port 8000`

---

## Quick Start

If you just want to run it quickly:

```bash
# from repo root
cp backend/.env.example backend/.env
# edit backend/.env and set SUPABASE_DB_URL
docker compose up --build backend

# new terminal
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173`.
