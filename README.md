# Parakh (परख) — Backend, Authentication & AI/ML Pipeline

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%20%7C%203.11%20%7C%203.12-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/Framework-FastAPI-009688.svg)](https://fastapi.tiangolo.com/)
[![PyTorch](https://img.shields.io/badge/AI%2FML-PyTorch%20%7C%20Transformers-EE4C2C.svg)](https://pytorch.org/)
[![OpenSource Model](https://img.shields.io/badge/Model-DeepGuard%20(MS--EffGCViT)-FF1744.svg)](https://huggingface.co/KoreaPeter/ms-eff-gcvit-deepfake-b0-ff-plus-plus)
[![Security](https://img.shields.io/badge/Auth-Argon2id%20%2B%20JWT-673AB7.svg)](https://tools.ietf.org/html/rfc9106)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Production-ready backend, authentication service, and AI/ML inference integration for **Parakh (परख)**, the deepfake face detection and synthetic media verification platform.

This repository implements the complete backend and machine learning pipeline using the open-source pretrained **DeepGuard** (`KoreaPeter/ms-eff-gcvit-deepfake-b0-ff-plus-plus`) model trained on the **FaceForensics++** benchmark.

---

## 🎯 Scope of Work

As part of the **Parakh** engineering team, this repository provides:
1. **Authentication & Identity**:
   - User registration and login with RFC 9106 **Argon2id** password hashing (`argon2-cffi`).
   - Dual-mode session handling: HttpOnly, SameSite cookies and standard `Authorization: Bearer <token>` for cross-origin and SPA clients.
   - Protected user profile (`/api/v1/auth/me`) and session management.

2. **Backend REST APIs**:
   - High-throughput **FastAPI** service with interactive Swagger/OpenAPI documentation at `/docs`.
   - **SQLAlchemy 2.0** ORM layer with full support for SQLite (development) and PostgreSQL (production).
   - Per-user media detection history, timeline analysis, frame inspection, and resource deletion.
   - CORS middleware pre-configured for local development (`localhost:3000`, `localhost:5173`) and the deployed GitHub Pages frontend (`https://anuragverma0907.github.io`).

3. **AI/ML Open-Source Model Integration**:
   - Seamless integration with the **DeepGuard** Multi-Scale Efficient Global Context Vision Transformer (**MS-EffGCViT**) checkpoint.
   - Automatic YOLOv8 face detection (`ultralytics`), facial cropping, and tensor normalization.
   - Non-blocking asynchronous execution via FastAPI threadpools.
   - Binary classification producing calibrated `fake_probability` scores and categorization verdicts (`likely_manipulated` vs `likely_authentic`).

4. **Video Temporal Analysis**:
   - MP4 decoding via **OpenCV** with configurable frame sampling (default: 12 frames).
   - Frame-by-frame probability scoring and aggregate arithmetic mean calculation.
   - Temporal clustering of suspicious frames to generate approximate manipulation segments.

5. **Evaluation & Robustness Tools**:
   - Offline batch evaluation script (`evaluate_manifest.py`) measuring Accuracy, Precision, Recall, F1, and generating ROC-AUC curves with Matplotlib.
   - Robustness perturbation tester (`robustness_eval.py`) evaluating sensitivity against JPEG compression and Gaussian blur.

6. **Frontend Integration Kits**:
   - Ready-to-use TypeScript client (`parakh-api.ts`) for Next.js 14 (`app/page.jsx`).
   - Vanilla JavaScript ES module client (`parakh-api.js`) for the static GitHub Pages SPA (`app.js`).

---

## 📁 Repository Structure

```
├── deepfake-backend/               # Core FastAPI backend & ML service
│   ├── app/
│   │   ├── cli.py                  # Offline batch command-line scanner
│   │   ├── config.py               # Pydantic environment configuration
│   │   ├── database.py             # SQLAlchemy session and engine
│   │   ├── dependencies.py         # Auth dependency & Bearer/Cookie extractors
│   │   ├── jobs.py                 # Asynchronous video analysis background worker
│   │   ├── main.py                 # FastAPI application routes & CORS middleware
│   │   ├── models.py               # SQLAlchemy database models (User, Detection)
│   │   ├── schemas.py              # Pydantic v2 validation schemas
│   │   ├── security.py             # Argon2id password hasher & JWT tokens
│   │   └── ml/
│   │       ├── media.py            # Image validation and OpenCV video sampling
│   │       ├── model_service.py    # DeepGuard pretrained model adapter & inference
│   │       └── pipeline.py         # Image & video analysis pipelines
│   ├── scripts/
│   │   ├── evaluate_manifest.py    # Benchmark evaluation & ROC curve generator
│   │   └── robustness_eval.py      # JPEG/blur sensitivity testing
│   ├── tests/                      # Automated test suite (Pytest)
│   ├── Dockerfile                  # Container definition
│   ├── docker-compose.yml          # Compose orchestration
│   ├── requirements.txt            # Core backend requirements
│   ├── requirements-ml.txt         # Pretrained ML & PyTorch requirements
│   ├── requirements-eval.txt       # Evaluation & plotting dependencies
│   ├── setup_windows.ps1           # Windows PowerShell setup script
│   └── run_windows.ps1             # Windows start script
├── frontend-integration/           # Client libraries for frontend developers
│   ├── FRONTEND_INTEGRATION.md     # Step-by-step wiring guide for Next.js & Vanilla JS
│   ├── parakh-api.ts               # Typed client for Next.js 14
│   └── parakh-api.js               # ES module client for Vanilla JS / GitHub Pages
├── .gitignore                      # Git ignore patterns
└── README.md                       # Main project documentation
```

---

## 🚀 Quick Start

### 1. Installation (Windows PowerShell)

```powershell
cd deepfake-backend

# Initialize virtual environment and install core backend dependencies
.\setup_windows.ps1

# To install PyTorch and the pretrained model runtime:
.\setup_windows.ps1 -WithML
```

Or manually:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip install -r requirements-ml.txt
Copy-Item .env.example .env
```

### 2. Configure Environment

Edit `deepfake-backend/.env` and generate a secure secret key:

```powershell
python -c "import secrets; print(secrets.token_urlsafe(48))"
```

Paste the generated string into `SECRET_KEY`.

### 3. Start the Server

```powershell
# Using the PowerShell runner
.\run_windows.ps1

# Or directly with Uvicorn
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

- **Interactive API Documentation (Swagger)**: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- **Service Health Check**: [http://127.0.0.1:8000/health](http://127.0.0.1:8000/health)

---

## 📡 API Endpoints Overview

All routes are served under prefix `/api/v1`.

| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/auth/register` | Public | Register new user account; returns access token |
| `POST` | `/auth/login` | Public | Authenticate user; returns JWT & sets session cookie |
| `POST` | `/auth/logout` | Public | Invalidate session |
| `GET` | `/auth/me` | Protected | Fetch authenticated user information |
| `POST` | `/detections/image` | Protected | Upload image (`jpg`, `png`, `webp`) for real-time inference |
| `POST` | `/detections/video` | Protected | Upload video (`mp4`) for background frame analysis |
| `GET` | `/detections` | Protected | List detection history for current user |
| `GET` | `/detections/{id}` | Protected | Retrieve detection results by ID |
| `GET` | `/detections/{id}/frames` | Protected | Inspect frame-level score list |
| `GET` | `/detections/{id}/timeline`| Protected | Inspect suspicious timeline intervals |
| `DELETE`| `/detections/{id}` | Protected | Remove detection record from history |

---

## 🧪 Testing

Run the automated test suite with `pytest`:

```powershell
cd deepfake-backend
python -m pytest -v
```

All 14 unit and integration tests verify:
- Argon2id password hashing and JWT issuance
- User authentication and access restrictions
- Input validation (unsupported media rejection, file size limits)
- DeepGuard model score parsing, binary label handling, and face detection fallback
- Video sampling and asynchronous job execution
- CORS preflight headers for GitHub Pages and development origins

---

## 🤖 Pretrained Model Details

- **Architecture**: Multi-Scale Efficient Global Context Vision Transformer (`ms_eff_gcvit_b0`)
- **Weights Source**: [`KoreaPeter/ms-eff-gcvit-deepfake-b0-ff-plus-plus`](https://huggingface.co/KoreaPeter/ms-eff-gcvit-deepfake-b0-ff-plus-plus)
- **Benchmark**: Trained on **FaceForensics++** (Deepfakes, Face2Face, FaceSwap, NeuralTextures)
- **Face Detector**: YOLOv8-face (`ultralytics`)
- **License**: MIT License

---

## 👥 Authors & Team Contributions

Developed for **Parakh (परख)** under **MnM Developers**:
- **Shivang Saxena** — Backend Architecture, Authentication & AI/ML Integration
- **Tanu Thakur** — Backend, Database & Auth
- **Aditya Verma** — AI/ML & Backend
- **Anurag Verma** — Frontend Development
- **Nishu Kumari** — Frontend Development
- **Yash Gupta** — Frontend Development
- **Charvi Goyal** — Data Science & Database
- **Agrima Tripathi** — Technical Documentation
