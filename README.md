<p align="center">
  <h1 align="center">🔍 SCANNER API</h1>
  <p align="center">
    <b>Production-Grade Document Scanning System for Emirates ID & Passport Processing</b>
  </p>
  <p align="center">
    <a href="#-features"><img src="https://img.shields.io/badge/Detection-YOLOv8-FF6F00?style=for-the-badge&logo=yolo" alt="YOLOv8"></a>
    <a href="#-features"><img src="https://img.shields.io/badge/OCR-EasyOCR-2196F3?style=for-the-badge" alt="EasyOCR"></a>
    <a href="#-features"><img src="https://img.shields.io/badge/MRZ-PassportEye-4CAF50?style=for-the-badge" alt="PassportEye"></a>
    <a href="#-tech-stack"><img src="https://img.shields.io/badge/API-FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI"></a>
  </p>
  <p align="center">
    <img src="https://img.shields.io/badge/python-3.9+-3776AB?logo=python&logoColor=white" alt="Python 3.9+">
    <img src="https://img.shields.io/badge/version-1.0.0-blue" alt="Version 1.0.0">
    <img src="https://img.shields.io/badge/Python-66.1%25-3776AB" alt="Python">
    <img src="https://img.shields.io/badge/JavaScript-12.9%25-F7DF1E" alt="JavaScript">
    <img src="https://img.shields.io/badge/CSS-12.6%25-1572B6" alt="CSS">
    <img src="https://img.shields.io/badge/HTML-8.4%25-E34F26" alt="HTML">
  </p>
</p>

---

An intelligent document scanning system that extracts structured data from **Emirates ID cards** and **Passports** via a REST API. The system uses two separate processing pipelines — **YOLO object detection + EasyOCR** for Emirates ID field extraction, and **PassportEye MRZ parsing** (ICAO 9303 compliant) for passport processing. Both pipelines include text normalization, field validation, and per-field confidence scoring. A clean **HTML/JS frontend** provides drag-and-drop upload with real-time result display.

> **Use Cases:** KYC/AML compliance · Identity verification · Government document processing · Border control automation · Banking onboarding · HR employee verification

---

## 📑 Table of Contents

- [✨ Features](#-features)
- [🏗️ Architecture](#️-architecture)
- [📁 Project Structure](#-project-structure)
- [🛠️ Tech Stack](#️-tech-stack)
- [⚡ Quick Start](#-quick-start)
  - [Backend Setup](#backend-setup)
  - [Frontend Setup](#frontend-setup)
- [🔌 API Reference](#-api-reference)
  - [Health Check](#get-apiv1health)
  - [Scan Emirates ID](#post-apiv1idscan)
  - [Scan Passport](#post-apiv1passportscan)
- [🖥️ Frontend](#️-frontend)
- [⚙️ Configuration](#️-configuration)
- [🏛️ Design Decisions](#️-design-decisions)
- [🔧 Troubleshooting](#-troubleshooting)
- [🚀 Production Deployment](#-production-deployment)
- [🗺️ Roadmap](#️-roadmap)
- [🤝 Contributing](#-contributing)

---

## ✨ Features

### Emirates ID Processing
| Feature | Details |
|---------|---------|
| 🎯 **YOLO Field Detection** | Custom-trained model detects 7 field regions on Emirates ID cards |
| 🔤 **EasyOCR Extraction** | Multi-language text extraction from detected regions |
| 🧹 **Text Normalization** | Handles common OCR errors (O→0, I→1, etc.) |
| ✅ **ID Number Validation** | Emirates ID format validation (`784-XXXX-XXXXXXX-X`) |
| 📅 **Date Consistency** | Cross-validates dates across fields |
| 📊 **Per-Field Confidence** | Combined detection + OCR confidence using geometric mean |

### Passport Processing
| Feature | Details |
|---------|---------|
| 📖 **MRZ Extraction** | Machine Readable Zone parsing from passport images |
| 🌐 **ICAO 9303 Compliance** | International standard for machine-readable travel documents |
| ✅ **Checksum Validation** | MRZ check digit verification |
| 📅 **Date Parsing** | Automatic date format standardization |
| 🏳️ **Nationality Normalization** | Country code to full name conversion |

### API & System
| Feature | Details |
|---------|---------|
| 📡 **Versioned REST API** | Clean `/api/v1/` endpoint structure |
| 📄 **Auto Documentation** | Swagger UI at `/docs` + ReDoc at `/redoc` |
| 🔗 **CORS Enabled** | Frontend-ready cross-origin support |
| ⚠️ **Warning System** | Non-blocking warnings for validation issues |
| ⏱️ **Processing Metrics** | Per-request processing time tracking |
| 🏗️ **Stateless Architecture** | No database — each request is independent |
| 🧩 **Plugin Architecture** | Abstract `BasePipeline` — easily add new document types |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    Frontend (HTML + CSS + JS)                      │
│                                                                   │
│   index.html          upload.html            result.html          │
│   Document Type       Drag & Drop            Extracted Data       │
│   Selection           File Upload            Display              │
└──────────────────────────┬───────────────────────────────────────┘
                           │  HTTP/REST
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                    FastAPI Backend (v1.0.0)                        │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                    API Layer (/api/v1/)                       │ │
│  │                                                              │ │
│  │  GET  /health          → System health check                 │ │
│  │  POST /id/scan         → Emirates ID processing              │ │
│  │  POST /passport/scan   → Passport processing                 │ │
│  └──────────┬─────────────────────────────┬─────────────────────┘ │
│             │                             │                       │
│  ┌──────────▼──────────┐       ┌──────────▼──────────┐           │
│  │  Emirates ID        │       │  Passport            │           │
│  │  Pipeline           │       │  Pipeline             │           │
│  │                     │       │                       │           │
│  │  YOLO Detection     │       │  PassportEye          │           │
│  │  → Field Cropping   │       │  (MRZ Parser)         │           │
│  │  → EasyOCR          │       │  → ICAO 9303          │           │
│  │  → Normalization    │       │  → Checksum           │           │
│  │  → Validation       │       │  → Validation         │           │
│  └──────────┬──────────┘       └──────────┬───────────┘           │
│             │                             │                       │
│  ┌──────────▼─────────────────────────────▼──────────────────┐   │
│  │                  Shared Utilities                          │   │
│  │                                                            │   │
│  │  normalization.py  │  validation.py  │  image.py           │   │
│  │  Text cleaning     │  Field checks   │  Image preprocessing│   │
│  │  OCR error fixes   │  Format rules   │  File handling      │   │
│  └────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
SCANNER-API/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                        # FastAPI app factory, CORS, router mount
│   │   │
│   │   ├── api/
│   │   │   └── v1/                        # Versioned API endpoints
│   │   │       ├── __init__.py
│   │   │       ├── router.py              # API router aggregation
│   │   │       └── endpoints/             # Individual endpoint handlers
│   │   │
│   │   ├── core/                          # Configuration & exceptions
│   │   │   ├── config.py                  # Settings: thresholds, paths, limits
│   │   │   └── exceptions.py              # Custom exception hierarchy
│   │   │
│   │   ├── models/                        # Data schemas
│   │   │   ├── __init__.py
│   │   │   └── responses.py              # Pydantic response models
│   │   │
│   │   ├── pipelines/                     # Document processing engines
│   │   │   ├── __init__.py
│   │   │   ├── base.py                    # Abstract BasePipeline class
│   │   │   ├── emirates_id.py             # Emirates ID: YOLO + EasyOCR pipeline
│   │   │   └── passport.py               # Passport: PassportEye MRZ pipeline
│   │   │
│   │   └── utils/                         # Shared utilities
│   │       ├── __init__.py
│   │       ├── image.py                   # Image preprocessing & file handling
│   │       ├── normalization.py           # Text cleaning & OCR error correction
│   │       └── validation.py             # Field format validation rules
│   │
│   ├── models/                            # ML model weights
│   │   └── best.pt                        # YOLOv8 trained model (not in repo)
│   │
│   ├── requirements.txt                   # Python dependencies
│   └── run.py                             # Uvicorn server launcher
│
├── frontend/
│   ├── index.html                         # Document type selection page
│   ├── upload.html                        # Drag-and-drop file upload page
│   ├── result.html                        # Extracted data results page
│   ├── css/
│   │   └── style.css                      # Gradient-styled responsive CSS
│   └── js/
│       └── app.js                         # Frontend logic (upload, API calls, display)
│
├── .gitignore
└── README.md
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Emirates ID Detection** | [YOLOv8](https://github.com/ultralytics/ultralytics) (Ultralytics) | Detect 7 field regions on ID cards |
| **Text Extraction** | [EasyOCR](https://github.com/JaidedAI/EasyOCR) | Multi-language OCR from cropped regions |
| **Passport Parsing** | [PassportEye](https://github.com/konstantint/PassportEye) | MRZ extraction & ICAO 9303 parsing |
| **Backend Framework** | [FastAPI](https://fastapi.tiangolo.com/) 0.109 | Async REST API with auto-generated docs |
| **Server** | [Uvicorn](https://www.uvicorn.org/) 0.27 | High-performance ASGI server |
| **Validation** | [Pydantic v2](https://docs.pydantic.dev/) + Pydantic-Settings | Request/response schemas & config |
| **Image Processing** | [Pillow](https://pillow.readthedocs.io/) + [NumPy](https://numpy.org/) | Image preprocessing & manipulation |
| **Frontend** | HTML + CSS + Vanilla JavaScript | Document selection, upload, result display |
| **Language** | Python 3.9+ | 66.1% of codebase |

---

## ⚡ Quick Start

### Prerequisites

- **Python** 3.9+
- **YOLO model file** (`best.pt`) — trained for Emirates ID field detection
- **GPU** (optional, recommended for faster processing)

### Backend Setup

```bash
# 1. Clone the repository
git clone https://github.com/Fairoos-palakkal/SCANNER-API.git
cd SCANNER-API

# 2. Navigate to backend
cd backend

# 3. Create virtual environment
python -m venv venv
source venv/bin/activate    # On Windows: venv\Scripts\activate

# 4. Install dependencies
pip install -r requirements.txt

# 5. Place your YOLO model
mkdir -p models
cp /path/to/your/best.pt models/best.pt

# 6. Start the server
python run.py
```

The API will be available at **http://localhost:8000**

- **Swagger UI:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc

### Frontend Setup

```bash
# Option 1: Simple HTTP server
cd frontend
python -m http.server 3000

# Option 2: Just open in browser
open frontend/index.html
```

Access the UI at **http://localhost:3000**

---

## 🔌 API Reference

### `GET /api/v1/health`

Health check endpoint.

**Response:**
```json
{
  "status": "healthy",
  "timestamp": "2026-01-10T12:00:00",
  "version": "1.0.0"
}
```

---

### `POST /api/v1/id/scan`

Upload an Emirates ID image to extract structured data.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | `UploadFile` | ✅ | Emirates ID image (JPG, PNG) |

#### cURL Example
```bash
curl -X POST "http://localhost:8000/api/v1/id/scan" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/emirates_id.jpg"
```

#### Python Example
```python
import requests

url = "http://localhost:8000/api/v1/id/scan"
files = {"file": open("emirates_id.jpg", "rb")}
response = requests.post(url, files=files)
print(response.json())
```

#### Response
```json
{
  "document_type": "emirates_id",
  "fields": {
    "id_number": {
      "value": "784-2020-1234567-1",
      "confidence": 0.95,
      "bbox": [100, 200, 300, 250]
    },
    "full_name": {
      "value": "Ahmed Ali Mohammed",
      "confidence": 0.92,
      "bbox": [100, 260, 400, 310]
    },
    "nationality": {
      "value": "United Arab Emirates",
      "confidence": 0.88
    },
    "date_of_birth": {
      "value": "15/03/1990",
      "confidence": 0.91
    },
    "expiry_date": {
      "value": "20/06/2028",
      "confidence": 0.89
    }
  },
  "processing_time_ms": 1234.56,
  "warnings": [],
  "metadata": {
    "model": "YOLO + EasyOCR"
  }
}
```

---

### `POST /api/v1/passport/scan`

Upload a passport image to extract data from the MRZ zone.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | `UploadFile` | ✅ | Passport image (JPG, PNG) |

#### cURL Example
```bash
curl -X POST "http://localhost:8000/api/v1/passport/scan" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/passport.jpg"
```

#### Response
```json
{
  "document_type": "passport",
  "fields": {
    "full_name": {
      "value": "Ahmed Ali Mohammed",
      "confidence": 0.95
    },
    "passport_number": {
      "value": "A12345678",
      "confidence": 0.95
    },
    "nationality": {
      "value": "United Arab Emirates",
      "confidence": 0.95
    },
    "date_of_birth": {
      "value": "15/03/1990",
      "confidence": 0.95
    },
    "expiry_date": {
      "value": "20/06/2028",
      "confidence": 0.95
    },
    "sex": {
      "value": "M",
      "confidence": 0.95
    }
  },
  "processing_time_ms": 890.12,
  "warnings": [],
  "metadata": {
    "model": "passporteye (MRZ)",
    "standard": "ICAO 9303"
  }
}
```

---

## 🖥️ Frontend

The frontend is a multi-page vanilla HTML/CSS/JS application with three screens:

| Page | File | Description |
|------|------|-------------|
| 📋 **Document Selection** | `index.html` | Choose between Emirates ID or Passport scanning |
| ☁️ **Upload** | `upload.html` | Drag-and-drop or click-to-browse file upload |
| 📊 **Results** | `result.html` | Display extracted fields with image preview |

**Frontend Features:**
- 🎨 Gradient-styled modern UI with responsive design
- 📁 Drag-and-drop file upload with visual feedback
- 🖼️ Image preview alongside extracted data
- ⚡ Real-time API integration with loading states

---

## ⚙️ Configuration

Edit `backend/app/core/config.py` to customize:

| Setting | Default | Description |
|---------|---------|-------------|
| `MAX_FILE_SIZE_MB` | `10` | Maximum upload file size |
| `MIN_DETECTION_CONFIDENCE` | — | Minimum YOLO detection confidence |
| `MIN_OCR_CONFIDENCE` | — | Minimum EasyOCR text confidence |
| `YOLO_MODEL_PATH` | `models/best.pt` | Path to YOLOv8 weights |
| `EASYOCR_LANGUAGES` | `["en"]` | OCR language support |
| `EASYOCR_GPU` | `True` | Enable/disable GPU for EasyOCR |
| `DATE_INPUT_FORMATS` | — | Accepted date input formats |
| `DATE_OUTPUT_FORMAT` | — | Standardized output date format |
| `ALLOWED_ORIGINS` | `["*"]` | CORS allowed origins |

---

## 🏛️ Design Decisions

| Decision | Rationale |
|----------|-----------|
| **Pipeline Separation** | Emirates ID and Passport pipelines inherit from `BasePipeline` as separate classes — clean separation, no logic leakage, easy to add new document types |
| **Stateless Architecture** | No database or persistence — each request is independent, models loaded once at startup (singleton pattern) |
| **Custom Exception Hierarchy** | `ScannerBaseException` base class with structured error responses — no stack traces exposed to clients |
| **Geometric Mean Confidence** | Detection + OCR confidence combined using geometric mean to penalize weak links in the pipeline |
| **Per-Field Normalization** | Separate normalization functions per field type — handles OCR character confusion, date standardization, nationality mapping |
| **Versioned API** | `/api/v1/` prefix ensures backward compatibility as the API evolves |

---

## 🔧 Troubleshooting

| Issue | Solution |
|-------|----------|
| **Model not found** | Ensure `best.pt` is placed in `backend/models/best.pt` |
| **CUDA/GPU errors** | Set `EASYOCR_GPU: bool = False` in `config.py` to disable GPU |
| **CORS errors** | Verify API is running on `localhost:8000` and check `ALLOWED_ORIGINS` in config |
| **Low confidence results** | Ensure image is clear, well-lit, properly oriented, and all text is in focus |
| **Import errors** | Verify all dependencies are installed: `pip install -r requirements.txt` |

---

## 🚀 Production Deployment

```bash
# Use Gunicorn with Uvicorn workers
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker
```

**Production Checklist:**

| Category | Action |
|----------|--------|
| 🔒 **Security** | Restrict CORS — update `ALLOWED_ORIGINS` to specific domains |
| 🔑 **Auth** | Add API key/JWT authentication via FastAPI dependencies |
| 🚦 **Rate Limiting** | Use `slowapi` library to prevent abuse |
| 📊 **Logging** | Integrate with CloudWatch, Datadog, or similar; log all requests and errors |
| ⚡ **Performance** | Track processing times, set alerts for slow requests |
| 📈 **Scaling** | Run multiple instances behind a load balancer with shared model storage |

---

## 🗺️ Roadmap

- [x] Emirates ID field detection with YOLO
- [x] EasyOCR text extraction with normalization
- [x] Passport MRZ parsing with ICAO 9303 compliance
- [x] FastAPI REST API with versioned endpoints
- [x] Pydantic response schemas with confidence scoring
- [x] Frontend with document selection, upload, and results
- [x] Custom exception hierarchy and structured errors
- [x] Per-field confidence tracking (geometric mean)
- [ ] Arabic text extraction support
- [ ] Batch processing for multiple documents
- [ ] Database integration for processed records
- [ ] Docker containerization
- [ ] API key authentication
- [ ] Rate limiting with `slowapi`
- [ ] Additional document types (Driving License, Visa, etc.)
- [ ] WebSocket support for real-time processing status

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. **Fork** the repository
2. **Create** your feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** your changes (`git commit -m 'Add amazing feature'`)
4. **Push** to the branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request

### Adding a New Document Type

To add a new document type (e.g., Driving License):

1. Create `backend/app/pipelines/driving_license.py`
2. Inherit from `BasePipeline`
3. Implement the `process()` method
4. Add a new endpoint in `backend/app/api/v1/`
5. Register the route in `router.py`

---

## 📄 License

Internal use only. For issues or questions, contact the engineering team.

---

<p align="center">
  <b>Built with ❤️ by <a href="https://github.com/Fairoos-palakkal">Fairoos Palakkal</a></b>
  <br>
  <sub>If you found this project useful, please consider giving it a ⭐</sub>
</p>


