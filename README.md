# SAPS Digital Case Docket Management and Accountability System

The SAPS Digital Case Docket Management and Accountability System (DPCDMAS) is a local React, Express, TypeScript, and SQLite prototype for police case-docket workflows and accountability review.

The application provides role-specific workstations for docket intake, investigation, evidence handling, supervisory review, compliance oversight, reporting, citizen self-service, and system administration. It is a prototype and must not be treated as a production police-records, evidence, identity, or access-control system without a formal security, legal, operational, and privacy review.

## What the application does

### Authentication and entry points

- Opens at a login screen unless the browser session contains `saps_session_active=true`.
- Provides Officer and Admin login modes.
- The Officer selector can switch among the non-admin, non-victim seeded operational profiles.
- The Admin mode selects the seeded system administrator.
- The login form includes a password/PIN field, but the current client login flow selects a local profile and completes after a short simulated verification delay. It does not submit the entered password to `/api/auth/login`.
- Fingerprint, facial, and quick-admin controls are simulated client-side profile switches; they are not hardware biometric integrations.
- The Station Self-Service Kiosk can be opened before login and from an operational user session.
- Logout removes the browser session marker and returns to the login screen.

### Operational modules

- **Role dashboard:** renders a different workstation for officer, supervisor, commander, auditor, management, and victim profiles. The admin dashboard opens user management.
- **Case dockets:** lists accessible dockets and opens a case detail workstation. Officers and commanders can open/register cases through the intake workflow.
- **Victim and citizen intake:** captures complainant, incident, sworn-statement, domestic-violence referral, suspect, evidence, and dispatch information. Intake tickets use identifiers such as `VAP-2026-000152`; case dockets use CAS-style identifiers such as `CAS 142/09/2026`.
- **Refusal and escalation workflow:** records intake refusals, supervisor decisions, escalation references, possible overrule-and-register actions, and disciplinary notices. The public kiosk can submit an unlawful-refusal report.
- **Investigation workspace:** manages investigation tasks, task statuses, witness statements, suspects, custody details, investigation diary entries, audio/voice field notes, and AI-assisted field-note formatting.
- **Witness statements:** records complainant, eyewitness, expert-witness, and suspect statements with sworn/signature fields and a generated digital-hash representation.
- **Suspects and custody:** displays suspects, bail/custody information, and the Section 50 first-court-appearance monitoring data represented by the case model.
- **Evidence (SAP 13):** records exhibits and media, supports image/audio/video/document attachments, calculates a SHA-256 browser hash when available, and records chain-of-custody transfers and high-risk flags.
- **Two-person closure review:** officers submit closure requests; supervisors and commanders review them. The workflow checks separation of duties and supports approval, rejection, return, and reopening paths.
- **Compliance engine:** evaluates configured rules for intake, domestic-violence Form 1/referrals, arrested-person rights, the 48-hour custody rule, SAP 13 evidence handling, and two-person closure review.
- **AI assistance:** supports compliance analysis, closure-risk analysis, fraud-risk analysis, statutory-deadline monitoring, chat, translation, and field-note formatting. Gemini is used only when a valid `GEMINI_API_KEY` is present; server-side deterministic rules are used otherwise. Client-side fallback behavior also exists for failed AI requests.
- **Complaints and oversight:** records complaints, resolution findings, and resolution summaries. Auditor and command workstations expose compliance and accountability information.
- **Reports and statistics:** provides role-appropriate station, compliance, and provincial reporting views.
- **Audit trail:** displays audit records and provides server-side SHA-256 hash-chain verification for the SQLite audit ledger.
- **User management:** administrators can create, edit, and delete user profiles. Admin navigation is restricted to user management and audit access; operational docket modules are blocked.
- **Data settings:** supports exporting the client dataset to JSON, importing JSON, resetting client state to the default demo dataset, and clearing client cases.

## Role access in the current UI

The role names and navigation below are taken from `src/types.ts`, `src/data/seedData.ts`, and `src/components/Sidebar.tsx`.

| Role | Current navigation and scope |
| --- | --- |
| `officer` | My Workstation, Case Dockets, Register Case (CSC), Witness Statements, Suspects & Bail, Evidence (SAP 13). Unassigned dockets, closure reviews, and audit are restricted. |
| `supervisor` | Supervisor Dashboard, Unassigned Dockets, All Case Dockets, Closure Reviews, Evidence Oversight, Suspects & Custody, Reports & Stats. |
| `commander` | Commander Console, Open / Register Case, Docket Allocation, Station Docket Roll, Evidence (SAP 13), Closure Authorisations, Station Performance, Audit Trail. |
| `auditor` | IPID Console, Docket Compliance Audit, Cryptographic Audit, Compliance Reports. Field intake, witness, suspect, and evidence modules are restricted. |
| `management` | Provincial Console, Provincial Dockets, Provincial Statistics, Audit Trail. |
| `victim` | Citizen Docket Portal dashboard. The login selector excludes this profile; it remains available in the seeded data and role model. |
| `admin` | Manage Users and Audit Trail. Operational police docket modules are restricted. |

## Seeded demo profiles

The browser client seeds these profiles from `src/data/seedData.ts`. The login screen selects profiles rather than requiring a real credential exchange.

| Name | Role | Station or organisation |
| --- | --- | --- |
| K. Mthembu | `officer` | Johannesburg Central |
| Lt. Col. M. Jacobs | `commander` | Johannesburg Central |
| Sgt. D. Khumalo | `officer` | Johannesburg Central |
| Ofc. L. Maseko | `officer` | Johannesburg Central |
| Ofc. P. Naidoo | `officer` | Johannesburg Central |
| Constable Sipho Sithole | `officer` | Pretoria Central SAPS |
| Detective Sergeant Nomvula Khumalo | `officer` | Pretoria Central SAPS |
| Captain David van der Merwe | `supervisor` | Pretoria Central SAPS |
| Colonel Thandiwe Ndlovu | `commander` | Pretoria Central SAPS |
| Brigadier Johan Botha | `management` | Gauteng Provincial Headquarters |
| Inspector Fatima Patel | `auditor` | IPID |
| Lerato Mokoena | `victim` | Pretoria Central Community Area |
| Thabo Molefe | `admin` | SAPS National Technology Management |

The seed files also contain example tickets, cases, evidence, complaints, compliance rules, audit records, anomalies, and kiosk-presence records. The client initializes cases and evidence as empty when no client-side case/evidence state exists; use the Settings reset action when the full demo dataset is required.

## Technology and storage

- **Frontend:** React 19, TypeScript, Vite, Tailwind CSS 4, Lucide React, Motion, jsPDF, and the Google GenAI SDK.
- **Backend:** Express with `tsx` for development and `esbuild` for the production server bundle.
- **Database:** SQLite through Node’s built-in `node:sqlite` API. The server creates `data/saps_police_system.sqlite` and uses WAL mode, foreign keys, and normal synchronous mode. SQLite sidecar files such as `-wal` and `-shm` may appear.
- **Client state:** most visible UI data is loaded from and written to browser `localStorage` under `dpcdmas_*` keys. The login portal marker is stored in `sessionStorage` under `saps_session_active`.
- **Server state:** the Express API persists users, cases, evidence, tickets, complaints, audit logs, security sessions, and security events in SQLite.
- **Integration boundary:** the client directly calls selected server endpoints for diary entries, closure review, AI analysis, statutory-deadline analysis, database status/reset, and audit verification. Many other client workflows remain local-storage driven, so a browser reset and a SQLite reset are separate operations.

## Requirements

- Node.js 22.5 or newer is recommended because the server imports the built-in `node:sqlite` module. Node.js 25.8.2 was used to verify this repository.
- npm 11 or a compatible current npm release.
- A modern browser with `localStorage`, `sessionStorage`, Web Crypto, FileReader, and (where voice notes are used) browser speech/audio APIs.
- A Gemini API key is optional. Without it, the local rule engine remains available.

## Local development

From the repository root:

```bash
npm install
npm run dev
```

Open <http://localhost:3000>.

The `dev` script runs `tsx server.ts`. Express and Vite are mounted in the same process, and the server listens on port `3000` on all interfaces. The server initializes the SQLite database at startup.

### Environment variables

Create a `.env` file in the repository root only when needed:

```env
GEMINI_API_KEY=your_gemini_api_key_here
```

`GEMINI_API_KEY` enables server-side Gemini requests. The placeholder value in `.env.example` is treated as unset. `APP_URL` is documented in `.env.example` for hosted environments but is not required for local startup by the current server code.

## Production build and preview

Build both the Vite client and bundled Express server:

```bash
npm run build
```

This writes the browser bundle to `dist/` and the server bundle to `dist/server.cjs`.

Start the bundled server:

```bash
npm start
```

The production server also listens on port `3000` and serves the built Vite application. `npm run preview` starts Vite’s separate static preview server and is not the normal full-stack production command.

Other scripts:

```bash
npm run lint   # TypeScript check with no emit
npm run clean  # Removes dist and server.cjs on Unix-like shells
```

On Windows PowerShell, remove `dist` manually if `npm run clean` is not supported by the active shell because the script uses `rm -rf`.

## HTTP API

The Express server exposes these routes. JSON request and response shapes are defined by the handlers and domain types in `server.ts`, `server/database.ts`, and `src/types.ts`.

### Authentication and system

- `POST /api/auth/login`
- `POST /api/auth/switch-user`
- `GET /api/auth/me`
- `POST /api/auth/logout`
- `GET /api/database/status`
- `POST /api/database/seed` with optional `forceReset`
- `GET /api/health`

### Audit and users

- `POST /api/audit/verify`
- `GET /api/audit?limit=50`
- `GET /api/security/events?limit=50`
- `GET /api/users`

### Cases and investigation

- `GET /api/cases`
- `GET /api/cases/:id`
- `POST /api/cases`
- `PUT /api/cases/:id`
- `POST /api/cases/:id/closure-request`
- `POST /api/cases/:id/closure-review`
- `POST /api/cases/:id/reopen`
- `POST /api/cases/:id/diary`

### Evidence, tickets, and complaints

- `GET /api/evidence`
- `POST /api/evidence`
- `POST /api/evidence/:id/transfer`
- `GET /api/tickets`
- `POST /api/tickets`
- `GET /api/complaints`
- `POST /api/complaints`

### AI assistance

- `POST /api/ai/analyze`
- `POST /api/ai/statutory-deadlines`
- `POST /api/ai/chat`
- `POST /api/ai/translate`
- `POST /api/ai/format-field-note`

Most routes return `{ success, data }` on success or `{ success: false, error }` on failure. Routes that use a session token accept it through the `Authorization` header. In development, the server has a fallback user when no valid token is supplied, so this API behavior is not production-grade authentication.

## Data reset and privacy notes

- **Settings reset in the browser:** clears client `localStorage` and restores the client seed arrays. It does not reset the SQLite database.
- **Server database reset:** `POST /api/database/seed` can reseed the SQLite database; `forceReset` controls the reset behavior in the server database layer.
- **Clear all cases:** the client action clears client-side cases and evidence state and should not be confused with deleting server database records.
- The repository may contain SQLite database and WAL files under `data/`. Treat them as application data and do not commit sensitive real-world records.
- The sample data contains personally identifying-looking names, contact details, identity numbers, and badge numbers. Use only the supplied fictional/demo records for development.

## Verification

The current repository has been verified with:

```bash
npm install
npm run lint
npm run build
```

The development server was also checked at `http://localhost:3000` and returned HTTP 200.
