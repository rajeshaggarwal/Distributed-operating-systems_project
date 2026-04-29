## Architecture Diagram

```mermaid
flowchart TD

%% Users
U1[Public User]
U2[Admin User]

%% Client Layer
C[Client Device OS Windows macOS Linux Android iOS]
B[Web Browser]

U1 --> C
U2 --> C
C --> B

%% Internet
NET[Internet HTTPS]
B --> NET

%% Web Server Node
subgraph WS[Web Server Node Linux]
    NGINX[Nginx Reverse Proxy]
    FE[React Frontend]
    APP[Django App Gunicorn]
end

NET --> NGINX
NGINX --> FE
NGINX --> APP

%% User Interaction with Frontend
U1 <-->|Submit Enquiry| FE
U2 <-->|Admin Dashboard| FE

%% Backend Services
APP --> API[Django REST API]

FE -->|API Calls| API
FE -->|JWT Auth| AUTH

API --> AUTH[Auth Service]
API --> ENQ[Enquiry Service]
API --> CAT[Catalogue Service]
API --> REP[Report Service]

%% Database Node
subgraph DBNODE[Database Node Linux]
    DB[(PostgreSQL or Supabase)]
end

%% Redis Node
subgraph REDISNODE[Cache Queue Node Linux]
    REDIS[(Redis)]
end

%% Worker Node
subgraph WORKERNODE[Worker Node Linux]
    CELERY[Celery Worker]
    BEAT[Celery Beat]
end

%% Database Connections
AUTH --> DB
ENQ --> DB
CAT --> DB
REP --> DB

%% Redis / Queue
API --> REDIS
CELERY --> REDIS
BEAT --> REDIS

%% Async Processing
API -->|Trigger Tasks| CELERY
BEAT -->|Schedule Jobs| CELERY

%% Reports
CELERY --> REP
REP -->|Send XLSX Report| EMAIL

%% External Services
EMAIL[SMTP Email Server]
SMS[Twilio Service]

ENQ -->|Send OTP Email| EMAIL
ENQ -->|Send OTP SMS| SMS

%% Storage
STORAGE[File Storage]
CAT --> STORAGE

%% Download Flow
ENQ -->|Generate Signed Link| FE
FE -->|Download Request| API
API --> CAT
CAT --> STORAGE
