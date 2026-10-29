# AICTE Security Deposit / FDR Management Portal

An evidence-driven prototype and working FastAPI backend for managing AICTE
security deposits, fixed-deposit receipts (FDRs), payment claims, bank
statement reconciliation, document checks, release/refund requests, reports,
notifications, and audit records.

This repository is also prepared for a project presentation. The
[presentation fact sheet](docs/SLIDE_FACTS.md) is the source of truth for
slide claims, implementation status, technical details, and a six-slide
presentation outline.

> **Important:** This is a prototype/demo and is not an official AICTE
> service. Do not enter real credentials, personal information, institutional
> records, or financial data. The local demo is not approved for production
> financial, identity, or document processing.

## Contents

- [What the project does](#what-the-project-does)
- [Technology stack](#technology-stack)
- [Run locally](#run-locally)
- [Run the complete stack with Docker](#run-the-complete-stack-with-docker)
- [Test the backend](#test-the-backend)
- [Explore the application](#explore-the-application)
- [Create the project PPT](#create-the-project-ppt)
- [Repository structure](#repository-structure)
- [Current status and limitations](#current-status-and-limitations)
- [Production-readiness checklist](#production-readiness-checklist)

## What the project does

The portal provides separate institution and officer experiences around a
shared security-deposit workflow:

1. An institution submits an FDR/RTGS instrument, payment claim, and supporting
   documents.
2. The system imports or reads bank-statement data and attempts to match a
   claim using UTR, amount, date, and narration signals.
3. Document processing extracts fields, checks dates and file properties, and
   reports discrepancies for review.
4. An authorised officer reviews the evidence and makes the workflow decision.
5. Approved release/refund requests, deductions, notifications, reports, and
   audit events are recorded by the backend.

The system deliberately separates automated signals from human decisions:
matching and document checks can flag or route work, but they do not prove
fraud or replace authorised officer approval.

## Technology stack

| Layer | Technologies |
| --- | --- |
| Frontend | React 18, React Router, Vite, JavaScript/JSX, plain CSS, GSAP |
| Backend | Python 3.10+, FastAPI, Uvicorn, Pydantic |
| Persistence | SQLAlchemy 2, SQLite for local development, PostgreSQL 16 for Docker |
| Security | JWT access tokens, bcrypt password verification, role/region scoping |
| Processing | PyPDF2, pandas, openpyxl, ZIP/XML inspection, SHA-256 file hashes |
| Matching | RapidFuzz plus exact UTR, amount, and date-window checks |
| Testing | pytest and FastAPI `TestClient` |

## Run locally

### Prerequisites

- Node.js and npm compatible with the installed Vite version
- Python 3.10 or newer
- PowerShell on Windows, or an equivalent shell on another platform

### 1. Start the backend

From the repository root:

```powershell
pip install -r backend/requirements.txt
$env:PYTHONPATH="backend;."
python backend/run.py
```

The API is available at:

- API base URL: <http://127.0.0.1:8000>
- Swagger UI: <http://127.0.0.1:8000/docs>
- ReDoc: <http://127.0.0.1:8000/redoc>
- Health check: <http://127.0.0.1:8000/health>

The default local configuration uses SQLite. Set `DATABASE_URL` when using a
different database.

### 2. Start the frontend

In a second terminal:

```powershell
npm install
npm run dev
```

Open the Vite URL, normally <http://localhost:5173>.

Available npm commands:

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start the Vite development server |
| `npm run build` | Create the production frontend bundle in `dist/` |
| `npm run preview` | Preview the production bundle locally |

### Environment variables

The frontend uses the deployed API by default. To use a different backend,
set `VITE_API_URL` to the backend origin without a trailing `/api`, then
restart the Vite server or rebuild the frontend:

```powershell
$env:VITE_API_URL="http://127.0.0.1:8000"
npm run dev
```

For a local backend, set at least a strong `SECRET_KEY` before starting the
API. For Docker Compose, copy `.env.example` to `.env` and replace both
placeholder values:

```powershell
Copy-Item .env.example .env
```

Never commit `.env`, passwords, signing keys, API keys, or real uploaded
documents.

## Run the complete stack with Docker

Docker Compose starts PostgreSQL, the FastAPI API, the React frontend, and
Mailpit:

```powershell
Copy-Item .env.example .env
# Edit .env and set unique values for POSTGRES_PASSWORD and SECRET_KEY.
docker compose up -d --build
```

Services:

| Service | Address |
| --- | --- |
| Frontend | <http://localhost:5173> |
| API / Swagger UI | <http://localhost:8000/docs> |
| PostgreSQL | `localhost:5432` |
| Mailpit web UI | <http://localhost:8025> |

Stop the stack with:

```powershell
docker compose down
```

Add `-v` only when you intentionally want to remove the PostgreSQL volume and
all local database data.

## Test the backend

Run the backend unit and endpoint tests from the repository root:

```powershell
$env:PYTHONPATH="backend;."
python -m pytest backend/tests -v
```

The frontend currently has no configured automated test, lint, or type-check
script in `package.json`. At minimum, validate frontend changes with:

```powershell
npm run build
```

## Explore the application

The main frontend flows are available from the home and sign-in screens. The
following routes are useful when collecting presentation screenshots:

| Route | Demonstrates |
| --- | --- |
| `/#/institution-preview` | Institution dashboard and local submission preview |
| `/#/officer-preview` | Officer command-centre preview |
| `/#/dashboard` | Portal dashboard |
| `/#/verify` | Transaction verification |
| `/#/documents` | Document management |
| `/#/assistant` | Natural-language assistant interface |
| `/#/calculator` | Deposit and penalty calculations |
| `/#/maturity-tracker` | Maturity tracking |
| `/#/release-requests` | Release/refund workflow |
| `/#/audit` | Audit view |

The preview pages use browser `localStorage` for local demo entries. Selecting
a file stores its name in the preview list; it does not upload the file.

### Demo accounts

The backend seeds demo accounts for institution users, scrutiny officers,
finance, audit, committee, bureau-head, and administrator roles. The backend
documentation lists the usernames and the demo password. Use these accounts
only in a disposable local environment and change the password strategy before
any deployment.

## Create the project PPT

### Recommended workflow

1. **Read the source of truth.** Start with
   [docs/SLIDE_FACTS.md](docs/SLIDE_FACTS.md). It distinguishes
   `IMPLEMENTED`, `PARTIAL`, `FRONTEND-ONLY`, `PLANNED`, and `NOT FOUND`.
2. **Run the demo.** Start the frontend and backend, then open the routes above.
3. **Capture evidence.** Use screenshots of the dashboard, verification,
   document, assistant, release, audit, and officer views. Prefer one clear
   screenshot per idea.
4. **Build a six-slide story.** Keep each slide focused on one message:

   | Slide | Message | Evidence to show |
   | --- | --- | --- |
   | 1. Title | AICTE Security Deposit / FDR Management Portal | Product name and one-line purpose |
   | 2. Proposed solution | A traceable workflow from submission to officer decision | Institution and officer screens; workflow arrows |
   | 3. Technical approach | React frontend, FastAPI services, database, matching, document checks, and audit trail | Architecture diagram or stack diagram |
   | 4. Feasibility and viability | Working backend paths and testable local deployment | Swagger UI, test output, Docker services |
   | 5. Impact and benefits | Better visibility, structured review, evidence traceability, and fewer manual handoffs | Dashboard, verification, release, and audit screens |
   | 6. Research and next steps | What is implemented, what is partial, and what is required for production | Status table and roadmap |
5. **Label the maturity honestly.** Say “prototype”, “demo”, “partial”, or
   “frontend-only” wherever applicable. Do not present local seed data as live
   institutional data.
6. **Keep the conclusion actionable.** End with the next steps: approve
   policy and ownership, integrate identity and data sources, complete security
   controls, and validate the production deployment.

### Presentation rules

- Use the exact project name and capabilities documented in
  [docs/SLIDE_FACTS.md](docs/SLIDE_FACTS.md).
- Do not claim official AICTE deployment, live bank integration, real OCR,
  production authentication, or complete multilingual support.
- Do not put unsupported accuracy percentages, speed claims, user counts, or
  financial impact figures on slides.
- Explain that automated matching and document analysis produce evidence and
  flags; an authorised officer makes consequential decisions.
- Keep detailed implementation notes in speaker notes rather than filling
  slides with endpoint lists.
- Use screenshots from the local demo only, with synthetic data and no secrets.

## Repository structure

```text
.
├── backend/
│   ├── app/
│   │   ├── routers/       # FastAPI API routes
│   │   ├── services/      # Matching, rules, documents, reports, audit
│   │   └── ...
│   ├── tests/             # Backend unit and endpoint tests
│   └── requirements.txt
├── docs/
│   └── SLIDE_FACTS.md     # Evidence-based presentation content
├── public/                # Logos and banner assets
├── src/
│   ├── components/        # Shared UI and portal shells
│   ├── pages/             # Portal, preview, workflow, and report screens
│   ├── data/              # API wrappers and local preview data
│   └── styles/            # Plain CSS
├── docker-compose.yml
├── .env.example
└── package.json
```

## Current status and limitations

The backend contains real API, persistence, workflow, matching, document,
notification, report, and audit paths for local demonstration. The frontend
contains both API-backed portal screens and local preview screens. The
repository is not a production-ready public service.

Before describing the project as production-ready, it still needs:

- An approved identity provider, MFA, session lifecycle, and account recovery.
- Server-side authorization review, threat modelling, security testing, and
  secret management.
- Authoritative AICTE policy rules, versioning, approvals, and boundary-case
  validation.
- Approved bank/FDR integrations and reconciliation procedures.
- Private document storage, malware scanning, retention, encryption, and
  access-controlled downloads.
- Production monitoring, backups, restore drills, incident response, and
  disaster recovery.
- Accessibility, contract, integration, and end-to-end test coverage.

See the detailed evidence and status assessment in
[docs/SLIDE_FACTS.md](docs/SLIDE_FACTS.md) and the backend-specific notes in
[backend/README.md](backend/README.md).

## Production-readiness checklist

Use this checklist before any real deployment:

- [ ] Scope, data classification, policy ownership, and retention are approved.
- [ ] Identity, roles, institution/region boundaries, and least privilege are
      verified server-side.
- [ ] Database migrations, backups, restore tests, and rollback procedures are
      in place.
- [ ] Bank, notification, document, and storage integrations are authorized.
- [ ] Financial calculations and workflow transitions are tested against
      approved policy examples.
- [ ] Security, privacy, accessibility, penetration, and dependency reviews
      are complete.
- [ ] Logs, metrics, alerts, audit review, and incident runbooks are ready.
- [ ] Development, staging, and production environments are separated.
- [ ] Go-live approval and operational ownership are documented.
