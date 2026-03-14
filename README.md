# APIDoctorCheckUp — Infrastructure

Docker Compose configuration for running the full APIDoctorCheckUp stack locally with a single command.

**Live App:** https://api-doctor-check-up-client.vercel.app  
**API Repository:** https://github.com/janlu89/APIDoctorCheckUp-API  
**Client Repository:** https://github.com/janlu89/APIDoctorCheckUp-Client

---

## What This Runs

| Service | Port | Description |
|---|---|---|
| `api` | 8080 | .NET 10 backend — built from APIDoctorCheckUp-API |
| `client` | 4200 | Angular 21 frontend — built from APIDoctorCheckUp-Client |

The API database is created and migrated automatically on first start. No manual database setup required.

---

## Prerequisites

- Docker Desktop
- Git

---

## Quick Start

```bash
# Clone this repository
git clone https://github.com/janlu89/APIDoctorCheckUp-Infra.git
cd APIDoctorCheckUp-Infra

# Clone the application repositories alongside this one
git clone https://github.com/janlu89/APIDoctorCheckUp-API.git
git clone https://github.com/janlu89/APIDoctorCheckUp-Client.git

# Copy and fill in environment variables
cp .env.example .env

# Start the full stack
docker compose up
```

Open `http://localhost:4200` in your browser.

---

## Environment Variables

Copy `.env.example` to `.env` and fill in your values before starting:

| Variable | Description |
|---|---|
| `JWT_SECRET` | JWT signing key — minimum 32 characters |
| `ADMIN_USERNAME` | Admin login username |
| `ADMIN_PASSWORD` | Admin login password |

---

## Stopping the Stack

```bash
# Stop all containers
docker compose down

# Stop and remove volumes (clears the database)
docker compose down -v
```

---

## Production Deployment

The production stack uses managed services rather than Docker Compose:

| Component | Service | Notes |
|---|---|---|
| API | Render.com | Docker deployment, Frankfurt region |
| Database | Neon.tech | PostgreSQL, EU West 2 |
| Frontend | Vercel | CDN, auto-deploys on push to master |
| Keep-alive | UptimeRobot | Pings `/health` every 5 minutes |

See the individual repository READMEs for full deployment instructions.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────┐
│                   Browser                        │
│         Angular 21 — Vercel CDN                 │
└────────────────────┬────────────────────────────┘
                     │ HTTP + WebSocket (SignalR)
┌────────────────────▼────────────────────────────┐
│              .NET 10 Web API                     │
│              Render.com — Frankfurt              │
│                                                  │
│  Background workers (one per endpoint)           │
│  Alert evaluator + incident lifecycle            │
│  SignalR hub — broadcasts on every check         │
└────────────────────┬────────────────────────────┘
                     │ Npgsql
┌────────────────────▼────────────────────────────┐
│            PostgreSQL — Neon.tech                │
│            EU West 2 (London)                    │
└─────────────────────────────────────────────────┘
```
