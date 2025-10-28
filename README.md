# Sibyl
# **FinOps Command Center (MVP \- Visibility Engine)**

## **Purpose**

This repository contains the Minimum Viable Product (MVP) for the **FinOps Command Center**, an internal platform designed to bring clarity, attribution, and basic compliance monitoring to cloud spending. This initial version focuses solely on establishing core visibility – answering the fundamental questions: "Who is spending what?" and "Are our resources tagged correctly?"

This MVP is the first step towards the larger vision outlined in the [Full Project Blueprint]((./project-blueprint.md).

## **MVP Scope: The "Visibility Engine"**

This MVP delivers foundational capabilities focused on **AWS only**, specifically:

1. **Pillar 1: Data Ingestion (Minimal)**  
   * Ingests **AWS Cost and Usage Report (CUR)** data (assuming Parquet format from S3).  
   * Connects to **AWS CloudWatch** for basic operational metrics (for future correlation, though not visualized in MVP).  
   * *Deferred:* Connectors for business metrics, financial planning data, other cloud providers, or other metric sources.  
2. **Pillar 2: Cost Attribution (Core)**  
   * Implements a **Business Hierarchy Model** (Team \> BU \> BG) via database tables.  
   * Includes a simple **Service Catalog** mapping resources/tags to internal services/teams.  
   * Provides a **Tagging Integration Rules Engine** to normalize tags and enrich cost data for accurate attribution.  
3. **Pillar 3: Compliance (Finder Only)**  
   * Includes an **Untagged Resource Finder** that scans AWS (via API or processed CUR data) to identify resources missing required tags.  
   * *Deferred:* Compliance scorecards, showback/chargeback mechanisms.  
4. **Pillar 6: The "Cockpit" (Basic Dashboard API)**  
   * Provides API endpoints to deliver data for a simple "Cost by Team (Last 30 Days)" view.  
   * Includes calculation logic for Month-over-Month (MoM) trend analysis for this specific view.  
   * *Deferred:* UI dashboard, advanced forecasting, unit economics, budget reconciliation, optimization recommendations.

## **Tech Stack**

* **Backend:** Python (FastAPI) \- Excellent for data processing, API performance, and async operations.  
* **Frontend:** React with TypeScript \- Component reusability and type safety.  
* **Database:** PostgreSQL \- Superior for analytical queries and JSONB support.  
* **Data Processing:** Pandas (within Python backend for CUR processing).  
* **ORM:** SQLAlchemy \- Robust ORM for interacting with PostgreSQL.  
* **Migrations:** Alembic \- Handles database schema changes.  
* **Task Queue:** Celery with Redis \- For asynchronous processing of large CUR files and enrichment tasks.  
* **Caching:** Redis \- For caching frequently accessed data or intermediate results.  
* **Containerization:** Docker (via docker-compose.yml) \- For consistent development and deployment environments.  
* **API Documentation:** Auto-generated via FastAPI (Swagger UI/ReDoc).

## **Project Structure**

finops-command-center/  
├── backend/  
│   ├── app/  
│   │   ├── \_\_init\_\_.py  
│   │   ├── main.py             \# FastAPI entrypoint  
│   │   ├── config.py           \# Settings management (Pydantic)  
│   │   ├── database.py         \# SQLAlchemy setup  
│   │   ├── models/             \# SQLAlchemy ORM models  
│   │   │   ├── \_\_init\_\_.py  
│   │   │   ├── organization.py   \# BusinessGroup, BusinessUnit, Team  
│   │   │   ├── service.py        \# Service model  
│   │   │   ├── resource.py       \# CloudResource, TagRule models  
│   │   │   ├── cost.py           \# CostData model  
│   │   │   └── metric.py         \# MetricData model  
│   │   ├── schemas/            \# Pydantic data validation schemas  
│   │   │   ├── \_\_init\_\_.py  
│   │   │   ├── organization.py  
│   │   │   ├── service.py  
│   │   │   └── cost.py  
│   │   ├── api/                \# FastAPI routers/endpoints  
│   │   │   ├── \_\_init\_\_.py  
│   │   │   ├── v1/  
│   │   │   │   ├── \_\_init\_\_.py  
│   │   │   │   ├── organization.py \# Org hierarchy CRUD  
│   │   │   │   ├── cost.py         \# Pillar 6 Dashboard API  
│   │   │   │   └── compliance.py   \# Pillar 3 Untagged API  
│   │   ├── core/               \# Business logic for Pillars  
│   │   │   ├── \_\_init\_\_.py  
│   │   │   ├── pillar1\_ingestion/  
│   │   │   │   ├── \_\_init\_\_.py  
│   │   │   │   ├── cur\_processor.py      \# CUR S3 download & Pandas parsing  
│   │   │   │   └── cloudwatch\_connector.py \# CloudWatch metric fetching  
│   │   │   ├── pillar2\_attribution/  
│   │   │   │   ├── \_\_init\_\_.py  
│   │   │   │   ├── tag\_enrichment.py    \# Tag normalization & linking  
│   │   │   │   └── service\_catalog.py   \# Service mapping logic  
│   │   │   ├── pillar3\_compliance/  
│   │   │   │   ├── \_\_init\_\_.py  
│   │   │   │   └── untagged\_finder.py   \# AWS API scanning & DB checks  
│   │   │   └── pillar6\_cockpit/  
│   │   │       ├── \_\_init\_\_.py  
│   │   │       └── dashboard\_service.py \# Logic for dashboard data aggregation  
│   │   └── tasks/              \# Celery background tasks  
│   │       ├── \_\_init\_\_.py  
│   │       └── celery\_tasks.py     \# e.g., async CUR processing, enrichment  
│   ├── alembic/              \# Database migration scripts  
│   │   └── versions/  
│   ├── tests/                \# Unit and integration tests  
│   ├── requirements.txt      \# Python dependencies  
│   └── .env.example          \# Example environment variables  
├── frontend/                 \# React frontend application  
│   ├── src/  
│   │   ├── components/  
│   │   ├── pages/  
│   │   ├── services/         \# API client  
│   │   ├── types/            \# TypeScript types  
│   │   └── App.tsx  
│   ├── package.json  
│   └── tsconfig.json  
└── docker-compose.yml        \# Docker setup for services (backend, db, redis, celery)

## **Database Schema (PostgreSQL)**

*(High-level overview. See backend/models/ and backend/alembic/ for detailed definitions and migration scripts.)*

\-- Pillar 2: Organization Hierarchy  
CREATE TABLE business\_groups (  
    id SERIAL PRIMARY KEY,  
    name VARCHAR(255) NOT NULL UNIQUE,  
    description TEXT,  
    created\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    updated\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP  
);  
CREATE TABLE business\_units (  
    id SERIAL PRIMARY KEY,  
    name VARCHAR(255) NOT NULL,  
    business\_group\_id INTEGER REFERENCES business\_groups(id) ON DELETE CASCADE,  
    description TEXT,  
    created\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    updated\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    UNIQUE(name, business\_group\_id)  
);  
CREATE TABLE teams (  
    id SERIAL PRIMARY KEY,  
    name VARCHAR(255) NOT NULL,  
    business\_unit\_id INTEGER REFERENCES business\_units(id) ON DELETE CASCADE,  
    owner\_email VARCHAR(255),  
    slack\_channel VARCHAR(255),  
    created\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    updated\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    UNIQUE(name, business\_unit\_id)  
);

\-- Pillar 2: Service Catalog  
CREATE TABLE services (  
    id SERIAL PRIMARY KEY,  
    name VARCHAR(255) NOT NULL UNIQUE,  
    description TEXT,  
    team\_id INTEGER REFERENCES teams(id) ON DELETE SET NULL,  
    service\_type VARCHAR(100), \-- e.g., 'API', 'Web', 'Worker', 'Database'  
    repository\_url VARCHAR(500),  
    documentation\_url VARCHAR(500),  
    created\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    updated\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP  
);

\-- Pillar 1 & 3: Cloud Resources (discovered state)  
CREATE TABLE cloud\_resources (  
    id SERIAL PRIMARY KEY,  
    resource\_id VARCHAR(255) NOT NULL, \-- AWS resource ID  
    resource\_type VARCHAR(100), \-- e.g., 'EC2', 'RDS', 'S3'  
    resource\_arn VARCHAR(500),  
    region VARCHAR(50),  
    account\_id VARCHAR(50),  
    tags JSONB, \-- Store all tags as JSON  
    service\_id INTEGER REFERENCES services(id) ON DELETE SET NULL,  
    team\_id INTEGER REFERENCES teams(id) ON DELETE SET NULL,  
    is\_tagged BOOLEAN DEFAULT FALSE,  
    compliance\_score DECIMAL(5,2), \-- 0-100 score  
    discovered\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    last\_seen TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    UNIQUE(resource\_id, account\_id)  
);  
\-- Add GIN index on tags for efficient querying  
CREATE INDEX idx\_cloud\_resources\_tags ON cloud\_resources USING GIN (tags);  
CREATE INDEX idx\_cloud\_resources\_team ON cloud\_resources(team\_id);  
CREATE INDEX idx\_cloud\_resources\_service ON cloud\_resources(service\_id);

\-- Pillar 1: Cost Data (from CUR)  
CREATE TABLE cost\_data (  
    id BIGSERIAL PRIMARY KEY,  
    line\_item\_id VARCHAR(255),  
    usage\_start\_date TIMESTAMP NOT NULL,  
    usage\_end\_date TIMESTAMP NOT NULL,  
    account\_id VARCHAR(50),  
    resource\_id VARCHAR(255),  
    service\_name VARCHAR(100), \-- AWS service name (EC2, S3, etc.)  
    usage\_type VARCHAR(255),  
    operation VARCHAR(255),  
    region VARCHAR(50),  
    cost DECIMAL(18,6),  
    usage\_amount DECIMAL(18,6),  
    usage\_unit VARCHAR(50),  
    currency VARCHAR(10) DEFAULT 'USD',  
    tags JSONB,  
    \-- Attribution (from Pillar 2\)  
    team\_id INTEGER REFERENCES teams(id) ON DELETE SET NULL,  
    service\_id INTEGER REFERENCES services(id) ON DELETE SET NULL,  
    business\_unit\_id INTEGER REFERENCES business\_units(id) ON DELETE SET NULL,  
    business\_group\_id INTEGER REFERENCES business\_groups(id) ON DELETE SET NULL,  
    is\_attributed BOOLEAN DEFAULT FALSE,  
    created\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP  
);  
\-- Add indexes on dates, team\_id, service\_id, resource\_id, tags  
CREATE INDEX idx\_cost\_data\_dates ON cost\_data(usage\_start\_date, usage\_end\_date);  
CREATE INDEX idx\_cost\_data\_team ON cost\_data(team\_id);  
CREATE INDEX idx\_cost\_data\_service ON cost\_data(service\_id);  
CREATE INDEX idx\_cost\_data\_resource ON cost\_data(resource\_id);  
CREATE INDEX idx\_cost\_data\_tags ON cost\_data USING GIN (tags);

\-- Pillar 1: CloudWatch Metrics  
CREATE TABLE metric\_data (  
    id BIGSERIAL PRIMARY KEY,  
    resource\_id VARCHAR(255),  
    metric\_name VARCHAR(255) NOT NULL,  
    namespace VARCHAR(255),  
    timestamp TIMESTAMP NOT NULL,  
    value DECIMAL(18,6),  
    unit VARCHAR(50),  
    dimensions JSONB,  
    \-- Attribution  
    team\_id INTEGER REFERENCES teams(id) ON DELETE SET NULL,  
    service\_id INTEGER REFERENCES services(id) ON DELETE SET NULL,  
    created\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP  
);  
\-- Add indexes on resource\_id, timestamp, service\_id  
CREATE INDEX idx\_metric\_data\_resource ON metric\_data(resource\_id);  
CREATE INDEX idx\_metric\_data\_timestamp ON metric\_data(timestamp);  
CREATE INDEX idx\_metric\_data\_service ON metric\_data(service\_id);

\-- Pillar 2: Tag Normalization Rules  
CREATE TABLE tag\_rules (  
    id SERIAL PRIMARY KEY,  
    rule\_name VARCHAR(255) NOT NULL UNIQUE,  
    tag\_key VARCHAR(255) NOT NULL,  
    pattern VARCHAR(500), \-- Regex pattern for matching  
    normalized\_key VARCHAR(255), \-- What to normalize to  
    priority INTEGER DEFAULT 0, \-- Higher priority rules run first  
    is\_active BOOLEAN DEFAULT TRUE,  
    created\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    updated\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP  
);

\-- Pillar 3: Compliance Tracking & Untagged Resources  
CREATE TABLE compliance\_scans (  
    id SERIAL PRIMARY KEY,  
    scan\_date TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,  
    total\_resources INTEGER,  
    tagged\_resources INTEGER,  
    untagged\_resources INTEGER,  
    compliance\_percentage DECIMAL(5,2),  
    scan\_duration\_seconds INTEGER,  
    status VARCHAR(50) \-- 'completed', 'failed', 'in\_progress'  
);  
CREATE TABLE untagged\_resources (  
    id SERIAL PRIMARY KEY,  
    scan\_id INTEGER REFERENCES compliance\_scans(id) ON DELETE CASCADE,  
    resource\_id VARCHAR(255),  
    resource\_type VARCHAR(100),  
    resource\_arn VARCHAR(500),  
    account\_id VARCHAR(50),  
    region VARCHAR(50),  
    monthly\_cost DECIMAL(18,6), \-- Estimate based on recent cost\_data  
    missing\_tags TEXT\[\], \-- Array of required tags that are missing  
    discovered\_at TIMESTAMP DEFAULT CURRENT\_TIMESTAMP  
);  
\-- Add indexes on scan\_id, monthly\_cost  
CREATE INDEX idx\_untagged\_resources\_scan ON untagged\_resources(scan\_id);  
CREATE INDEX idx\_untagged\_resources\_cost ON untagged\_resources(monthly\_cost DESC);

## **Key Backend Components (MVP Focus)**

* **config.py**: Manages environment variables (DB URL, AWS keys, S3 bucket, Redis URL, required tags) using Pydantic's BaseSettings. Caches settings using lru\_cache.  
* **database.py**: Sets up the SQLAlchemy async engine, session factory, and defines the Base for declarative models.  
* **models/**: Defines the database tables using SQLAlchemy ORM (e.g., organization.py defines BusinessGroup, BusinessUnit, Team; cost.py defines CostData). Relationships between tables (e.g., teams to business\_units) are defined here.  
* **schemas/**: Defines Pydantic models for API request/response validation (e.g., CostByTeamResponse, schemas for creating/updating organization entities). These ensure data consistency.  
* **api/v1/cost.py**: FastAPI router providing the endpoint for Pillar 6 (e.g., GET /api/v1/costs/by\_team). Uses dependency injection to get DB session and calls dashboard\_service.  
* **api/v1/compliance.py**: FastAPI router providing the endpoint for Pillar 3 (e.g., GET /api/v1/compliance/untagged or POST /api/v1/compliance/scans). Calls logic in untagged\_finder.  
* **core/pillar1\_ingestion/cur\_processor.py**: Contains the CURProcessor class with methods like list\_cur\_files, download\_and\_parse\_cur (using Pandas), normalize\_cur\_data (mapping columns, extracting tags to JSONB), and bulk\_insert\_costs (using SQLAlchemy's bulk\_insert\_mappings).  
* **core/pillar2\_attribution/tag\_enrichment.py**: Implements the TagEnrichmentEngine class. Loads tag\_rules, builds team\_mapping and service\_mapping dictionaries on initialization. Provides methods to normalize\_tags, extract\_team\_id, extract\_service\_id, and enrich\_cost\_data (iterating through CostData records and updating attribution fields).  
* **core/pillar3\_compliance/untagged\_finder.py**: Contains the UntaggedResourceFinder class. Uses Boto3 clients (ec2, rds, s3) to scan resources (\_scan\_ec2\_instances, etc.), checks their tags against REQUIRED\_TAGS from settings, potentially estimates monthly\_cost from recent cost\_data, and saves findings to untagged\_resources table, linked to a compliance\_scans record.  
* **core/pillar6\_cockpit/dashboard\_service.py**: Holds the business logic for the dashboard API. Queries cost\_data using SQLAlchemy, performs necessary aggregations (group by team, sum cost for last 30 days), calculates MoM trends, and returns data structured according to Pydantic schemas.  
* **tasks/celery\_tasks.py**: Defines Celery tasks decorated with @celery\_app.task. Examples: process\_cur\_file\_task (takes S3 key, calls CURProcessor), enrich\_all\_cost\_data\_task (calls TagEnrichmentEngine), run\_compliance\_scan\_task (calls UntaggedResourceFinder).

## **Getting Started**

1. **Clone the repository:**  
   git clone \<repository-url\>  
   cd finops-command-center

2. **Configure Environment:**  
   * Copy backend/.env.example to backend/.env.  
   * Fill in your specific database credentials (PostgreSQL DATABASE\_URL), AWS\_ACCESS\_KEY\_ID, AWS\_SECRET\_ACCESS\_KEY, S3 bucket name (CUR\_S3\_BUCKET), CUR\_S3\_PREFIX, REDIS\_URL, and required AWS tags (REQUIRED\_TAGS as a comma-separated string like "Team,Service,Environment") in backend/.env.  
3. **Build and Run with Docker Compose:**  
   docker-compose up \--build \-d

   This command builds the Docker images for the backend, frontend (if defined in the docker-compose.yml), database, Redis, and Celery worker, then starts them in detached mode. Check logs with docker-compose logs \-f backend or docker-compose logs \-f worker.  
4. **Run Database Migrations:**  
   docker-compose exec backend alembic upgrade head

   This command executes the Alembic migration scripts (backend/alembic/versions/) inside the running backend container to create or update the database schema defined in your SQLAlchemy models (backend/app/models/).  
5. **Access the API Documentation:**  
   * Navigate to http://localhost:8000/docs (or your configured backend port) in your web browser. FastAPI's automatic Swagger UI will load, showing available API endpoints defined in backend/app/api/v1/ and allowing you to interact with them directly.  
6. **(Optional) Seed Initial Data:** You may need to manually add initial data for business\_groups, business\_units, teams, services, and tag\_rules via SQL, a seeding script, or potentially simple CRUD endpoints added to backend/app/api/v1/organization.py and a new tag\_rules.py router.  
7. **(Optional) Trigger Initial CUR Processing:** You might need an initial script (e.g., using docker-compose exec backend python initial\_load.py) or a temporary API endpoint to trigger the Celery task (process\_cur\_for\_date\_range task, likely defined in backend/app/tasks/celery\_tasks.py) to ingest historical CUR data for a specified date range.  
8. **(Optional) Run Frontend:**  
   * Navigate to the frontend/ directory.  
   * Install dependencies: npm install  
   * Start the development server: npm start (ensure the API base URL configured in your React app's services points to your running backend, e.g., http://localhost:8000).

## **Future Development Beyond MVP**

This MVP focuses strictly on core AWS visibility and attribution via API. Subsequent phases will build upon this foundation by adding:

* A dedicated **Frontend UI** using React/TypeScript to visualize the data provided by the API.  
* **Pillar 4:** The "Proving Ground" for unit cost testing.  
* **Pillar 5:** Predictive Forecasting ("Sibyl") using R or Python stats libraries (like Prophet, Statsmodels).  
* **Pillar 7:** Automated Capacity Plan Generation ("Architect") using templates.  
* **Pillar 8:** Budget Reconciliation module, ingesting data from finance systems.  
* **Pillar 9:** Governance Gantry integrations with deployment tools (Terraform, CloudFormation).  
* Support for **GCP, Azure**, and other data sources (metrics, costs).  
* More sophisticated dashboarding, optimization recommendations (Pillar 6 expansion), and analytics.