
<p align="center">
  <img src="assets/agentbay-logo.png" alt="AgentBay Logo" width="120" />
</p>

<h1 align="center">AgentBay</h1>

<p align="center">
  <strong>A2A-Powered Multi-Platform E-Commerce System</strong>
</p>

<p align="center">
  <a href="https://www.youtube.com/watch?v=wEN63wCB_js">🎬 Watch the Demo</a>
</p>

<p align="center">
  Built by <strong>Sanskar Thapa</strong>, <strong>Damarrion (DIIZZY) Morgan-Harper</strong>, and <strong>Joshua Soteras</strong>
</p>

---

An intelligent e-commerce orchestration platform that uses **Google ADK** agents to automatically create, manage, and negotiate product listings across multiple marketplaces (eBay and Tetsy).

## Table of Contents

- [Features](#features)
- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Running the Application](#running-the-application)
- [Usage](#usage)
- [API Endpoints](#api-endpoints)
- [Database Schema](#database-schema)
- [Technology Stack](#technology-stack)
- [Multi-Agent System](#multi-agent-system)
- [Data Flow](#data-flow)
- [System Architecture](#system-architecture)
- [eBay Integration Notes](#ebay-integration-notes)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

---

## Features

> Skillfully using Google's Agent 2 Agent (A2A) Protocol

- **Intelligent Agent Orchestration** — Multi-agent system with specialized agents for each platform
- **AI-Powered Product Analysis** — Upload product images and automatically extract details using Google Gemini AI
- **Multi-Platform Listing** — Seamlessly create listings on eBay and Tetsy from a single interface
- **Automated Price Negotiation** — AI agents handle buyer-seller negotiations on Tetsy with intelligent pricing strategies
- **Real-Time Dashboard** — WebSocket-powered live updates of all listings
- **Unified Management** — Track and manage inventory across all platforms in one place

## Architecture Overview

This project uses a multi-tier architecture with:

| Layer | Technology |
|-------|------------|
| Agent System | Google ADK Multi-Agent System (Gemini 2.5-Flash) |
| Frontend | React, TypeScript, Tailwind CSS, Shadcn UI |
| Backend | FastAPI, Python, WebSocket support |
| Database | SQLite (Listings and negotiations) |
| Integrations | eBay Sandbox API, Tetsy API |

## Project Structure

```
AgentBay/
├── backend/                    # Main API server (Port 8000)
│   ├── apis.py                # Primary API routes and WebSocket
│   ├── db.py                  # SQLite database helpers
│   ├── ebay_api.py            # eBay sandbox integration (Port 8001)
│   ├── requirements.txt       # Python dependencies
│   └── offersb.db             # SQLite database
│
├── front/                      # React frontend (Port 5173)
│   ├── src/
│   │   ├── pages/             # Dashboard and CreateListing pages
│   │   ├── components/        # Reusable UI components
│   │   ├── lib/               # API client and utilities
│   │   └── types/             # TypeScript type definitions
│   └── package.json
│
├── specialty_agents/           # Google ADK agent system
│   ├── my_agent/              # Root orchestrator (Port 10000)
│   ├── ebay_agent/            # eBay specialist (Port 10002)
│   └── tetsy_agent/           # Tetsy specialist (Port 10001)
│
└── Tetsy/                      # Tetsy platform implementation
    ├── backend/               # Negotiation API (Port 8050)
    └── frontend/              # Buyer-seller interface
```

## Prerequisites

- **Python** 3.8+
- **Node.js** 18+
- **Google API Key** (for Gemini AI)
- **eBay Developer Account** (for eBay integration)

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/sskarz/AgentBay.git
cd AgentBay
```

### 2. Backend Setup

```bash
cd backend
pip install -r requirements.txt
```

Create a `.env` file in the `backend/` directory:

```env
GOOGLE_API_KEY=your_google_api_key_here
EBAY_CLIENT_ID=your_ebay_client_id
EBAY_CLIENT_SECRET=your_ebay_client_secret
EBAY_REDIRECT_URI=http://localhost:8001/oauth/callback
```

### 3. Frontend Setup

```bash
cd front
npm install
```

### 4. Agent Setup

The agents use Google ADK and share the same `requirements.txt` from the backend:

```bash
cd specialty_agents/my_agent
pip install -r ../../backend/requirements.txt
```

## Running the Application

You need to start **7 services** in separate terminal windows:

| Terminal | Service | Command | Port |
|----------|---------|---------|------|
| 1 | Main Backend API | `cd backend && python apis.py` | 8000 |
| 2 | eBay Backend | `cd backend && uvicorn ebay_api:app --reload --port 8001` | 8001 |
| 3 | Tetsy Backend | `cd Tetsy/backend && python main.py` | 8050 |
| 4 | Root Orchestrator Agent | `cd specialty_agents/my_agent && python -m __main__` | 10000 |
| 5 | Tetsy Agent | `cd specialty_agents/tetsy_agent && python -m __main__` | 10001 |
| 6 | eBay Agent | `cd specialty_agents/ebay_agent && python -m __main__` | 10002 |
| 7 | Frontend | `cd front && npm run dev` | 5173 |

## Usage

### Creating a Listing

1. Navigate to the **Create Listing** page
2. **Upload Product Image** — Drag & drop or click to upload
3. **AI Analysis** — Gemini automatically extracts product details (name, price, description, brand)
4. **Review & Edit** — Confirm or modify the extracted details
5. **Select Platform** — Choose eBay or Tetsy
6. **Submit** — The orchestrator agent routes to the appropriate platform agent
7. **View Dashboard** — See your listing appear in real-time

### Managing Listings

- **Dashboard** — View all listings across platforms with live updates
- **Status Tracking** — Monitor listing status (Draft, Pending, Live, Sold)
- **Platform Filtering** — See which platform each listing is on (color-coded)

### Handling Negotiations (Tetsy)

The Tetsy agent automatically handles buyer offers with intelligent pricing:

| Offer Range | Agent Response |
|-------------|---------------|
| ≥ 85% of asking price | Accept the offer |
| < 85% of asking price | Counter at 90% |
| Very low offers | Reject politely |

All negotiations are tracked in the Tetsy backend database.

## API Endpoints

### Main Backend (Port 8000)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | API information |
| `/health` | GET | Health check |
| `/api/analyze-product-image` | POST | AI image analysis (Gemini) |
| `/api/stream` | WebSocket | Real-time listing updates |
| `/api/create-listing-with-agent` | POST | Create listing via agent system |
| `/api/add_item` | POST | Direct database insertion |

### eBay Backend (Port 8001)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/start-auth` | GET | Initiate eBay OAuth |
| `/oauth/callback` | GET | OAuth redirect handler |
| `/publish` | POST | Publish listing to eBay |
| `/create-all-policies` | POST | Create business policies |

### Tetsy Backend (Port 8050)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/negotiations` | GET | Get all negotiations |
| `/api/negotiations/{id}` | GET | Get negotiation details |
| `/api/negotiations` | POST | Start new negotiation |
| `/api/negotiations/{id}/messages` | POST | Send message/offer |

## Database Schema

### Main Database (`offersb.db`)

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT,
    platform TEXT NOT NULL,
    price REAL NOT NULL,
    high_price REAL,              -- Highest offer received
    status TEXT,                   -- 'listed', 'sold', 'pending'
    quantity INTEGER,
    imageSrc BLOB,                -- Product image bytes
    createdAt TIMESTAMP,
    updatedAt TIMESTAMP
);
```

### Tetsy Database (`negotiations.db`)

```sql
CREATE TABLE negotiations (
    id TEXT PRIMARY KEY,
    product_id TEXT,
    buyer_id TEXT,
    seller_id TEXT,
    status TEXT,                   -- 'pending', 'accepted', 'rejected', 'counter'
    last_offer_amount REAL,
    created_at TIMESTAMP
);

CREATE TABLE messages (
    id TEXT PRIMARY KEY,
    negotiation_id TEXT,
    sender_id TEXT,
    sender_type TEXT,              -- 'buyer', 'seller'
    type TEXT,                     -- 'message', 'offer', 'counter_offer'
    offer_amount REAL
);
```

## Technology Stack

### Backend

| Technology | Purpose |
|------------|---------|
| FastAPI | Modern Python web framework |
| Uvicorn | ASGI server |
| SQLite | Embedded database |
| WebSockets | Real-time communication |
| Google Generative AI | Gemini 2.0-flash for image analysis |
| Google ADK | Multi-agent orchestration framework |
| Httpx | Async HTTP client |
| Pydantic | Data validation |

### Frontend

| Technology | Purpose |
|------------|---------|
| React 19 | UI library |
| TypeScript | Type-safe JavaScript |
| Vite | Build tool and dev server |
| Tailwind CSS | Utility-first CSS |
| Shadcn UI | Component library (Radix UI + Tailwind) |
| React Router | Client-side routing |
| React Hook Form | Form management |
| Zod | Schema validation |
| Axios | HTTP client |
| Sonner | Toast notifications |

### AI / Agents

| Technology | Purpose |
|------------|---------|
| Google ADK | Agent development kit |
| Gemini 2.5-Flash | LLM for agent intelligence |
| RemoteA2aAgent | Agent-to-agent communication pattern |

## Multi-Agent System

### Root Orchestrator Agent (Port 10000)

- **Purpose:** Routes listing requests to appropriate platform agents
- **Model:** Gemini 2.5-Flash
- **Sub-agents:** `tetsy_agent`, `ebay_agent`

### eBay Agent (Port 10002)

- **Purpose:** Publishes listings to eBay Sandbox
- **Tools:**
  - `publish_to_ebay()` — Creates inventory items and offers
  - Database integration for tracking

### Tetsy Agent (Port 10001)

- **Purpose:** Manages Tetsy listings and handles negotiations
- **Tools:**
  - `post_listing_to_tetsy()` — Creates listings
  - `check_tetsy_notifications()` — Monitors offers
  - `respond_to_negotiation()` — Intelligent offer handling
- **Strategy:**
  - Accept offers ≥ 85% of asking price
  - Counter at 90% for lower offers
  - Professional communication with buyers

## Data Flow

```
User uploads image → Gemini AI analysis → Frontend form
    ↓
User confirms details + selects platform → API call
    ↓
Main Backend /api/create-listing-with-agent
    ↓
Root Agent (Gemini 2.5-Flash) decides routing
    ↓
    ├─→ eBay Agent → eBay API → eBay Sandbox
    │   └─→ Database save
    │
    └─→ Tetsy Agent → Tetsy Backend → Tetsy Database
        └─→ Database save
            ↓
WebSocket broadcasts update → All connected dashboards refresh
```

## System Architecture

<details>
<summary><strong>Complete System Architecture Diagram</strong> (click to expand)</summary>

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USER INTERFACE LAYER                            │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │              React Frontend (Port 5173)                               │  │
│  │  ┌─────────────────────┐      ┌──────────────────────────────────┐  │  │
│  │  │  CreateListing Page │      │      Dashboard Page              │  │  │
│  │  │  - Image Upload     │      │  - Real-time listing view        │  │  │
│  │  │  - AI Analysis      │      │  - Status tracking               │  │  │
│  │  │  - Platform Select  │      │  - Platform filtering            │  │  │
│  │  └─────────────────────┘      └──────────────────────────────────┘  │  │
│  │               │                            ▲                          │  │
│  │               │ HTTP POST                  │ WebSocket                │  │
│  │               │ (FormData)                 │ (real-time)              │  │
│  └───────────────┼────────────────────────────┼──────────────────────────┘  │
└─────────────────┼────────────────────────────┼─────────────────────────────┘
                  │                            │
                  ▼                            │
┌─────────────────────────────────────────────────────────────────────────────┐
│                          MAIN BACKEND LAYER (Port 8000)                      │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                     FastAPI Application (apis.py)                      │  │
│  │                                                                         │  │
│  │  POST /api/analyze-product-image                                       │  │
│  │  ┌────────────────────────────────────────────────────────┐           │  │
│  │  │ 1. Receive image upload (multipart/form-data)          │           │  │
│  │  │ 2. Encode to base64                                    │           │  │
│  │  │ 3. Send to Gemini 2.0-flash-exp                        │           │  │
│  │  │ 4. Parse JSON response                                 │           │  │
│  │  │ 5. Extract: name, description, price, brand            │           │  │
│  │  │ 6. Return product details to frontend                  │           │  │
│  │  └────────────────────────────────────────────────────────┘           │  │
│  │                                                                         │  │
│  │  POST /api/create-listing-with-agent                                   │  │
│  │  ┌────────────────────────────────────────────────────────┐           │  │
│  │  │ 1. Receive: name, description, price,                  │           │  │
│  │  │             quantity, brand, platform                   │           │  │
│  │  │ 2. Create Runner with root_agent                       │           │  │
│  │  │ 3. Invoke agent with prompt                            │           │  │
│  │  │ 4. Collect agent response                              │           │  │
│  │  │ 5. Return response to frontend                         │           │  │
│  │  └────────────────────────────────────────────────────────┘           │  │
│  │                                                                         │  │
│  │  WebSocket /api/stream                                                 │  │
│  │  ┌─────────────────────────────────────────┐                          │  │
│  │  │ 1. Accept WebSocket connection          │                          │  │
│  │  │ 2. Every 2s: query DB → send JSON       │                          │  │
│  │  └─────────────────────────────────────────┘                          │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│         ┌───────────────┐    ┌─────────────┐                                │
│         │  SQLite DB     │    │  Gemini AI  │                                │
│         │ (offersb.db)   │    │  (Google)   │                                │
│         └───────────────┘    └─────────────┘                                │
└─────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   │ Invokes via Runner
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       MULTI-AGENT ORCHESTRATION LAYER                        │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │              Root Orchestrator Agent (Port 10000)                      │  │
│  │              Model: Gemini 2.5-Flash (Google ADK)                      │  │
│  │                                                                         │  │
│  │  Sub-agents:                                                            │  │
│  │  - tetsy_agent (RemoteA2aAgent, localhost:10001)                       │  │
│  │  - ebay_agent (RemoteA2aAgent, localhost:10002)                        │  │
│  └───────────────────────────┬──────────────────┬──────────────────────────┘  │
└──────────────────────────────┼──────────────────┼────────────────────────────┘
                               │                  │
              ┌────────────────┘                  └────────────────┐
              ▼                                                    ▼
┌──────────────────────────────────────┐    ┌──────────────────────────────────┐
│    Tetsy Agent (Port 10001)          │    │    eBay Agent (Port 10002)       │
│    Model: Gemini 2.5-Flash           │    │    Model: Gemini 2.5-Flash       │
│                                      │    │                                  │
│  Tools:                              │    │  Tools:                          │
│  - post_listing_to_tetsy()           │    │  - publish_to_ebay()             │
│  - check_tetsy_notifications()       │    │                                  │
│  - respond_to_negotiation()          │    │  Publishes to eBay Sandbox       │
│                                      │    │  via eBay Backend (Port 8001)    │
│  Posts to Tetsy Backend (Port 8050)  │    │                                  │
└──────────────────────────────────────┘    └──────────────────────────────────┘
```

</details>

<details>
<summary><strong>Detailed Flow Sequences</strong> (click to expand)</summary>

### Flow 1: Create Listing via AI Analysis

```
User uploads image
    ↓
CreateListing Page (React Frontend)
    ↓ POST /api/analyze-product-image (multipart/form-data)
Main Backend API (Port 8000)
    ↓ Send image + prompt
Google Generative AI (gemini-2.0-flash-exp)
    ↓ Returns: { name, description, price, brand, quantity }
Main Backend API → Parse JSON → Return to frontend
    ↓
CreateListing Page → Pre-fill form → User reviews → User submits
    ↓
[Continue to Flow 2]
```

### Flow 2: Agent-Based Listing Creation

```
CreateListing Page
    ↓ POST /api/create-listing-with-agent (name, description, price, quantity, brand, platform)
Main Backend API (Port 8000)
    ↓ Invoke agent via ADK Runner
Root Orchestrator Agent (Port 10000) — Gemini 2.5-Flash
    ↓ Routes based on platform
    ├─→ Tetsy Agent (Port 10001) → [Flow 3]
    └─→ eBay Agent (Port 10002)  → [Flow 4]
```

### Flow 3: Tetsy Platform Listing

```
Tetsy Agent
    ↓ Calls post_listing_to_tetsy()
    ├─→ POST http://localhost:8050/api/listings → Tetsy DB (negotiations.db)
    └─→ POST http://localhost:8000/api/add_item → Main DB (offersb.db)
            ↓ WebSocket broadcasts every 2s
        All Dashboard clients update
```

### Flow 4: eBay Platform Listing

```
eBay Agent
    ↓ Calls publish_to_ebay()
    ├─→ POST http://localhost:8001/publish
    │       ↓ 1. Validate OAuth token
    │       ↓ 2. Create inventory item
    │       ↓ 3. Create & publish offer
    │       ↓ 4. Return listing URL
    └─→ POST http://localhost:8000/api/add_item → Main DB (offersb.db)
```

### Flow 5: Tetsy Buyer Negotiation

```
Buyer makes offer ($40 on a $50 item)
    ↓ POST /api/negotiations
Tetsy Backend (Port 8050) → Creates negotiation record
    ↓ Webhook: POST http://localhost:10001/webhook/message
Tetsy Agent — Decision Logic (Gemini 2.5-Flash):
    │ offer_percentage = 40/50 = 80%
    │ 80% < 85% threshold → Counter at 90% ($45)
    ↓ Calls respond_to_negotiation(type="counter", amount=45)
Tetsy Backend → Updates negotiation → Buyer sees counter-offer
```

### Flow 6: WebSocket Real-Time Updates

```
Dashboard clients connect → ws://localhost:8000/api/stream
    ↓
Main Backend (every 2 seconds):
    → Query SQLite DB (SELECT * FROM users)
    → Convert BLOB images to base64
    → Send JSON array to all connected clients
    ↓
Dashboard re-renders with updated listings, status badges, platform indicators
```

</details>

### Port Allocation

| Port | Service | Technology | Purpose |
|------|---------|------------|---------|
| 5173 | React Frontend | Vite | User interface |
| 8000 | Main Backend API | FastAPI | Core orchestration & WebSocket |
| 8001 | eBay Integration Backend | FastAPI | eBay Sandbox API integration |
| 8050 | Tetsy Backend | FastAPI | Negotiation & listing management |
| 10000 | Root Orchestrator Agent | Google ADK | Multi-agent routing |
| 10001 | Tetsy Agent | Google ADK | Tetsy operations & negotiation |
| 10002 | eBay Agent | Google ADK | eBay listing publication |

### Component Dependencies

```
Frontend (React)
    ↓
    ├─→ Main Backend API ──→ SQLite DB (offersb.db)
    │       ↓
    │       ├─→ Google Gemini (image analysis)
    │       └─→ Root Agent ──→ Google ADK
    │               ├─→ Tetsy Agent
    │               │       ├─→ Tetsy Backend ──→ SQLite DB (negotiations.db)
    │               │       └─→ Main Backend (save listing)
    │               └─→ eBay Agent
    │                       ├─→ eBay Backend ──→ eBay Sandbox API
    │                       └─→ Main Backend (save listing)
    └─→ Tetsy Backend (for buyer negotiations)
            └─→ Tetsy Agent (webhook for auto-response)
```

## eBay Integration Notes

- Uses **eBay Sandbox** environment for testing
- Requires **OAuth 2.0** authentication flow
- **Business policies** (fulfillment, payment, return) are mandatory
- SKU format: alphanumeric only (no special characters)
- Category-specific product attributes required

### eBay Setup Steps

1. Visit `/start-auth` to initiate OAuth
2. Complete authorization on eBay
3. Run `/optin-to-business-policies`
4. Create policies via `/create-all-policies`
5. Now ready to publish listings

## Troubleshooting

<details>
<summary><strong>Agents not connecting</strong></summary>

- Ensure all backend services are running first
- Check that ports 8000, 8001, 8050, 10000–10002 are available
- Verify `GOOGLE_API_KEY` is set in `.env`

</details>

<details>
<summary><strong>eBay authentication fails</strong></summary>

- Check eBay credentials in `.env`
- Ensure redirect URI matches eBay app settings
- Verify you're using sandbox credentials for sandbox environment

</details>

<details>
<summary><strong>WebSocket disconnects</strong></summary>

- Check browser console for errors
- Ensure main backend (port 8000) is running
- Frontend auto-reconnects every 3 seconds

</details>

<details>
<summary><strong>Image analysis fails</strong></summary>

- Verify `GOOGLE_API_KEY` is valid
- Check image format is supported (JPEG, PNG)
- Ensure image file size is reasonable (< 10MB)

</details>

## Development

### Running Tests

```bash
# Backend tests
cd backend && pytest

# Frontend tests
cd front && npm test
```

### Code Style

```bash
# Frontend linting
cd front && npm run lint
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

This project was developed for **CalHacks 2025**.

## Acknowledgments

- Built with [Google ADK](https://google.github.io/adk-docs/) and Gemini AI
- [eBay Developers Program](https://developer.ebay.com/) for sandbox access
- [Shadcn UI](https://ui.shadcn.com/) for beautiful components
- [FastAPI](https://fastapi.tiangolo.com/) and [React](https://react.dev/) communities
