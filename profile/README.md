# Vita Care Hospital

**Book appointments. Chat with an assistant grounded in hospital docs. See how LLM apps are wired in production—not just an API wrapper.**

[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=white)](https://github.com/Vita-Care-Hospital/hospital-ui)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)](https://github.com/Vita-Care-Hospital/hospital-api)
[![Qdrant](https://img.shields.io/badge/RAG-Qdrant-DC244C)](https://github.com/Vita-Care-Hospital/chatbot-ai)
[![SQLite](https://img.shields.io/badge/Bookings-SQLite-003B57?logo=sqlite&logoColor=white)](https://github.com/Vita-Care-Hospital/hospital-api)

Three services · tool-calling agent · RAG over your KB · guardrails for bookings & slot times

## Repositories

| Repo | Role |
| --- | --- |
| [**hospital-ui**](https://github.com/Vita-Care-Hospital/hospital-ui) | React + Vite SPA, portfolio capture scripts |
| [**hospital-api**](https://github.com/Vita-Care-Hospital/hospital-api) | FastAPI booking JSON + SQLite |
| [**chatbot-ai**](https://github.com/Vita-Care-Hospital/chatbot-ai) | LLM chat, tools, RAG, admin ingest |
| [**.github**](https://github.com/Vita-Care-Hospital/.github) | Org profile, architecture & design docs |

---

## Demo

Full walkthrough: title slide → home → manual booking → chat (general + RAG + guided book) → admin KB ingest.

<video src="https://github.com/Vita-Care-Hospital/hospital-ui/raw/main/docs/media/vitacare-demo.mp4" controls playsinline width="100%"></video>

**[▶ Open narrated demo (MP4)](https://github.com/Vita-Care-Hospital/hospital-ui/blob/main/docs/media/vitacare-demo.mp4)** · [Silent WebM](https://github.com/Vita-Care-Hospital/hospital-ui/blob/main/docs/media/vitacare-demo.webm)

---

## Screenshots

| | |
|:---:|:---:|
| **Home** — departments from the booking API | **Book** — manual slot flow on `/book` |
| ![Home](docs/screenshots/home.png) | ![Book appointment](docs/screenshots/book-appointment.png) |
| **Assistant** — chat + guided cards | **Admin** — bookings & knowledge-base ingest |
| ![Chatbot](docs/screenshots/chatbot.png) | ![Admin](docs/screenshots/admin.png) |

---

## What this project is

**Vita Care Hospital** is a portfolio-grade demo: a React SPA talks to a **booking API** (SQLite) and a **chatbot API** (Groq or Ollama + Qdrant). Patients book without the LLM on the happy path; the assistant can book via tools, search ingested FAQs, and fall back safely when docs are empty.

| You get | How |
| --- | --- |
| Guided booking | Card UI + optional LLM tool calls to the same JSON API |
| Hospital Q&A | RAG (`search_hospital_docs`) with no invented hospital facts |
| KB admin | Upload `.txt` / `.md` / `.pdf` / `.docx` → chunk → embed → Qdrant |
| Safer replies | Emergency gate, booking-truth guard, slot-time checks |

**Stack:** React + Vite · FastAPI ×2 · SQLite · Qdrant · Ollama embeddings · Groq or Ollama chat.

---

## Architecture

```text
Browser  →  hospital-ui (:5173)
              ├─ /api/*  →  hospital-api (:8001)  SQLite
              └─ /ai/*   →  chatbot-ai (:8000)    tools + RAG → Qdrant, Groq/Ollama
```

```mermaid
flowchart LR
  UI["hospital-ui"] --> API["hospital-api"]
  UI --> AI["chatbot-ai"]
  AI --> API
  AI --> Q["Qdrant"]
  AI -.-> L["Groq / Ollama"]
  API --> DB["SQLite"]
```

Diagrams & trade-offs: [architecture.md](docs/architecture.md) · [design-decisions.md](docs/design-decisions.md)

---

## Run locally

Clone the three app repos (sibling folders), then start Qdrant and each service:

```powershell
git clone https://github.com/Vita-Care-Hospital/hospital-ui.git
git clone https://github.com/Vita-Care-Hospital/hospital-api.git
git clone https://github.com/Vita-Care-Hospital/chatbot-ai.git

docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant

# Terminal 2 — booking API (:8001)
cd hospital-api
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
python -m uvicorn app.main:app --reload --port 8001

# Terminal 3 — chatbot (:8000)
cd ..\chatbot-ai
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
python -m uvicorn app.main:app --reload --port 8000

# Terminal 4 — UI (:5173)
cd ..\hospital-ui
npm install
npm run dev
```

Open **http://127.0.0.1:5173** — chat FAB bottom-right.

| Service | Repository |
| --- | --- |
| UI | [Vita-Care-Hospital/hospital-ui](https://github.com/Vita-Care-Hospital/hospital-ui) |
| Booking API | [Vita-Care-Hospital/hospital-api](https://github.com/Vita-Care-Hospital/hospital-api) |
| Chat + RAG | [Vita-Care-Hospital/chatbot-ai](https://github.com/Vita-Care-Hospital/chatbot-ai) |

---

## Highlights (for reviewers)

- **Hand-written agent loop** — LangChain only for loaders / split / Qdrant; orchestration stays readable in `chat_service.run_chat`.
- **Guardrails** — no “booked” without a tool `id`; slot times checked against `list_slots`; malformed tool calls recovered.
- **RAG policy** — Vita Care facts only from snippets; general medical answers prefixed when KB is empty.
- **Idempotent ingest** — re-upload same filename replaces chunks, no duplicates.

---

Demo-grade, single-machine — useful as a reference for grounded LLM apps and a base for auth, observability, and deployment.
