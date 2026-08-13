# Velora — AI-Moderated Social Platform

A social platform where every post is screened by AI **before** it goes live. Text, images, and documents (PDF/DOCX/TXT) are all analyzed by Google Gemini for harmful content — hate speech, violence, harassment, terrorism, self-harm, sexual content, child safety, illegal activity, prompt injection, and jailbreak attempts — and scored for risk before publishing is allowed.

Built as a local/LAN full-stack app: FastAPI + SQLite backend, React + TypeScript + Tailwind frontend.

## How it works

```
User submits post (text / image / document)
        ↓
File validation (type, size, format)
        ↓
Content extraction (text extraction from PDF/DOCX/TXT)
        ↓
Gemini analysis (text or vision, JSON-only response)
        ↓
Risk scoring + category classification
        ↓
Post allowed (safe) or blocked (harmful / critical) + result stored
        ↓
Scan result + audit log persisted to SQLite
```

If Gemini flags content as `harmful` or `critical`, the post is rejected with the reason returned to the user and the uploaded file is deleted — nothing harmful is ever persisted to the feed.

## Features

- **Pre-publish moderation** — every post (text, image, or document) is scanned by Gemini before it's added to the feed.
- **Multi-format support** — plain text, JPG/JPEG/PNG images (via Gemini vision), and PDF/DOCX/TXT documents (text extracted before analysis).
- **Standalone moderation tools** — dedicated text, image, and document moderation pages independent of posting, for quick one-off scans.
- **Risk scoring** — every scan returns a 0–100 risk score, a status (`safe` / `warning` / `harmful` / `critical`), matched categories, a reason, and a confidence score.
- **Scan history & audit log** — every moderation call and outcome is recorded to SQLite for later review.
- **User accounts** — JWT-based auth (register/login) gates posting; the feed itself is viewable without an account.
- **Resilient AI calls** — retries with exponential backoff, timeout handling, and defensive JSON parsing around the Gemini API.
- **Prompt-injection resistant by design** — the moderation system prompt explicitly instructs Gemini to never follow instructions embedded in the content it's analyzing.

## Tech stack

| Layer | Tech |
|---|---|
| Frontend | React 19 + TypeScript, Vite, Tailwind CSS, React Router |
| Backend | FastAPI, SQLAlchemy, Pydantic / pydantic-settings |
| AI | Google Gemini (`gemini-2.5-flash`) — text + vision |
| Document parsing | PyPDF2, python-docx |
| Auth | JWT (`python-jose`) + `passlib` (bcrypt) |
| Database | SQLite |

## Project structure

```
backend/
  app/
    main.py                 # FastAPI app entrypoint
    config.py                # env-driven settings (Gemini key, JWT secret, paths)
    database.py               # SQLAlchemy engine/session setup
    api/
      auth.py                  # register / login
      posts.py                 # create/list/get posts (moderation gated)
      text.py                  # standalone text moderation endpoint
      image.py                 # standalone image moderation endpoint
      document.py              # standalone document moderation endpoint
      history.py                # scan history endpoint
    services/
      moderation.py             # orchestrates moderation + persistence
      gemini_client.py           # Gemini API wrapper (retry, timeout, JSON parsing)
      file_processing.py         # file validation + text extraction
      auth_service.py            # JWT issue/verify, password hashing
    models/                   # SQLAlchemy models (user, post, scan_result, audit_log)
    schemas/                  # Pydantic request/response models
    prompts/                  # moderation system prompt
  requirements.txt

frontend/
  src/
    pages/                   # Feed, CreatePost, Dashboard, Analytics, History,
                               # TextModeration, ImageModeration, DocumentModeration,
                               # Login, Register, Profile, ModerationFeed
    components/
    contexts/                # auth context
    api/                     # API client wrappers

db/
  schema.sql                 # reference schema
  init.sql                   # reference init script
```

## Getting started

### 1. Get a Gemini API key
Grab one from [Google AI Studio](https://aistudio.google.com/apikey).

### 2. Backend

```bash
cd backend
python -m venv .venv

# Windows
.venv\Scripts\Activate.ps1
# macOS/Linux
source .venv/bin/activate

pip install -r requirements.txt
```

Copy `.env.example` to `.env` in the **project root** (not inside `backend/`) and fill in your key:

```bash
cp .env.example .env
```

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

The app is designed to run entirely on your local machine or LAN — no user data or content ever leaves your network except the specific content sent to the Gemini API for analysis.

## Security notes

- `GEMINI_API_KEY` and `JWT_SECRET_KEY` are read from environment variables — never commit a real `.env` file (the included `.gitignore` already excludes it).
- Blocked/flagged uploads are deleted immediately rather than retained.
- The moderation prompt explicitly guards against prompt injection: content submitted for moderation cannot instruct Gemini to change its own behavior.
- This is a local/LAN-oriented app and does not currently include hardening for public internet deployment (rate limiting, HTTPS enforcement, etc.).

## License

MIT — see [LICENSE](LICENSE) (add one before publishing if you want an explicit license).
