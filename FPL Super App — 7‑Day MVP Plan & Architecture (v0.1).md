# FPL Super App — 7‑Day MVP Plan & Architecture (v0.1)

**Owner:** Gethyn Ellis
**Date:** 11 Nov 2025
**Goal:** Deliver a viable, demo‑ready MVP in **7 days** that:

* Displays live FPL web data via **Power BI** (embedded).
* Gates the full analytical model behind **Entra ID External** sign‑up/sign‑in.
* Uses **Azure SQL Database** + **Microsoft Fabric** for the data tier.
* Is the basis for onboarding additional developers quickly.

---

## 1) MVP Scope & Success Criteria

### In‑scope (MVP)

* **Public landing page** with value proposition and teaser charts (free access).
* **Sign‑up/sign‑in** using **Entra ID External** (authentication) with two roles: `Free`, `Member`.
* **Member dashboard** with embedded **Power BI** report powered by Azure SQL/Fabric model.
* **Daily FPL data ingestion** (players, teams, fixtures, live GW summary) → Azure SQL (Bronze/Silver) → Fabric Lakehouse/Warehouse → Power BI model refresh.
* **Basic profile** (user record, favourite team, preferred captain metric).
* **Audit/telemetry** (App Insights + basic usage events: sign‑ins, report views).

### Out‑of‑scope (post‑MVP)

* Payments/subscriptions, social features, ML recommendations, mobile apps, heavy personalisation, rate‑limited per‑user APIs.

### Success criteria (by Day 7)

* A user can **sign up** and **sign in** via Entra ID External.
* A signed‑in user sees a **Power BI** report embedded in the web app, using **role‑based access** to the full model.
* A public visitor sees teaser visuals only (or a placeholder).
* Daily pipeline runs end‑to‑end within budget and completes < 30 minutes.
* A demo script runs cleanly from zero: deploy infra, run pipelines, create report, embed, log in.

---

## 2) High‑Level Architecture

```
[User Browser]
     | HTTPS
     v
[Front End]
  (Next.js or React on Azure Static Web Apps)
     | OAuth/OIDC
     v
[Entra ID External]
     | JWT w/ roles (Free/Member)
     v
[Backend API]
  (FastAPI or Azure Functions HTTP)
     |
     |--(service principal)--> [Power BI REST API]
     |                          (embed tokens)
     |
     |--(ADO.NET/pyodbc)-----> [Azure SQL Database]
     |
     |--(Fabric APIs)--------> [Microsoft Fabric]

[Data Ingestion]
  Azure Function / Fabric Data Pipeline →
  Bronze (raw FPL JSON) → Silver (clean) → Gold (model views)
  Stored in Azure SQL + Fabric Lakehouse/Warehouse
```

**Why this split?**

* **Static Web Apps (SWA)** makes auth integration and CI/CD simple.
* **FastAPI / Azure Functions** keeps the server thin (token exchange, embed tokens, profile endpoints).
* **Fabric** provides scalable modelling + Power BI; **Azure SQL** offers relational persistence and easy joins for APIs.

---

## 3) User Roles & Journeys (MVP)

### Roles

* **Visitor (anonymous):** Can view public landing and teaser charts only.
* **Member (authenticated via Entra ID External):** Sees full Power BI model and Member dashboard.

### Journeys

1. **Visitor → Member:** Click **Sign up** → Entra ID External user flow → redirect to Member dashboard.
2. **Member login:** Entra ID External → JWT back to SWA → call backend → receive **Power BI embed token** → render full report.
3. **Daily refresh:** Function/Pipeline pulls FPL endpoints → persists raw + curated → triggers Fabric/Power BI refresh.

---

## 4) Data Model (MVP cut)

### Source: FPL Official endpoints (read‑only)

* `/bootstrap-static` (players, teams, positions, current GW meta)
* `/fixtures` (fixtures schedule)
* `/event/{id}/live` (live points per GW) — optional for MVP if time is tight.

### Azure SQL (curated minimal schema)

* **Teams**(TeamID PK, Name, ShortName, Strength, Code)
* **Players**(PlayerID PK, TeamID FK, WebName, Position, Cost, SelectedBy, Status, NowCost, Form)
* **Fixtures**(FixtureID PK, GW, HomeTeamID FK, AwayTeamID FK, KickoffUtc, HomeDiff, AwayDiff)
* **Gameweeks**(GW PK, StartUtc, EndUtc, Finished)
* **Users**(UserID PK GUID, Email, DisplayName, FavouriteTeamID FK, CreatedUtc, Role)
* **Telemetry**(Id PK, UserID, EventName, EventUtc, Meta JSON)

### Fabric Lakehouse/Warehouse

* Bronze tables for raw JSON (landing).
* Silver tables (column‑typed, denormalised views for PBI).
* Gold views for **Power BI semantic model**: `dimTeam`, `dimPlayer`, `dimGameweek`, `factPlayerGW` (if time allows).

---

## 5) Power BI Design (MVP)

* **One report** with two pages:

  1. **Overview (Public/Teaser):** Top 10 players by form, upcoming favourable fixtures, next GW deadlines (limited dataset).
  2. **Member Dashboard:** Full player explorer, fixtures difficulty matrix, captain picks (basic DAX).
* **Dataset** hosted on **Fabric capacity** (or Pro if capacity available).
* **Row‑level gating** not required for MVP; feature gating handled in app (role check) and separate public vs member report if needed.
* **Refresh:** daily after FPL updates; optional on‑demand refresh via API.

---

## 6) Authentication & Authorisation (Entra ID External)

* **Tenant:** Create Entra ID External (formerly Azure AD B2C) with a **user flow** for sign‑up/sign‑in.
* **App registrations:**

  * **Web Client** (SWA): SPA, redirect URIs, CORS.
  * **Backend API**: Expose scope `api.access`.
  * **Power BI Service Principal**: separate app reg with necessary permissions in Power BI Admin (Embed for your customers).
* **Tokens:** SPA acquires ID token + access token to call backend; backend validates JWT and issues **Power BI embed tokens**.
* **Roles/claims:** Assign `role=Member` post‑sign‑up (default `Free`), store in Users table; lightweight admin endpoint for role flip.

---

## 7) Backend API (minimal)

**Framework:** FastAPI (Python) or Azure Functions HTTP (Python/Node).
**Endpoints:**

* `GET /api/health` → 200
* `GET /api/me` → profile from Users
* `POST /api/profile` → update display name, favourite team
* `GET /api/pbi/embed-token?reportId=...` → returns embed token for authenticated Members
* `POST /api/ingest/run` (protected, admin) → trigger data pull

**Services:**

* **FPLClient** (requests)
* **SqlRepo** (sqlalchemy/pyodbc)
* **PBIClient** (Power BI REST: get dataset/report, generate embed token)

**Secrets:** Azure Key Vault (SAS/connection strings, client secrets).

---

## 8) Data Ingestion & Orchestration

* **Option A (fastest):** Azure Function (Timer Trigger daily) → call FPL API → write Bronze JSON to Fabric + curated to Azure SQL.
* **Option B:** Fabric Data Pipelines/Notebooks for ingestion & transformation; trigger via schedule.

**Transformations:**

* Map FPL JSON → `Teams`, `Players`, `Fixtures`, `Gameweeks`.
* Simple type cleaning, null handling, deduping.
* Optional `factPlayerGW` if time remains.

---

## 9) Hosting & Environments

* **Front end:** Azure Static Web Apps (SWA)
* **Backend:** Azure Functions (Consumption) or App Service (Free/Basic)
* **Database:** Azure SQL (Basic/DTU S0)
* **Fabric:** Lakehouse + Power BI workspace on F capacity (or trial)
* **Monitoring:** App Insights + Log Analytics (basic).
* **Environments:** `dev` and `prod` (two SWA sites, two function apps, one shared Fabric workspace if needed).

---

## 10) CI/CD & Repo Structure

**Repo layout**

```
/infra        # bicep/terraform for SWA, Functions, SQL, Key Vault
/frontend     # Next.js app (SWA build)
/backend      # FastAPI or Functions
/pipelines    # Fabric notebooks/pipelines or Azure Functions ingestion
/powerbi      # .pbix/.tmdl and deployment scripts
/docs         # this document, ADRs, decisions
```

**Branching**: `main` (prod), `develop` (dev), feature branches.
**Pipelines**: GitHub Actions or Azure DevOps: build → test → deploy per env.
**Secrets**: GitHub OIDC to Azure; store app secrets in Key Vault.

---

## 11) Day‑by‑Day Execution Plan (7 days)

**Day 1 – Foundations**

* Create repos, issues, Kanban board.
* Provision Entra ID External tenant + user flow.
* Scaffold SWA front end and backend API; wire basic auth flow end‑to‑end (silent login works).
* Create Azure SQL + baseline schema.

**Day 2 – Data Ingestion (Bronze/Silver)**

* Implement ingestion (Function or Fabric) for `/bootstrap-static`.
* Persist Teams/Players/Gameweeks/Fixtures to Azure SQL.
* Add daily schedule; idempotency + simple retry.

**Day 3 – Power BI Dataset & Report**

* Build minimal dataset (Fabric) referencing SQL/Fabric tables.
* Create two report pages (Teaser + Member).
* Validate refresh.

**Day 4 – Embedding & Roles**

* Configure Power BI service principal + workspace permissions.
* Implement backend endpoint to generate **embed tokens**.
* Front end renders teaser for Visitors, full report for Members.

**Day 5 – UX polish & Profile**

* Add profile page (display name, favourite team).
* Basic navigation, responsive layout, loading states.
* Add minimal telemetry (page views, sign‑ins, embed loads).

**Day 6 – Hardening & Docs**

* Error handling, timeouts, retries, logging.
* Threat model checklist; secure headers, CORS, CSRF where relevant.
* Populate run‑books and a demo script.

**Day 7 – Testing & Launch**

* UAT script, bug‑bash, smoke tests.
* Prepare demo data, screenshots, and a 2‑minute video walkthrough.
* Tag release `mvp-1` and publish.

---

## 12) Security, Compliance & Privacy (MVP)

* Use **Entra ID External** user flows and secure defaults.
* Store minimum PII (email, display name, favourite team).
* Data residency: UK region where possible.
* Key Vault for secrets; least‑privilege across services.
* DDoS/abuse guard via rate limiting on backend.

---

## 13) Cost Guardrails (rough, per month)

* SWA Free/Standard: £0–£10
* Azure Functions (consumption): £<10
* Azure SQL S0: ~£25–£30
* Fabric capacity (trial or F SKU small): varies; plan for trial or Pro during MVP
* App Insights/Logs: £5–£20
  **Target:** < £100 during MVP using trials/dev SKUs.

---

## 14) Developer Onboarding Checklist

* Access to Azure subscription, Entra ID External tenant, Fabric workspace.
* Clone repo; install Node 20+, Python 3.11+, Azure CLI.
* `frontend`: env vars for Entra IDs (authority, clientId), API base URL.
* `backend`: Key Vault access; env vars for SQL, PBI credentials; run `uvicorn`/`func start`.
* `pipelines`: run local ingestion once; verify SQL tables populated.
* Run `make dev` (or npm scripts) to start full stack locally.

---

## 15) Risks & Mitigations

* **Power BI embedding complexity** → use service principal, follow embed-for‑customers guide; fallback to non‑embedded link behind login if blocked.
* **Fabric capacity limits** → start with small dataset; store in Azure SQL; optimise visuals.
* **FPL API changes/rate limits** → cache responses, back‑off retries; reduce refresh frequency.
* **Timeline pressure** → prioritise Day 1–4 deliverables; defer Player‑GW fact table if needed.

---

## 16) Next Steps (Post‑MVP Roadmap)

* Payments + entitlements (Stripe) → `Pro` role.
* Personalised ML recommendations (captain, transfers).
* Team sync via FPL login (if permissible) or user input.
* Mobile app wrapper (Capacitor) once engagement proven.
* Community features: mini‑leagues, sharing, comments.

---

## 17) Appendix: Minimal SQL (illustrative)

```sql
CREATE TABLE dbo.Teams (
  TeamID       INT PRIMARY KEY,
  Name         NVARCHAR(100) NOT NULL,
  ShortName    NVARCHAR(10)  NOT NULL,
  Strength     INT           NULL,
  Code         INT           NULL
);

CREATE TABLE dbo.Players (
  PlayerID     INT PRIMARY KEY,
  TeamID       INT NOT NULL REFERENCES dbo.Teams(TeamID),
  WebName      NVARCHAR(100) NOT NULL,
  Position     NVARCHAR(20)  NOT NULL,
  Cost         DECIMAL(6,2)  NULL,
  SelectedBy   DECIMAL(5,2)  NULL,
  Status       NVARCHAR(10)  NULL,
  NowCost      DECIMAL(6,2)  NULL,
  Form         DECIMAL(4,2)  NULL
);

CREATE TABLE dbo.Gameweeks (
  GW           INT PRIMARY KEY,
  StartUtc     DATETIME2 NOT NULL,
  EndUtc       DATETIME2 NOT NULL,
  Finished     BIT       NOT NULL DEFAULT 0
);

CREATE TABLE dbo.Fixtures (
  FixtureID    INT PRIMARY KEY,
  GW           INT NOT NULL REFERENCES dbo.Gameweeks(GW),
  HomeTeamID   INT NOT NULL REFERENCES dbo.Teams(TeamID),
  AwayTeamID   INT NOT NULL REFERENCES dbo.Teams(TeamID),
  KickoffUtc   DATETIME2 NOT NULL,
  HomeDiff     INT NULL,
  AwayDiff     INT NULL
);

CREATE TABLE dbo.Users (
  UserID       UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
  Email        NVARCHAR(256) NOT NULL UNIQUE,
  DisplayName  NVARCHAR(100) NULL,
  FavouriteTeamID INT NULL REFERENCES dbo.Teams(TeamID),
  Role         NVARCHAR(20) NOT NULL DEFAULT 'Free',
  CreatedUtc   DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME()
);
```

---

**End of document.**
