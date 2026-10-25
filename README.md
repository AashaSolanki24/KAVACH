# KAVACH: AI-Assisted Safety and Incident Response

KAVACH is a web-based safety prototype that connects a user-facing app, an incident API, an AI audio-analysis service, and a dispatcher dashboard. It is intended for demonstration and evaluation; it does not guarantee emergency response and should not replace local emergency services.

## The problem

A basic alert may tell responders that someone needs help without giving them useful context. KAVACH explores a workflow that can combine an SOS, location updates, audio analysis, and incident details in a dispatcher dashboard.

## What the prototype includes

| Capability | Description |
|---|---|
| SOS and monitoring | User-facing web flows for starting and monitoring a safety incident. |
| Location updates | Sends available location information during an active incident, subject to browser permissions and connectivity. |
| Audio analysis | Sends captured audio to the AI service for transcription and analysis. |
| Risk assessment | Produces a risk score and contextual indicators to help organize incident information. A score is not a definitive assessment of danger. |
| Dispatcher dashboard | Shows incidents and available location and risk information to an authenticated dispatcher. |
| Incident reports | Generates incident reports from information available to the system. |
| Safe Ride | Provides a web-based monitoring flow for a ride or commute. |

The mobile application has been removed from this version. The user app and dispatcher dashboard are browser-based.

## How it works

1. A user starts an SOS or monitoring flow in the web app.
2. The browser shares location and, where the user grants permission, audio evidence.
3. The backend creates or updates the incident and requests analysis from the AI engine.
4. The AI engine returns a transcript and contextual risk information.
5. The backend sends incident updates to the dispatcher dashboard over Socket.IO.
6. A dispatcher signs in to review the available incident details and report.

## Architecture

KAVACH runs as four application services backed by PostgreSQL:

- **User web app:** React, Vite, and TypeScript (`frontend/`)
- **Backend API:** Node.js, Express, Socket.IO, and PostgreSQL (`backend/`)
- **AI engine:** Python, FastAPI, and Faster-Whisper (`ai-engine/`)
- **Dispatcher dashboard:** React and Vite (`police-dashboard/`)
- **Database:** PostgreSQL

## Run locally on Windows

### Prerequisites

Install Node.js with npm, Python 3.10 or later, and PostgreSQL. Make sure PostgreSQL is running and create a database named `kavach` (or update `DATABASE_URL` to match your database).

### Configure environment files

1. Copy the project-root `.env.example` to `.env`.
2. In `.env`, set `DATABASE_URL` to your local PostgreSQL connection, and set unique local values for `JWT_SECRET`, `AI_ENGINE_SECRET`, and `DISPATCHER_INVITE_CODE`. The backend and AI engine read the root `.env`.
3. Create `frontend/.env` with:

   ```dotenv
   VITE_API_URL="http://localhost:5000"
   ```

4. Copy `police-dashboard/.env.example` to `police-dashboard/.env`. For local development, it should contain:

   ```dotenv
   VITE_API_URL="http://localhost:5000"
   VITE_SOCKET_URL="http://localhost:5000"
   ```

Keep `.env` files private. Commit the `.env.example` files with placeholders only. Twilio and Firebase settings are optional for a basic local demo; without valid Twilio credentials, real SMS and calls will not be sent.

### Install dependencies

From the project root, install the JavaScript dependencies:

```powershell
npm run install:all
```

The root `install:all` script installs the user web app, backend, and dispatcher dashboard. Install the AI service dependencies in a Python virtual environment:

```powershell
py -3.10 -m venv ai-engine\.venv
.\ai-engine\.venv\Scripts\Activate.ps1
python -m pip install -r ai-engine\requirements.txt
```

The first audio-analysis run may download Whisper model files, so it can take longer and require an internet connection.

### Start the services

Keep each service running in its own PowerShell terminal. Start PostgreSQL first, then:

**AI engine** (activate the virtual environment first):

```powershell
Set-Location ai-engine
uvicorn main:app --host 127.0.0.1 --port 8000 --reload
```

**Backend**, from the project root:

```powershell
npm run start:backend
```

The backend connects to PostgreSQL and runs its database initialization during startup.

**User web app**, from the project root:

```powershell
npm run start:frontend
```

Open the Vite address shown in the terminal (normally `http://localhost:5173`).

**Dispatcher dashboard**, from the project root:

```powershell
npm run start:dashboard
```

Open `http://localhost:3000`.

The backend listens on port `5000`; the AI engine listens on port `8000`. Use a dispatcher account with the police role to sign in. Dispatcher requests use a JWT from login; the dashboard does not use a shared `VITE_POLICE_API_KEY`.

## Environment variables

The main root `.env` settings are:

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Signs and verifies application and dispatcher sessions |
| `AI_ENGINE_SECRET` | Shared internal credential for backend-to-AI requests |
| `BACKEND_URL` | Backend base URL used when constructing report and evidence links |
| `AI_ENGINE_URL` | AI service address, normally `http://localhost:8000` locally |
| `ALLOWED_ORIGINS` | Allowed local browser origins |
| `DISPATCHER_INVITE_CODE` | Invite code used when registering a dispatcher |
| `TWILIO_SID`, `TWILIO_TOKEN`, `TWILIO_PHONE_NUMBER` | Optional Twilio integration settings |
| `LOCAL_TZ` | Time zone used by the AI engine, for example `Asia/Kolkata` |

The Vite apps read `VITE_API_URL` from their own app directories. The dispatcher dashboard also reads `VITE_SOCKET_URL`. Do not put private secrets in variables prefixed with `VITE_`; Vite exposes those values in the browser bundle.

## Authentication, privacy, and limitations

- Sensitive routes require authentication. Dispatcher access uses a dispatcher account and JWT role checks.
- Browser location and microphone features depend on user permissions, device support, and network availability.
- Treat location, audio, transcripts, and incident records as sensitive data. Use synthetic data for demos and do not upload real people's evidence without their informed permission.
- AI transcription and risk scoring can be incomplete or incorrect. They are decision-support features, not proof, a diagnosis, or a guarantee of safety or response.
- Test alerts locally before connecting any real phone numbers or external services.

## Deployment

The code can be deployed as separate web apps and services, but each environment needs its own database, secrets, allowed browser origins, and public service URLs. Configure deployment values outside the repository. Add only deployment URLs that you control and have verified.

## Roadmap

- Validate the incident workflow with users and relevant responders.
- Improve accessibility, connectivity handling, and multilingual transcription.
- Add trusted-contact escalation options and evaluate them with consent and privacy safeguards.
- Expand dashboard reporting and analytics.
