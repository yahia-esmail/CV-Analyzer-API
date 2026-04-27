# 📄 CVAnalyzerAPI

A powerful **RESTful API** built with **ASP.NET Core (.NET 10)** that allows users to upload their CVs and get AI-powered analysis using **Groq AI** (powered by LLaMA). The API provides scores, strengths, weaknesses, suggestions, and job match percentages — all secured with JWT authentication.

---

## 🚀 Features

- 🔐 **JWT Authentication** with Refresh Token support (stored in HttpOnly cookies)
- 📧 **Email-based Password Reset** via MailKit/SMTP
- 📂 **CV Upload & Storage** on Cloudinary (PDF files)
- 🤖 **AI-Powered CV Analysis** using Groq API (LLaMA model)
- 📊 **Detailed Analysis Results**: Score, Strengths, Weaknesses, Suggestions, Job Match %
- 🔗 **Shareable Analysis Links** via unique public tokens (no auth required)
- ♻️ **Re-analyze** an already uploaded CV anytime
- 🗄️ **HybridCache** for fast repeated data retrieval (in-memory + distributed)
- 🛡️ **Rate Limiting** on sensitive endpoints (login, forgot password, upload, analyze)
- 📋 **Structured Logging** with Serilog (Console + rolling File sinks)
- ⚡ **Polly Retry Policy** for resilient external HTTP calls
- ✅ **FluentValidation** for robust request input validation
- 🧱 **Global Exception Handler** middleware for consistent error responses
- 🌐 **CORS** configured for Angular frontend (`localhost:4200` & Vercel deployment)

---

## 🏗️ Architecture Overview

```
CVAnalyzerAPI/
├── Controllers/          # API endpoints (AuthsController, CVsController)
├── Services/
│   ├── AuthServices/     # Register, Login, RefreshToken, ForgotPassword, ResetPassword
│   ├── CVServices/       # Upload, GetAll, GetAnalysis, ReAnalyze, Delete, Share
│   ├── AnalyzeServices/  # Groq AI & Gemini AI integration
│   ├── FileServices/     # Cloudinary file upload/delete
│   ├── EmailServices/    # SMTP email sending via MailKit
│   └── TokenServices/    # JWT & Refresh token generation
├── Models/               # EF Core entity models (CV, Analysis, ApplicationUser, RefreshToken)
├── DTOs/                 # Request/Response data transfer objects
├── Validators/           # FluentValidation validators
├── Middlewares/          # Global exception handler
├── Consts/               # App constants, settings, error codes
├── Data/                 # EF Core DbContext
├── Migrations/           # Database migration files
└── Templates/            # Email HTML templates
```

---

## 🔌 API Endpoints

### 🔑 Auth — `/api/Auths`

| Method | Endpoint | Description | Auth Required | Rate Limit |
|--------|----------|-------------|---------------|------------|
| `POST` | `/api/Auths/register` | Register a new user | ❌ | ❌ |
| `POST` | `/api/Auths/login` | Login and get JWT + refresh token | ❌ | ✅ 5 req / 5 min |
| `POST` | `/api/Auths/refresh-token` | Refresh JWT using cookie refresh token | ❌ | ❌ |
| `POST` | `/api/Auths/forgot-password` | Send password reset email | ❌ | ✅ 3 req / 1 hr |
| `POST` | `/api/Auths/reset-password` | Reset password using token | ❌ | ❌ |

---

### 📄 CVs — `/api/CVs`

> All endpoints require **Bearer JWT token** unless noted.

| Method | Endpoint | Description | Rate Limit |
|--------|----------|-------------|------------|
| `POST` | `/api/CVs/upload` | Upload a PDF CV and get AI analysis | ✅ 10 req / 1 hr |
| `GET` | `/api/CVs` | Get all CVs uploaded by the logged-in user | ❌ |
| `GET` | `/api/CVs/{id}/analysis` | Get analysis result for a specific CV | ❌ |
| `POST` | `/api/CVs/{id}/reanalyze` | Re-run AI analysis on an already uploaded CV | ✅ 15 req / 1 hr |
| `DELETE` | `/api/CVs/{id}` | Delete a CV and its associated data | ❌ |
| `GET` | `/api/CVs/share-analysis/{token}` | View shared analysis via public token | ✅ 60 req / 1 min (public) |

---

## 🤖 AI Analysis Output

When a CV is analyzed, the API returns a structured `Analysis` object:

```json
{
  "score": 85,
  "jobMatchPercentage": 78,
  "technicalAlignment": 80,
  "softSkillsFit": 75,
  "domainExperience": 90,
  "strengths": [
    { "icon": "💡", "heading": "Strong Technical Skills", "description": "..." }
  ],
  "weaknesses": ["Lacks project management experience", "..."],
  "suggestions": [
    { "heading": "Add Certifications", "description": "Consider adding AWS certification..." }
  ]
}
```

---

## 🔐 Authentication Flow

```
1. POST /register → returns JWT access token + sets refreshToken cookie
2. POST /login    → returns JWT access token + sets refreshToken cookie
3. Use JWT in Authorization header: Bearer <token>
4. When JWT expires → POST /refresh-token (cookie is sent automatically)
```

- **Access Token**: Short-lived JWT (15 minutes recommended)
- **Refresh Token**: Long-lived token stored in **HttpOnly, Secure, SameSite=None** cookie
- **Password Reset**: Email link with token → POST `/reset-password`

---

## ⚙️ Tech Stack & Dependencies

| Category | Technology |
|----------|-----------|
| **Framework** | ASP.NET Core (.NET 10) |
| **Database** | PostgreSQL via Npgsql + EF Core |
| **Identity** | ASP.NET Core Identity |
| **Authentication** | JWT Bearer Tokens |
| **AI Analysis** | Groq API (LLaMA) + Gemini API |
| **File Storage** | Cloudinary (PDF storage) |
| **PDF Parsing** | PdfPig |
| **Email** | MailKit (SMTP) |
| **Validation** | FluentValidation |
| **Caching** | HybridCache (in-memory + distributed) |
| **Resilience** | Polly (Retry policy for HTTP calls) |
| **Logging** | Serilog (Console + File sinks) |
| **Result Handling** | OneOf (discriminated unions) |
| **API Docs** | ASP.NET Core OpenAPI (Scalar UI) |

---

## 🛠️ Setup & Configuration

### Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- PostgreSQL Server
- Cloudinary account
- Groq API key
- SMTP email service credentials

### 1. Clone the repository

```bash
git clone https://github.com/your-username/CVAnalyzerAPI.git
cd CVAnalyzerAPI
```

### 2. Configure User Secrets (recommended for local dev)

```bash
dotnet user-secrets init
dotnet user-secrets set "JwtSettings:SecretKey" "your-super-secret-key"
dotnet user-secrets set "JwtSettings:Issuer" "CVAnalyzerAPI"
dotnet user-secrets set "JwtSettings:Audience" "CVAnalyzerClient"
dotnet user-secrets set "GeminiSettings:ApiKey" "your-gemini-api-key"
dotnet user-secrets set "GroqSettings:ApiKey" "your-groq-api-key"
dotnet user-secrets set "CloudinarySettings:CloudName" "your-cloud-name"
dotnet user-secrets set "CloudinarySettings:ApiKey" "your-cloudinary-api-key"
dotnet user-secrets set "CloudinarySettings:ApiSecret" "your-cloudinary-secret"
dotnet user-secrets set "EmailSettings:Host" "smtp.gmail.com"
dotnet user-secrets set "EmailSettings:Port" "587"
dotnet user-secrets set "EmailSettings:Username" "your-email@gmail.com"
dotnet user-secrets set "EmailSettings:Password" "your-app-password"
```

### 3. Configure `appsettings.json`

Update the connection string:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=CVAnalyzerDb;Username=postgres;Password=yourpassword"
  }
}
```

### 4. Apply Database Migrations

```bash
dotnet ef database update
```

### 5. Run the API

```bash
dotnet run
```

API will be available at: `https://localhost:7xxx`  
OpenAPI docs (dev only): `https://localhost:7xxx/openapi/v1.json`

---

## 🛡️ Rate Limiting Summary

| Policy | Endpoint | Limit |
|--------|----------|-------|
| `login` | `POST /login` | 5 requests per 5 minutes (per IP) |
| `ForgotPassword` | `POST /forgot-password` | 3 requests per 1 hour (per IP) |
| `UploadCV` | `POST /upload` | 10 requests per 1 hour (per IP+User) |
| `Analyze` | `POST /{id}/reanalyze` | 15 requests per 1 hour (per IP+User) |
| `public-link` | `GET /share-analysis/{token}` | 60 requests per 1 minute (per IP) |

When a rate limit is exceeded, the API returns **HTTP 400** with:
```json
{ "error": "Too many requests. Please try again later." }
```

---

## 📝 Logging

Serilog is configured to log to:
- **Console** — formatted with timestamp and level
- **File** — rolling daily logs at `../private/logs/log-<date>.txt`

Log levels:
- `Information` — Default for application events
- `Warning` — Suppressed for `Microsoft.*` and `System.*` namespaces

---

## 🌐 CORS Policy

Allowed origins:
- `http://localhost:4200` (local Angular dev)
Allows all methods, all headers, and credentials (cookies).

---

## 📁 Data Models

### `CV`
| Field | Type | Description |
|-------|------|-------------|
| `Id` | int | Primary key |
| `UserId` | string | Owner user ID |
| `FileName` | string | Original file name |
| `FilePath` | string | Cloudinary URL |
| `ExtractedText` | string | Parsed text from PDF |
| `UploadedAt` | DateTime | Upload timestamp |
| `ShareToken` | Guid | Unique public sharing token |

### `Analysis`
| Field | Type | Description |
|-------|------|-------------|
| `Id` | int | Primary key |
| `CVId` | int | Related CV ID |
| `Score` | int | Overall CV score (0–100) |
| `JobMatchPercentage` | int? | Match % against job description |
| `TechnicalAlignment` | int | Technical skills match score |
| `SoftSkillsFit` | int | Soft skills score |
| `DomainExperience` | int | Domain experience score |
| `Strengths` | List | Icon + Heading + Description |
| `Weaknesses` | List\<string\> | List of weak points |
| `Suggestions` | List | Heading + Description |
| `CreatedAt` | DateTime | Analysis timestamp |

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add some feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Submit a pull request

---

## 📜 License

This project is licensed under the **MIT License**.
