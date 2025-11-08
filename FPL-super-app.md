

## 🏗️ FPL High-Level Vision

A web application that:

* Displays **Fantasy Premier League stats** (players, teams, points, fixtures, price changes).
* Provides **weekly recommendations** (captain, transfers, sleepers, and differentials).
* Predicts **match results and player performance** using **machine learning** models.
* Lets users **track mini-leagues**, favourite players, and compare their own teams.
* Runs fully on **Microsoft Azure**, designed for **scalability, data analytics, and AI integration**.

---

## 🧱 Architecture Overview

| Layer                             | Description                                                               | Technology                                                                                                                 |
| --------------------------------- | ------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **Frontend (Web App)**            | User interface for dashboards, stats, and recommendations                 | React.js (Next.js for SSR) + Tailwind CSS + Deployed via **Azure Static Web Apps**                                         |
| **Backend API**                   | REST + GraphQL endpoints for FPL data, ML predictions, and authentication | Python **FastAPI** or **Node.js (Express/NestJS)** hosted on **Azure App Service**                                         |
| **Database Layer**                | Stores FPL player data, match history, user profiles, recommendations     | **Azure SQL Database** (for structured relational data) + **Azure Blob Storage** (for static JSON files or media)          |
| **Data Ingestion & Processing**   | Fetches FPL data, cleans and stores it                                    | **Azure Functions** (scheduled via Timer Trigger) pulling data from [FPL API](https://fantasy.premierleague.com/api/)      |
| **Machine Learning & Prediction** | Predicts match results, assists, and captain picks                        | **Azure Machine Learning Service** + **Python (scikit-learn, pandas, XGBoost)** + optional **Fabric or Synapse Notebooks** |
| **Data Analytics & Reporting**    | Dashboards for player stats and trends                                    | **Power BI Embedded** or **Azure Fabric Warehouse**                                                                        |
| **Authentication**                | User sign-in, optional team linkage                                       | **Azure AD B2C** for secure user management                                                                                |
| **Monitoring & CI/CD**            | Automated deployment and health monitoring                                | **GitHub Actions** + **Azure Monitor / Application Insights**                                                              |

---

## 🔁 Data Flow

1. **Data Ingestion**

   * Azure Function runs daily → fetches [Fantasy Premier League API](https://fantasy.premierleague.com/api/bootstrap-static/).
   * Cleans and transforms data → saves to Azure SQL tables: `Players`, `Teams`, `Fixtures`, `Gameweeks`.
   * Historical stats archived in Blob Storage.

2. **ML Pipeline**

   * Azure ML / Fabric Notebook trains models using historical data.
   * Predictions written back to SQL table `Predictions` with confidence scores.

3. **API Layer**

   * FastAPI exposes endpoints like:

     * `/api/players`
     * `/api/recommendations`
     * `/api/predictions`
   * Uses SQLAlchemy ORM to query data.
   * Includes caching layer (Redis or Azure Cache for Redis).

4. **Frontend**

   * Next.js React app fetches from backend.
   * Uses dynamic routes: `/player/[id]`, `/gameweek/[gw]`, `/predictions`.
   * Displays Power BI Embedded charts for visuals.

---

## 🧰 Development Plan

### Phase 1: Foundation

* ✅ Set up Azure DevOps / GitHub repository.
* ✅ Create **Azure SQL Database** and define schema.
* ✅ Scaffold backend with **FastAPI** and **SQLAlchemy**.
* ✅ Implement CI/CD pipeline to Azure App Service.

### Phase 2: Data Pipeline

* ⚙️ Build Azure Function to fetch FPL API data.
* ⚙️ Store player, fixture, and gameweek data in SQL.
* ⚙️ Schedule it daily using Azure Timer Trigger.

### Phase 3: Web Frontend

* 🌐 Create React/Next.js front end.
* 🌐 Integrate charts (Recharts, Chart.js, or Power BI Embedded).
* 🌐 Display team stats, fixture lists, and leaderboards.

### Phase 4: Machine Learning

* 🤖 Use Azure ML or Fabric Notebook to:

  * Train predictive models (e.g., regression or gradient boosting).
  * Generate weekly captain/transfer recommendations.
  * Store outputs in SQL and expose via API.

### Phase 5: Authentication and Personalisation

* 🔐 Add Azure AD B2C login.
* 👤 Allow users to favourite players, compare teams, and save settings.

### Phase 6: Deployment & Scaling

* 🚀 Deploy front end to **Azure Static Web Apps**.
* 🚀 Backend to **Azure App Service** with staging slots.
* 📈 Configure **Azure Monitor + Application Insights** for telemetry.

---

## 🧩 Azure Resources Summary

| Resource                     | Purpose                       |
| ---------------------------- | ----------------------------- |
| **Azure SQL Database**       | Main structured data store    |
| **Azure Blob Storage**       | Static assets, archived JSONs |
| **Azure App Service**        | Host backend API              |
| **Azure Static Web Apps**    | Host front end                |
| **Azure Functions**          | Scheduled ingestion jobs      |
| **Azure Machine Learning**   | Model training and scoring    |
| **Azure AD B2C**             | Authentication                |
| **Azure Monitor & Insights** | Logging and telemetry         |
| **Azure Cache for Redis**    | Speed up frequent queries     |

---

## 💻 Using Codex for Development

You can use **GitHub Copilot** or **Azure OpenAI Codex** to:

* Autogenerate FastAPI routes, SQLAlchemy models, and Pydantic schemas.
* Build React components and pages rapidly.
* Scaffold test data ingestion scripts for the FPL API.
* Write unit tests and documentation comments automatically.

---

## 🗓️ Suggested Timeline (8 Weeks MVP)

| Week | Focus    | Deliverables                                   |
| ---- | -------- | ---------------------------------------------- |
| 1    | Setup    | Repo, Azure infra, CI/CD pipeline              |
| 2    | Database | SQL schema, ORM setup                          |
| 3    | Data     | FPL ingestion + cron jobs                      |
| 4    | Backend  | API routes + Swagger docs                      |
| 5    | Frontend | React UI + charts                              |
| 6    | ML       | Predictive models (simple regression baseline) |
| 7    | Auth     | Azure AD B2C + user preferences                |
| 8    | Polish   | Dashboard styling, monitoring, deploy MVP      |

---

## 📊 Example SQL Schema

| Table         | Key Columns                                                                        |
| ------------- | ---------------------------------------------------------------------------------- |
| `Players`     | `PlayerID`, `Name`, `TeamID`, `Position`, `Price`, `TotalPoints`                   |
| `Teams`       | `TeamID`, `Name`, `ShortName`, `Strength`                                          |
| `Fixtures`    | `FixtureID`, `HomeTeamID`, `AwayTeamID`, `Kickoff`, `GW`, `HomeGoals`, `AwayGoals` |
| `Gameweeks`   | `GW`, `DeadlineTime`, `AverageScore`, `HighestScore`                               |
| `Predictions` | `FixtureID`, `PredictedResult`, `Confidence`, `ModelVersion`                       |

---

Would you like me to **generate the Azure SQL Database schema (CREATE TABLE scripts)** and **FastAPI project scaffold** next — so you can start coding with Codex?
