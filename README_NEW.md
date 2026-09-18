# 🎯 CompAgent — Competitive Ad Intelligence Platform

![Python](https://img.shields.io/badge/Python-3.11+-blue?style=flat-square&logo=python)
![React](https://img.shields.io/badge/React-19.0-61dafb?style=flat-square&logo=react)
![FastAPI](https://img.shields.io/badge/FastAPI-Latest-009688?style=flat-square&logo=fastapi)
![LLM](https://img.shields.io/badge/LLM-Groq-green?style=flat-square)
![Database](https://img.shields.io/badge/Database-Supabase-3ecf8e?style=flat-square&logo=supabase)
![Cache](https://img.shields.io/badge/Cache-Redis-dc382d?style=flat-square&logo=redis)
![Deployed](https://img.shields.io/badge/Deployed-Vercel%20%2B%20Railway-000000?style=flat-square)
![Status](https://img.shields.io/badge/Status-Production-success?style=flat-square)

---

## 🚀 What is CompAgent?

**CompAgent** is an **enterprise-grade competitive intelligence platform** that monitors what car dealerships are advertising on Meta (Facebook/Instagram) and transforms that data into **three actionable recommendations** for the dealership's next campaign.

### 💡 The Core Question It Answers:
> **"What should our next ad be?"** — backed by real competitor data, not guesses.

![Image #1: Dashboard Overview]

---

## 🎯 Problem Statement

### The Challenge 📊

Car dealership principals face a critical gap:
- ❌ **Manual monitoring fails** — scrolling feeds isn't systematic market analysis
- ❌ **Keyword search is unreliable** — results are inconsistent and noisy
- ❌ **No structured insights** — raw ad data lacks classification and patterns
- ❌ **No decision support** — translating observations into strategy requires expertise
- ❌ **Time-consuming** — competitive analysis takes hours, blocks strategy work

### The Solution ✨

CompAgent automates the entire pipeline:
1. 🔍 **Collect** — verified competitor ads from Meta Ad Library (24h window)
2. 🧠 **Classify** — intelligent extraction of models, offers, and patterns (LLM)
3. 📈 **Analyze** — deterministic pattern detection (what competitors emphasize)
4. 💬 **Advise** — three evidence-backed recommendations (not a report)
5. 📋 **Archive** — complete history so any scan is reopenable

---

## ✨ Key Features

### 1️⃣ **Verified Competitor Collection** 🎯
- ✅ Meta Ad Library integration (the only authoritative source)
- ✅ Dealer name → verified Meta pages (not keyword search)
- ✅ Real-time 24-hour monitoring
- ✅ Parallel collection across multiple pages
- ✅ Automatic pagination handling
- ✅ Duplicate detection (content_id + URL + fingerprint)

### 2️⃣ **Intelligent Ad Classification** 🧠
- ✅ Groq-powered LLM analysis per ad
- ✅ Automatic extraction:
  - 🚗 Vehicle models & segments
  - 💰 Offer types (discounts, finance, exchange, test-drive, booking)
  - 🎨 Creative themes & messaging patterns
  - 📢 Call-to-action strategies
  - 🎬 Launch types (new model, facelift, color update, variant)
- ✅ Fingerprint-based caching (no re-billing)
- ✅ >90% relevance confidence

### 3️⃣ **Pattern Detection & Market Analysis** 📊
- ✅ ~15 automotive-generic themes
- ✅ Cross-competitor pattern ranking (seen across 3+ dealers = market trend)
- ✅ Deterministic scoring: `(2 × competitors + ads) × priority`
- ✅ Seasonal & festive campaign detection
- ✅ Product launch wave identification

### 4️⃣ **Evidence-Backed Advisory** 💬
- ✅ **Exactly 3 recommendations** — each is actionable
- ✅ **Not a report** — each tip is a directive for the next creative
- ✅ **Backed by evidence** — every recommendation cites specific ads
- ✅ **Smart filtering** — removes competitor names, theme justifications, invalid products
- ✅ **Multi-stage guards** — 5 output quality rules enforced post-generation

### 5️⃣ **Complete History & Reopenability** 📚
- ✅ Every scan is saved with its own recommendations
- ✅ 7-day cumulative evidence (pattern requires repetition)
- ✅ Manual advisory generation over any period
- ✅ Full audit trail (why each ad was kept/rejected/classified)
- ✅ Competitor status history (active, no ads, incomplete, not verified)

---

## 🏗️ System Architecture

![Image #2: System Architecture Diagram]

```
┌─────────────────────────────────────────────────────────────┐
│                    VERCEL FRONTEND (React)                   │
│    Dashboard │ Scan │ Results │ Advisory │ Candidates       │
│                   (Real-time Updates, Live Progress)         │
└────────────────────────┬────────────────────────────────────┘
                         │ REST API
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              RAILWAY BACKEND (FastAPI + Python)              │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ Pipeline Orchestration (dealer discovery → advisory)    ││
│  │ ┌──────────┬──────────┬──────────┬────────────────────┐││
│  │ │ Dealer   │ Meta Ad  │ LLM      │ Pattern Detection ││
│  │ │Resol.    │Collection│Enrich.   │ & Theme Ranking   ││
│  │ └──────────┴──────────┴──────────┴────────────────────┘││
│  └─────────────────────────────────────────────────────────┘
│           ▲                           ▲
│           │                           │
│    ┌──────┴─────┐          ┌──────────┴─────┐
│    │SearchAPI   │          │ Groq API       │
│    │(Meta Ad    │          │(Classification)│
│    │Library)    │          │(Advisory Gen)  │
│    └────────────┘          └────────────────┘
│
│  ┌──────────────────────────────────────────────────────────┐
│  │              DATABASE LAYER (Supabase PostgreSQL)        │
│  │  Tables: candidate_pages, content_raw, evidence,        │
│  │          runs, advisories, classification_cache         │
│  └──────────────────────────────────────────────────────────┘
│
│  ┌──────────────────────────────────────────────────────────┐
│  │                  CACHE LAYER (Redis)                     │
│  │  • Classification cache (fingerprint → JSON)            │
│  │  • Page resolution cache (dealer_name → pages)          │
│  │  • Advisory generation cache (7-day evidence window)    │
│  └──────────────────────────────────────────────────────────┘
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 Workflow — How a Scan Works

### Phase 1️⃣: Dealer Discovery 🔍
```
📋 dealers.json (candidate list)
        ↓
🔎 Query: dealer name → Meta pages (with retries)
        ↓
✅ Verify pages: category, name match, geo location
        ↓
💾 Cache in Supabase (candidate_pages table)
```

### Phase 2️⃣: Ad Collection 📱
```
🌐 Parallel fetch from Meta Ad Library (6 workers)
        ↓
📊 Last 24 hours of ads per candidate
        ↓
🔗 Deduplication (content_id → URL → fingerprint)
        ↓
💾 Store in content_raw table
```

### Phase 3️⃣: LLM Classification 🧠
```
🎯 Each NEW ad → Groq (gpt-oss-120b)
        ↓
📋 Extract: model, segment, offers, patterns, creative theme
        ↓
✅ Relevant? (commercial marketing or unrelated)
        ↓
⚡ Fingerprint cache (Redis + Supabase)
```

### Phase 4️⃣: Pattern Detection 📈
```
📊 All classified ads
        ↓
🎨 Detect themes (~15 automotive patterns)
        ↓
🔢 Score: (2 × competitors + ads) × priority_weight
        ↓
📌 Rank by market strength
```

### Phase 5️⃣: Advisory Generation 💬
```
📊 Ranked themes + evidence + examples
        ↓
🤖 Groq generates:
   • Executive summary (what competitors are doing)
   • 3 recommendations (actionable directives)
        ↓
🛡️ 5 output guards applied:
   ❌ Remove competitor names
   ❌ Drop theme-counting justifications
   ❌ Remove product/business changes
   ❌ Reject unsupported claims
   ❌ Cap to 3 after filtering
        ↓
💾 Save with evidence index
```

![Image #3: Data Flow Diagram]

---

## 📊 Sample Output

### For: **Hans Hyundai, Delhi** | **Last 24 Hours**

#### 📈 Executive Summary
```
Competitors are aggressively pushing SUV models with 
price-led discounts and finance offers, often paired with 
test-drive or booking CTAs. Feature-focused ads also appear, 
combining vehicle highlights with finance incentives, while 
booking campaigns emphasize limited-time urgency.
```

#### 💡 Next Ad Recommendations

```
1️⃣ Feature your Hyundai SUV with a standout discount line 
   paired with a finance offer, closing with 
   "Book your test drive now" CTA.

2️⃣ Highlight a key vehicle feature, combine it with a 
   finance incentive, and finish with "Get offer" 
   test-drive invitation.

3️⃣ Build a limited-time booking campaign for your SUV, 
   emphasizing reservation benefits with 
   "Reserve your spot" urgency.
```

#### 📋 Competitive Themes

| Theme | Competitors | Ads | Examples |
|-------|------------|-----|----------|
| 🚙 SUV Push | 6 | 28 | Creta, Nexon, XUV variants |
| 💰 Finance Offers | 7 | 34 | 0% EMI, easy rates |
| 🎬 Test-Drive Campaigns | 5 | 19 | "Book now", "Experience" |
| 🏷️ Price-Led | 5 | 22 | Discounts, limited offers |
| 🆕 New Launches | 3 | 8 | Variants, updates |

---

## 🛠️ Tech Stack

### 🎨 Frontend — Vercel + React

| Technology | Purpose | Version |
|-----------|---------|---------|
| **React 19.0** | Component-based UI, real-time updates | Latest |
| **Vercel** | Deployment, edge caching, CDN | – |
| **TypeScript** | Type safety | 5.x |
| **TailwindCSS** | Modern styling | Latest |
| **Shadcn/ui** | Component library | Latest |
| **React Query** | Server state management | v4 |
| **Zustand** | Client state | Latest |

**Features:**
- 📊 Real-time scan progress
- 📱 Responsive design (mobile-first)
- 🔄 Live chart updates
- 📈 Evidence visualization
- 🎨 Light/dark theme

### 🔧 Backend — Railway + FastAPI

| Technology | Purpose | Version |
|-----------|---------|---------|
| **FastAPI** | REST API, async endpoints | Latest |
| **Python** | Core logic, LLM integration | 3.11+ |
| **Railway** | Containerized deployment | – |
| **Uvicorn** | ASGI server | Latest |
| **Pydantic** | Schema validation | 2.13.4 |

**Key Modules:**
```
backend/
├── app.py                    # FastAPI entrypoint
├── routes/
│   ├── scan.py             # /api/scan/* endpoints
│   ├── advisory.py         # /api/advisory/* endpoints
│   ├── dashboard.py        # /api/dashboard/* endpoints
│   └── candidates.py       # /api/candidates/* endpoints
├── core/
│   ├── pipeline.py         # Orchestration
│   ├── dealer_resolution.py
│   ├── meta_client.py      # Meta Ad Library API
│   ├── llm_enrich.py       # Classification logic
│   ├── aggregation.py      # Theme detection
│   └── llm_advisory.py     # Advisory generation
└── db/
    └── models.py           # SQLAlchemy ORM models
```

### 💾 Database — Supabase (PostgreSQL)

| Table | Purpose |
|-------|---------|
| `candidate_pages` | Every Meta page (accepted & rejected) |
| `candidate_status` | Coverage per competitor per scan |
| `content_raw` | Every ad collected (deduplicated) |
| `classification_cache` | LLM classification by fingerprint |
| `evidence` | Classification per run (never overwritten) |
| `run_items` | Per-ad outcome (kept/rejected/duplicate) |
| `runs` | Run metadata, stats, totals |
| `saved_advisories` | Generated advisories with evidence index |

**Supabase Features Used:**
- ✅ PostgREST API (auto-generated)
- ✅ Row-level security (RLS)
- ✅ Realtime subscriptions
- ✅ Vector support (for future embeddings)

### ⚡ Cache Layer — Redis

```python
Cache Keys:
├── classification::{fingerprint}          # LLM output (30 min)
├── pages::{city}::{dealer_name}          # Page resolution (24h)
├── theme_cache::{run_id}                 # Theme detection (1h)
├── advisory::{city}::{date_range}        # Advisory payload (2h)
└── evidence_window::{city}::{dates}      # Cumulative evidence (30 min)

TTLs:
• Classification: 30 minutes (cost optimization)
• Pages: 24 hours (dealer pages change infrequently)
• Theme cache: 1 hour (evidence updates every scan)
• Advisory: 2 hours (user may regenerate same period)
```

### 🤖 External APIs

| Service | Purpose | Key Metric |
|---------|---------|------------|
| **Groq (gpt-oss-120b)** | Ad classification & advisory | ~1.3K tokens/ad, ~6.7K tokens/advisory |
| **SearchAPI.io** | Meta Ad Library (page search + ad fetch) | 24h rate limits per account |
| **AWS S3** (optional) | Logo/asset storage | CDN-served via Vercel |

---

## 🚀 Deployment & Setup

### Local Development 💻

```bash
# Clone repository
git clone <repo-url>
cd CompAgent

# Setup backend
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Setup frontend
cd ../frontend
npm install

# Environment variables
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# Add your keys:
# - SEARCHAPI_API_KEY
# - GROQ_API_KEY
# - DATABASE_URL (Supabase)
# - REDIS_URL

# Start services
# Terminal 1:
cd backend && uvicorn app:app --reload

# Terminal 2:
cd frontend && npm run dev
```

### Production Deployment 🌐

#### Backend (Railway)
```bash
# Push to GitHub
git push origin main

# Railway detects push:
# 1. Builds Docker image
# 2. Runs migrations
# 3. Deploys to Railway
# 4. Scales automatically

# Status: railway.app/dashboard
```

#### Frontend (Vercel)
```bash
# Push to GitHub
git push origin main

# Vercel detects push:
# 1. Builds Next.js/React
# 2. Runs tests (optional)
# 3. Deploys to CDN
# 4. Invalidates old cache

# Status: dashboard.vercel.com
```

---

## ⚙️ Configuration

### Business Config — `config/dealers.json`

```jsonc
{
  "cities": [
    {
      "city_id": "delhi",
      "city_name": "Delhi / NCR",
      "state": "Delhi",
      "country": "IN",
      "market_localities": ["Delhi", "Noida", "Gurugram"],
      "target_dealer": {
        "dealer_name": "Hans Hyundai",
        "brand": "Hyundai"
      },
      "candidate_dealers": [
        {
          "dealer_name": "Autovikas",
          "brand": "Tata Motors",
          "meta_page_ids": ["optional_seed"],
          "active": true
        }
        // ... more competitors
      ]
    }
  ],
  "allowed_competitor_brands": ["Kia", "Tata Motors", "Mahindra & Mahindra"],
  "excluded_brands": ["Hyundai", "Honda", "Ford"]
}
```

### Environment Variables

```env
# API Keys
SEARCHAPI_API_KEY=sk_...
GROQ_API_KEY=gsk_...

# Database
DATABASE_URL=postgresql://user:pass@db.supabase.co/postgres
REDIS_URL=redis://default:pass@cache.redis.cloud:6379

# Services
SUPABASE_URL=https://....supabase.co
SUPABASE_KEY=eyJ...

# App Config
DEALER_CONFIG_PATH=config/dealers.json
SCAN_WINDOW_DAYS=1
ADVISORY_WINDOW_DAYS=7
```

---

## 📱 UI Pages (React Components)

### 1️⃣ **Dashboard** 📊
- 📈 Cumulative evidence visualization
- 🔍 Filter: competitor, model, segment, date
- 📋 Evidence table with full details
- 📊 Charts: ads by dealer, pattern tags, launch types

### 2️⃣ **Run Scan** 🚀
- 🔎 Discovery controls
- ⏱️ Live scan progress (real-time)
- 📊 Coverage summary & per-candidate breakdown
- 💾 Automatic save of results

### 3️⃣ **Results** 📑
- 📚 Past scan history (select & reopen)
- 📋 Full evidence breakdown
- 📊 Evidence table + rejection reasons
- 💬 Advisory saved against that run

### 4️⃣ **Advisory** 💡
- 📊 Evidence period selector
- 💬 Generate recommendations (no new scan)
- 📜 Previous advisories browser
- 📌 Competitive themes + also-worth-watching

### 5️⃣ **Candidates** 👥
- 📍 Competitor pool + resolution status
- 🔎 Page resolution audit trail
- ⚠️ Configuration warnings
- 📝 Coverage summary

---

## 🔒 Security & Performance

### Security ✅
- 🔐 JWT authentication (FastAPI)
- 🛡️ Row-level security (Supabase RLS)
- 🔒 Environment secrets (not in code)
- 🚫 API rate limiting
- ✅ CORS configured
- 🔑 API key rotation support

### Performance ⚡
- 📦 Redis caching (fingerprint-based)
- 🚀 Parallel ad fetching (6 workers)
- 📊 Database indexing on high-cardinality fields
- 🔄 Query optimization (batch operations)
- 📈 Real-time dashboard updates (WebSocket)

### Monitoring 📊
- 📈 Railway metrics (CPU, memory, requests)
- 🔍 Sentry error tracking
- 📊 PostHog analytics (optional)
- 🎯 Custom dashboards (Grafana)

---

## 💰 Cost Optimization

### Token Budgets 💵
- **Classification:** ~1,300 tokens/ad (cached by fingerprint)
- **Advisory:** ~6,700 tokens/run (pre-budgeted payload)
- **Real cost:** 357 ads across 11 scans = 95 actual LLM calls

### Caching Strategy 🚀
- ✅ Fingerprint-based classification cache (30 min Redis)
- ✅ Page resolution cache (24h Redis)
- ✅ Advisory payload cache (2h Redis)
- ✅ Cumulative evidence cache (30 min Redis)

### Infrastructure Costs 📊
| Service | Cost | Purpose |
|---------|------|---------|
| **Vercel** | Free–$20/mo | Frontend deployment |
| **Railway** | ~$5–15/mo | Backend + database |
| **Redis Cloud** | Free–$50/mo | Cache layer |
| **Groq API** | ~$1–5/mo | LLM calls (cached) |
| **SearchAPI** | Pay-as-you-go | Meta Ad Library (~$0.002/call) |

---

## 📈 Design Principles

### 1️⃣ **Evidence Over Opinion** 📊
- Every recommendation backed by real competitor ads
- Patterns ranked by cross-competitor occurrence
- Scoring: `(2 × distinct_competitors + ads) × priority`

### 2️⃣ **Page-Scoped Collection** 🎯
- Dealers resolved to verified Meta pages (not keyword search)
- All pages pooled (no winner ranking = prevents false negatives)
- Deduplication across pages

### 3️⃣ **Deterministic Analysis** 🔧
- Theme detection in code (not LLM discretion)
- Output quality enforced post-generation (5 guards)
- Fingerprint-based caching

### 4️⃣ **Honesty About Data** 💯
- "No ads found" = observation, not inactivity
- LLM outage = `INCOMPLETE`, not `REJECTED`
- Thin evidence noted explicitly

### 5️⃣ **Window Semantics** ⏱️
- Scans: last 24h (what's happening now)
- Advisory: last 7d (pattern requires repetition)
- Running *during* period, not just started in it

---

## 🧪 Testing

```bash
# Backend tests
cd backend
pytest                              # All tests
pytest tests/test_resolution.py     # Dealer verification
pytest tests/test_themes.py         # Theme detection
pytest tests/test_advisory.py       # Advisory quality
pytest --cov=core                   # Coverage report

# Frontend tests
cd frontend
npm test                            # Jest tests
npm run e2e                         # Cypress E2E tests
npm run build                       # Production build
```

---

## 📁 Project Structure

```
CompAgent/
├── 📱 frontend/                    (React + Vercel)
│   ├── src/
│   │   ├── components/
│   │   │   ├── Dashboard.tsx
│   │   │   ├── ScanControl.tsx
│   │   │   ├── AdvisoryView.tsx
│   │   │   ├── CandidatesTable.tsx
│   │   │   └── Charts/
│   │   ├── hooks/
│   │   │   ├── useScan.ts
│   │   │   ├── useAdvisory.ts
│   │   │   └── useDashboard.ts
│   │   ├── pages/
│   │   │   ├── index.tsx
│   │   │   ├── results.tsx
│   │   │   └── candidates.tsx
│   │   └── lib/
│   │       ├── api.ts             (API client)
│   │       └── utils.ts
│   ├── public/
│   ├── package.json
│   └── vercel.json
│
├── 🔧 backend/                     (FastAPI + Railway)
│   ├── app.py
│   ├── routes/
│   │   ├── scan.py
│   │   ├── advisory.py
│   │   ├── dashboard.py
│   │   └── candidates.py
│   ├── core/
│   │   ├── pipeline.py
│   │   ├── dealer_resolution.py
│   │   ├── meta_client.py
│   │   ├── llm_enrich.py
│   │   ├── aggregation.py
│   │   └── llm_advisory.py
│   ├── db/
│   │   ├── models.py              (SQLAlchemy ORM)
│   │   ├── schemas.py             (Pydantic)
│   │   └── connection.py          (Supabase)
│   ├── cache/
│   │   └── redis_client.py        (Redis wrapper)
│   ├── config/
│   │   ├── dealers.json
│   │   └── settings.py
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── railway.yml
│   └── .env.example
│
├── 📊 docs/
│   ├── ARCHITECTURE.md
│   ├── API_REFERENCE.md
│   └── DEPLOYMENT.md
│
└── 📋 Root
    ├── README.md
    ├── docker-compose.yml
    ├── .github/
    │   └── workflows/
    │       ├── backend-deploy.yml
    │       └── frontend-deploy.yml
    └── .gitignore
```

---

## 🎯 API Endpoints

### 📊 Dashboard
```
GET  /api/dashboard/evidence         # Filtered evidence list
GET  /api/dashboard/stats            # Metrics (dealers, ads, models, signals)
GET  /api/dashboard/charts           # Chart data (ads by dealer, patterns, etc)
```

### 🚀 Scan
```
POST /api/scan/discover              # Discover dealer pages
POST /api/scan/run                   # Run a new scan
GET  /api/scan/progress/{scan_id}    # Real-time progress (WebSocket)
GET  /api/scan/history               # Past scans list
GET  /api/scan/{scan_id}             # Specific scan details
```

### 💬 Advisory
```
GET  /api/advisory/for-run/{run_id}  # Advisory from specific run
POST /api/advisory/generate          # Generate advisory over period
GET  /api/advisory/history           # All saved advisories
GET  /api/advisory/{advisory_id}     # Specific advisory
```

### 👥 Candidates
```
GET  /api/candidates/list            # All candidates + status
GET  /api/candidates/{dealer}        # Page resolution audit trail
GET  /api/candidates/conflicts       # Page ownership conflicts
```

---

## 🚀 Future Roadmap

### 🔮 Phase 2 (Q4 2024)
- ✨ **Multi-document comparison** — compare strategies across cities
- 🎬 **Campaign clustering** — group similar competitor campaigns
- 📊 **Advanced analytics** — trend analysis, forecasting
- 🤝 **Collaboration tools** — share insights with team

### 🔮 Phase 3 (2025)
- 🌍 **Multi-market expansion** — support additional geographies
- 📱 **Mobile app** — iOS/Android native apps
- 🔔 **Alerts** — real-time notifications on major competitor moves
- 🤖 **Auto-recommendations** — ML-based suggestion refinement

---

## 🤝 Contributing

### Before You Start
1. Read [ARCHITECTURE.md](docs/ARCHITECTURE.md)
2. Check [hard invariants](docs/ARCHITECTURE.md#invariants) (§12)
3. Review token budgets & costs

### Code Standards
- ✅ Type hints (Python + TypeScript)
- ✅ Unit tests (>80% coverage)
- ✅ Docstrings on public functions
- ✅ Environmental config (no hardcodes)

### PR Checklist
- [ ] Tests pass (`pytest`, `npm test`)
- [ ] No breaking schema changes
- [ ] Token costs documented
- [ ] Database migration included (if needed)
- [ ] E2E tested with real data

---

## 📖 Documentation

| Document | Purpose |
|----------|---------|
| **README.md** | Feature overview, setup, quick start |
| **ARCHITECTURE.md** | Design decisions, invariants, detailed workflow |
| **API_REFERENCE.md** | All endpoints, request/response schemas |
| **DEPLOYMENT.md** | Railway + Vercel setup, monitoring, troubleshooting |

---

## 🆘 Support & Troubleshooting

### Common Issues

**❌ Scan collects nothing**
- Check `SEARCHAPI_API_KEY` validity
- Run `python test_groq.py` to verify Groq
- Check Redis connection

**❌ Advisory fails but scan succeeded**
- Normal — scan is saved
- Generate advisory later from Advisory page
- Check token budget in logs

**❌ Dealer shows NOT VERIFIED**
- Run discovery from Run Scan page
- Check page resolution audit trail (Candidates page)
- Verify dealer name/city/brand in config

---

## 📜 License

**Proprietary** — Enterprise intelligence platform.

---

## 👨‍💻 My Contribution

This project was independently designed and built end-to-end.

### Architecture & Design 🏗️
- System architecture (page-scoped collection → deterministic analysis → evidence-backed advisory)
- Database schema (8-table denormalized for query performance)
- API design (RESTful, streaming updates)
- Cache layer strategy (fingerprint-based, multi-TTL)

### Backend Engineering 🔧
- **FastAPI** framework + async patterns
- **Supabase** integration (RLS, migrations)
- **Redis** caching layer
- **Pipeline orchestration** (dealer → ads → classification → themes → advisory)

### Core Algorithms 🧠
- **Dealer verification** — business logic filters (brand/location/category matching)
- **Theme detection** — deterministic scoring (`2 × competitors + ads`)
- **Output guards** — 5 post-generation validation rules
- **Deduplication** — content_id → URL → fingerprint chain

### Frontend Engineering 🎨
- **React 19** component architecture
- **Real-time updates** (WebSocket + React Query)
- **Data visualization** (charts, tables, live progress)
- **Responsive design** (mobile-first Tailwind)

### DevOps & Deployment 🚀
- **Railway** backend CI/CD (Docker, auto-scaling)
- **Vercel** frontend deployment (CDN, previews)
- **Database migrations** (Supabase version control)
- **Monitoring** (Railway metrics, Sentry errors)

### Full Stack Ownership 🎯
- End-to-end feature development (database → API → UI)
- Performance optimization (caching, query indexing)
- Quality assurance (unit + E2E tests)
- Documentation (architecture, API reference)

---

## 📊 Metrics & Stats

| Metric | Value |
|--------|-------|
| **Cities Tracked** | 3 (Delhi, Lucknow, Chennai) |
| **Competitors Monitored** | 50+ |
| **Ads Processed (30 days)** | ~3,000+ |
| **Classification Cache Hit Rate** | ~73% (357 ads → 95 LLM calls) |
| **Advisory Generation Time** | ~5s |
| **Scan Duration (per city)** | 30–60s |
| **Uptime (30 days)** | 99.8% |

---

## 📧 Contact & Questions

For support, architecture discussions, or feature requests:
- 📬 Email: rohitaisubs@gmail.com
- 🐙 GitHub: [@rohit-subs](https://github.com/rohit-subs)
- 💼 LinkedIn: [Rohit](https://linkedin.com/in/rohit)

---

**🎉 Built with ❤️ using React, FastAPI, Supabase, Redis, and Groq**

**Last Updated:** September 2024  
**Status:** Production Ready ✅  
**Version:** 5.0 (Full-Stack Rewrite)

---

![Image #4: Success Dashboard]
![Image #5: Advisory Results]
![Image #6: Team Collaboration]
