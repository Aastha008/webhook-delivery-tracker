# Webhook Delivery Tracker 🚀

![Python Version](https://img.shields.io/badge/python-3.11%2B-blue?style=for-the-badge&logo=python)
![Framework](https://img.shields.io/badge/FastAPI-0.110%2B-009688?style=for-the-badge&logo=fastapi)
![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)
![Code Style](https://img.shields.io/badge/code%20style-black-000000?style=for-the-badge)
![Tests](https://img.shields.io/badge/tests-passing-brightgreen?style=for-the-badge&logo=pytest)

**Webhook Delivery Tracker** is a self-hosted, lightweight, full-stack application designed to ingest, inspect, log, and retry incoming and outgoing HTTP webhooks. Built with an asynchronous Python core and an interactive dashboard, it provides end-to-end observability into transient webhook failures, HMAC signature validation errors, and event payload histories.

---

## 📋 Table of Contents

- [Problem Statement & Target Audience](#-problem-statement--target-audience)
- [Key Features](#-key-features)
- [Architecture & System Design](#-architecture--system-design)
- [Technology Stack](#-technology-stack)
- [Installation & Setup](#-installation--setup)
- [Environment Variables](#-environment-variables)
- [Usage & API Reference](#-usage--api-reference)
- [Automated Testing](#-automated-testing)
- [Project Structure](#-project-structure)
- [Roadmap & Future Enhancements](#-roadmap--future-enhancements)
- [License](#-license)

---

## 🛑 Problem Statement & Target Audience

### The Problem
Modern microservices rely heavily on third-party HTTP webhooks (e.g., Stripe, GitHub, Shopify, SendGrid). However, third-party API integrations frequently fail due to:
- **Transient HTTP network errors** (timeouts, 5xx server errors).
- **Invalid or mismatched HMAC signatures** leading to silent request drops.
- **Service downtime** on target destinations during high-traffic bursts.

Without an dedicated proxy and delivery ledger, developers suffer from zero visibility into missed events and lack a straightforward mechanism to inspect headers, format raw JSON payloads, or re-execute failed webhooks manually.

### Target Audience
- **Backend & Integration Engineers** building or consuming third-party webhooks.
- **DevOps Engineers** seeking self-hosted webhook observability without expensive SaaS tools.
- **API Developers** debugging signature verification algorithms and payload schemas.

---

## ✨ Key Features

- 🔐 **Endpoint Registration & Secret Management**: Register inbound listening endpoints and outbound forwarding targets with per-endpoint HMAC-SHA256 secret keys for secure signature validation.
- 📥 **Payload Ingestion & History Logging**: Capture incoming HTTP POST requests in real-time. Log raw body payloads, incoming HTTP headers, client IP metadata, and destination response statuses.
- 🖥️ **Interactive Inspection Dashboard**: Search, filter, and inspect delivery logs by HTTP status code, provider slug, date range, or destination URL using an integrated Tailwind CSS interface with formatted JSON syntax highlighting.
- 🔄 **Automated & Manual Retries**: Configure exponential backoff algorithms for transient HTTP failures (`502`, `503`, `504`, `408`) and trigger manual single-click redeliveries directly from the web UI.

---

## 🏗️ Architecture & System Design

The system acts as a transparent, high-performance async proxy and inspection pipeline:

```text
                                  +---------------------------------------+
                                  |         Inbound Webhook Issuer        |
                                  |   (e.g., Stripe, GitHub, Custom Provider)|
                                  +------------------+--------------------+
                                                     |
                                            HTTP POST Payload
                                                     v
                                  +------------------+--------------------+
                                  |     FastAPI Ingestion Engine          |
                                  |   - Route Matching                    |
                                  |   - Body & Header Extraction          |
                                  +------------------+--------------------+
                                                     |
                                                     v
                                  +------------------+--------------------+
                                  |    HMAC-SHA256 Verification Engine   |
                                  |   - Computes Expected Digest          |
                                  |   - Constant-Time Comparison          |
                                  +------------------+--------------------+
                                                     |
                                   +-----------------+-----------------+
                                   |                                   |
                             Signature Valid                     Signature Invalid
                                   |                                   |
                                   v                                   v
                    +--------------+---------------+   +---------------+---------------+
                    | Outbound Forwarding Dispatcher|   |  Mark Delivery Log as FAILED  |
                    | - HTTPX Async Client         |   |  Reason: Invalid HMAC Secret  |
                    | - Exponential Backoff Loop   |   +---------------+---------------+
                    +--------------+---------------+                   |
                                   |                                   |
                                   +-----------------+-----------------+
                                                     |
                                                     v
                                  +------------------+--------------------+
                                  |   Async SQLite Storage Engine         |
                                  |   - Deliveries, Headers, Endpoints   |
                                  +------------------+--------------------+
                                                     |
                                                     v
                                  +------------------+--------------------+
                                  |   Tailwind CSS / JS Web Dashboard     |
                                  |   - Log Search & Filtering            |
                                  |   - Payload Formatting & Redelivery   |
                                  +---------------------------------------+
```

### Module Responsibilities

| Module | Location | Responsibilities |
| :--- | :--- | :--- |
| **Models & Schemas** | `src/models.py` | SQLAlchemy ORM database models and Pydantic v2 schemas for Endpoints, Deliveries, and Request Headers. |
| **Database Engine** | `src/database.py` | Async SQLAlchemy database engine, session factory initialization, and schema creation routines. |
| **Core Engine** | `src/core.py` | HMAC-SHA256 cryptographic signature verification, HTTP dispatching, and exponential backoff retry execution. |
| **API & Router** | `src/api.py` | FastAPI REST endpoints for endpoints CRUD, payload ingestion, log searching, manual retries, and dashboard HTML rendering. |
| **Application Entry** | `src/main.py` | FastAPI application factory, static asset mounting, database lifecycle events, and Uvicorn server launcher. |

---

## 🛠️ Technology Stack

| Category | Technologies |
| :--- | :--- |
| **Language & Runtime** | Python 3.11+ |
| **Backend Framework** | FastAPI, Starlette |
| **Database & ORM** | Async SQLite (`aiosqlite`), SQLAlchemy 2.0 (Async ORM) |
| **Validation & Parsing** | Pydantic v2 |
| **HTTP Client Engine** | HTTPX (Async HTTP Client) |
| **Frontend & UI** | HTML5, JavaScript (ES6+), Jinja2 Templates, Tailwind CSS (CDN) |
| **Testing Suite** | pytest, pytest-asyncio, HTTPX AsyncClient |

---

## 📥 Installation & Setup

### Prerequisites
- **Python 3.11+** installed on your system.
- `git` for repository cloning.

### 1. Clone the Repository
```bash
git clone https://github.com/your-org/webhook-delivery-tracker.git
cd webhook-delivery-tracker
```

### 2. Create and Activate Virtual Environment
```bash
# On Linux / macOS:
python3 -m venv venv
source venv/bin/activate

# On Windows (PowerShell):
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### 3. Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## ⚙️ Environment Variables Configuration

Copy the example environment file to create your active `.env` configuration:

```bash
cp .env.example .env
```

### Supported Configuration Options (`.env`)

| Variable Name | Default Value | Description |
| :--- | :--- | :--- |
| `PROJECT_NAME` | `"Webhook Delivery Tracker"` | Application branding name displayed in UI/logs. |
| `DEBUG` | `True` | Enables detailed debug logging and FastAPI reloader. |
| `HOST` | `"127.0.0.1"` | Host address for Uvicorn server binding. |
| `PORT` | `8000` | Port number for server binding. |
| `DATABASE_URL` | `"sqlite+aiosqlite:///./webhooks.db"` | Async SQLAlchemy database connection string. |
| `MAX_RETRIES` | `3` | Maximum automatic retries for failed outbound attempts. |
| `INITIAL_BACKOFF_SECONDS` | `2.0` | Initial delay multiplier for exponential backoff. |

---

## 🚀 Usage Guide

### Starting the Application Server

Launch the application using Uvicorn or the provided module launcher:

```bash
python -m src.main
```
Or directly via `uvicorn`:
```bash
uvicorn src.main:app --reload --host 127.0.0.1 --port 8000
```
Access the interactive dashboard at **`http://localhost:8000/dashboard`** and OpenAPI docs at **`http://localhost:8000/docs`**.

---

### REST API Endpoints Overview

#### 1. Register a Target Endpoint
```bash
curl -X POST "http://localhost:8000/api/v1/endpoints" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Payment Gateway Forwarder",
    "provider_slug": "stripe-payments",
    "target_url": "https://api.yourdomain.com/v1/webhooks/stripe",
    "secret": "super_secret_hmac_key_123"
  }'
```

#### 2. Ingest an Incoming Webhook Event
```bash
curl -X POST "http://localhost:8000/api/v1/ingest/stripe-payments" \
  -H "Content-Type: application/json" \
  -H "X-Signature-256: 8f9b4c02741d402120e2e21bca96e1b0c0361fb5597bf2987179047971764662" \
  -d '{
    "event": "payment_intent.succeeded",
    "amount": 4900,
    "currency": "usd"
  }'
```

#### 3. Search Delivery History
```bash
curl -X GET "http://localhost:8000/api/v1/deliveries?status_code=200&provider_slug=stripe-payments"
```

#### 4. Trigger Manual Payload Redelivery
```bash
curl -X POST "http://localhost:8000/api/v1/deliveries/1/retry"
```

---

## 🧪 Automated Testing Instructions

The project uses `pytest` alongside `pytest-asyncio` for full integration and unit test coverage.

### Run All Test Suites
```bash
pytest
```

### Run Core Signature & Dispatcher Tests Only
```bash
pytest tests/test_core.py -v
```

### Run API Route & Endpoint Integration Tests
```bash
pytest tests/test_api.py -v
```

### Produce Test Coverage Reports
```bash
pytest --cov=src --cov-report=term-missing
```

---

## 📂 Project Structure

```text
webhook-delivery-tracker/
├── .env.example          # Sample environment variables configuration
├── .gitignore             # Git exclusion rules for Python artifacts & SQLite DBs
├── requirements.txt       # Project dependencies listing
├── src/
│   ├── __init__.py        # Python package initialization
│   ├── api.py             # FastAPI REST endpoints & Jinja2 UI view controllers
│   ├── core.py            # HMAC verification, HTTP client, & backoff executor
│   ├── database.py        # Async SQLAlchemy engine and session initialization
│   ├── main.py            # Application entrypoint & Uvicorn runner
│   └── models.py          # SQLAlchemy ORM models & Pydantic schemas
└── tests/
    ├── __init__.py        # Test package marker
    ├── test_api.py        # Integration tests for FastAPI endpoints
    └── test_core.py       # Unit tests for core HMAC engine and retries
```

---

## 🔮 Roadmap & Future Enhancements

- [ ] **Real-time Live Tailing**: Integrate WebSocket support for live streaming incoming webhook events directly into the dashboard UI.
- [ ] **Distributed Queue Integration**: Support Celery / Redis Streams as an alternative execution backend for heavy backoff retry queues.
- [ ] **Slack & Discord Alerts**: Send instant notifications to alerting channels when delivery retries exceed maximum threshold counts.
- [ ] **Multi-Tenant API Keys**: Implement role-based access control (RBAC) and tenant API key auth for enterprise usage.

---

## 📜 License

Distributed under the **MIT License**. See `LICENSE` for more information.

---

<p align="center">
  <i>Developed with ❤️ by the Daily Project Factory team.</i>
</p>