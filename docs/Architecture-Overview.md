# Mimariye Genel Bakış 
<details open>
<summary><strong>🇹🇷 Türkçe</strong></summary>
<br>

Bu sayfa, projenin yüksek seviyeli mimarisini, ana bileşenlerini ve bu bileşenlerin birbiriyle olan ilişkilerini açıklamaktadır.

---

*Bu alan projenin teknik mimarisi geliştikçe doldurulmalıdır.*

---

## Ana Bileşenler

- **Frontend:** Angular 20, TailwindCSS, RxJS, Signals
  - **Sorumluluklar:**
    - Kullanıcı arayüzü ve etkileşim
    - Profil kartı görselleştirme
    - Dashboard ve filtreleme arayüzü
    - Responsive design (web)
  - **Port:** `http://localhost:4200`

- **Backend API:** FastAPI, SQLAlchemy, Pydantic
  - **Sorumluluklar:**
    - RESTful API endpoints
    - Business logic ve veri validasyonu
    - GitHub/LinkedIn/Instagram API entegrasyonları
    - Database işlemleri (CRUD)
    - Authentication & Authorization (JWT)
  - **Port:** `http://localhost:8000`
  - **API Docs:** `http://localhost:8000/docs` (Swagger UI)
  
- **AI/NLP Servisi (Python AI Engine):** OpenAI API, spaCy, Transformers, Langchain
  - **Sorumluluklar:**
    - GitHub repo analizi ve beceri çıkarımı
    - LinkedIn profil parsing ve NLP
    - OpenAI GPT ile profil özeti oluşturma
    - Beceri skorlama ve kategorilendirme
  - **Entegrasyon:** Backend API'den internal fonksiyon çağrısı

- **Database:** PostgreSQL veritabanı
  - **Sorumluluklar:**
    - Kullanıcı profilleri
    - Sosyal medya verileri cache
    - Profil kartları metadata
    - Authentication tokens
  - **Port:** `localhost:5432`

- **Authentication Service:** FastAPI Security, PyJWT, OAuth2
  - **Sorumluluklar:**
    - Kullanıcı kayıt ve giriş
    - JWT token yönetimi
    - GitHub/LinkedIn OAuth entegrasyonu
    - Role-based access control (RBAC)

## Sistem Mimarisi Şeması

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Angular Web AppPort: 4200]
    end

    subgraph "API Gateway"
        API[FastAPI BackendPort: 8000]
    end

    subgraph "Service Layer"
        AUTH[Authentication ServiceJWT + OAuth2]
        AI[AI/NLP EngineOpenAI + spaCy]
        GITHUB[GitHub API Client]
        LINKEDIN[LinkedIn API Client]
        INSTA[Instagram API Client]
    end

    subgraph "Data Layer"
        DB[(PostgreSQLPort: 5432)]
        CACHE[(Redis CacheOptional)]
    end

    WEB -->|REST API| API
    
    API --> AUTH
    API --> AI
    API --> GITHUB
    API --> LINKEDIN
    API --> INSTA
    API --> DB
    API --> CACHE

    AI -->|Extract Skills| GITHUB
    AI -->|Parse Profile| LINKEDIN
    
    AUTH --> DB
```

## Veri Akışı

### Senaryo 1: Kullanıcı Profil Oluşturma
```mermaid
sequenceDiagram
    participant U as User (Web)
    participant API as FastAPI Backend
    participant AUTH as Auth Service
    participant GH as GitHub API
    participant AI as AI/NLP Engine
    participant DB as PostgreSQL

    U->>API: POST /api/auth/register
    API->>AUTH: Create User
    AUTH->>DB: Save User
    DB-->>AUTH: User Created
    AUTH-->>API: JWT Token
    API-->>U: 201 Created + Token

    U->>API: POST /api/profile/analyze{github_url, linkedin_url}
    API->>AUTH: Verify JWT
    AUTH-->>API: Valid User

    API->>GH: Fetch Repos & Commits
    GH-->>API: Repository Data

    API->>AI: Extract Skills & Experience
    AI->>AI: NLP ProcessingOpenAI API
    AI-->>API: Extracted Skills

    API->>DB: Save Profile Card
    DB-->>API: Profile Saved

    API-->>U: 200 OK + Profile Card
```

### Senaryo 2: Profil Kartı Görüntüleme
```
User → Frontend → API → Database → API → Frontend → User
  1. Kullanıcı profil kartına tıklar
  2. Frontend API'ye GET /api/profiles/{id} isteği gönderir
  3. API database'den profil kartını çeker
  4. Frontend kartı görselleştirir
```

### Senaryo 3: Akıllı Arama/Filtreleme
```
User → Frontend → API → Database → AI (Optional) → API → Frontend → User
  1. Kullanıcı "Python developer" arar
  2. API database'de full-text search yapar
  3. AI servisi sonuçları relevance'a göre sıralar
  4. Frontend sonuçları listeler
```
---

## Teknolojiler ve Gerekçeleri

### Backend: Python + FastAPI
**Neden seçildi:**
- OpenAI resmi SDK native Python desteği
- NLP kütüphaneleri (spaCy, NLTK, Transformers) Python'da güçlü
- Async/await desteği (performans)
- Otomatik API dokümantasyonu (Swagger/OpenAPI)
- Type hints ile type safety (Pydantic)
- Hızlı prototipleme ve geliştirme
- AI/ML ekosistemi zengin

**Dezavantajları:**
- Java kadar strict type system yok
- Thread-based concurrency sınırlı (GIL)

**Alternatifler:** Java Spring Boot, Node.js Express
**Karar:** AI/NLP odaklı proje olduğu için Python optimal seçim

---

### Frontend: Angular 20
**Neden seçildi:**
- Enterprise-grade framework
- Strong typing (TypeScript)
- Modern reactive programming (Signals)
- Built-in form validation, routing, HTTP client
- TailwindCSS ile hızlı UI geliştirme
- Component-based architecture

  **Dezavantajları:**
- Öğrenim eğrisi React'tan daha dik
- Bundle size biraz daha büyük

**Alternatifler:** React, Vue.js, Svelte
**Karar:** Type safety, enterprise patterns ve Angular 20'nin modern özellikleri

---

### Database: PostgreSQL
**Neden seçildi:**
- Open source ve güvenilir
- JSONB desteği (flexible schema)
- Full-text search (profil araması için kritik)
- Complex queries ve indexing
- ACID compliance
- SQLAlchemy ORM ile mükemmel entegrasyon

**Dezavantajları:**
- NoSQL kadar flexible değil

**Alternatifler:** MongoDB, MySQL, SQLite
**Karar:** Structured data + full-text search ihtiyacı için ideal

---

### AI/ML: OpenAI API + spaCy
**Neden seçildi:**
- OpenAI GPT-4: En gelişmiş NLP modeli
- spaCy: Hızlı ve production-ready NLP
- Langchain: LLM orchestration
- Transformers: Hugging Face modelleri

**Dezavantajları:**
- OpenAI API maliyeti (rate limiting gerekli)
- External dependency

**Alternatifler:** Self-hosted LLM, Claude API, Gemini
**Karar:** OpenAI industry standard ve en iyi sonuçlar

---

### Authentication: JWT + OAuth 2.0
**Neden seçildi:**
- Stateless authentication (scalable)
- OAuth 2.0 industry standard
- GitHub/LinkedIn login entegrasyonu kolay
- JWT token'lar lightweight

**Dezavantajları:**
- Token revocation kompleks olabilir

**Alternatifler:** Session-based auth, Auth0, Keycloak
**Karar:** Basit, standart, cost-effective

---

---

## 🔒 Güvenlik Mimarisi

### Authentication Flow
```
1. User → Login with GitHub/LinkedIn
2. OAuth Provider → Authorization Code
3. Backend → Exchange code for access token
4. Backend → Create JWT token
5. Frontend → Store JWT in httpOnly cookie
6. Subsequent requests → JWT in Authorization header
```

### Data Security
- Password hashing: bcrypt
- HTTPS only (TLS/SSL)
- Rate limiting: 100 requests/hour per user
- Input validation: Pydantic models
- SQL injection prevention: SQLAlchemy ORM
- CORS policy: Whitelisted domains
- API key rotation: Monthly

---

## 📊 Performans ve Scalability

### Caching Strategy
```python
# Redis cache for GitHub API responses
@cache(ttl=3600)  # 1 hour
async def fetch_github_repos(username: str):
    # Expensive API call
    pass
```

### Database Indexing
```sql
-- Full-text search index
CREATE INDEX idx_profile_skills ON profiles 
USING GIN (to_tsvector('english', skills));

-- User lookup index
CREATE INDEX idx_users_email ON users (email);
```

### API Rate Limiting
```python
from slowapi import Limiter

limiter = Limiter(key_func=get_remote_address)

@app.get("/api/profiles")
@limiter.limit("10/minute")
async def list_profiles():
    pass
```

---

## 🚀 Deployment Architecture
```mermaid
graph LR
    subgraph "Production Environment"
        LB[Load BalancerNginx]
        API1[FastAPI Instance 1]
        API2[FastAPI Instance 2]
        DB[(PostgreSQLPrimary)]
        DBR[(PostgreSQLReplica)]
        REDIS[(Redis Cache)]
    end

    LB --> API1
    LB --> API2
    API1 --> DB
    API2 --> DB
    API1 --> REDIS
    API2 --> REDIS
    DB --> DBR
```

### Deployment Platforms
- **Backend:** Railway, Render, Fly.io
- **Frontend:** Vercel, Netlify
- **Database:** Railway PostgreSQL, Supabase
- **Mobile:** App Store, Google Play Store

---

## 📝 API Endpoint Yapısı

### Authentication
```
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/refresh
GET    /api/auth/me
```

### Profiles
```
POST   /api/profiles/analyze    # Create profile from social media
GET    /api/profiles            # List all profiles (with filters)
GET    /api/profiles/{id}       # Get single profile
PUT    /api/profiles/{id}       # Update profile
DELETE /api/profiles/{id}       # Delete profile
```

### Skills
```
GET    /api/skills              # List all skills
GET    /api/skills/trending     # Trending skills
```

### Search
```
GET    /api/search?q=python&skills=django,fastapi
```

Detaylı API dokümantasyonu için: [API_SPECIFICATION.md](./API_SPECIFICATION.md)

---

## 🔄 CI/CD Pipeline
```mermaid
graph LR
    CODE[Code Push] --> GH[GitHub Actions]
    GH --> TEST[Run Tests]
    TEST --> BUILD[Build Docker Image]
    BUILD --> DEPLOY[Deploy to Railway]
    DEPLOY --> VERIFY[Health Check]
```

---

## 📚 Kaynaklar

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Angular Documentation](https://angular.io/docs)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [OpenAI API Reference](https://platform.openai.com/docs)

---

## 🔮 Gelecek İyileştirmeler

- [ ] GraphQL API desteği
- [ ] Real-time notifications (WebSockets)
- [ ] Machine Learning model training pipeline
- [ ] Multi-language support (i18n)
- [ ] Advanced analytics dashboard
- [ ] Kubernetes deployment

---

</details>

<details>
<summary><strong>🇬🇧 English</strong></summary>

<br>

This page describes the high-level architecture of the Project project, its main components, and the relationships between these components.

---

*This section should be filled out as the project's technical architecture develops.*

---

## Main Components

- **Frontend:** Angular 20, TailwindCSS, RxJS, Signals  
  - **Responsibilities:**
    - User interface and interaction
    - Profile card visualization
    - Dashboard and filtering interface
    - Responsive design (web)
  - **Port:** `http://localhost:4200`

- **Backend API:** FastAPI, SQLAlchemy, Pydantic  
  - **Responsibilities:**
    - RESTful API endpoints
    - Business logic and data validation
    - GitHub/LinkedIn/Instagram API integrations
    - Database operations (CRUD)
    - Authentication & Authorization (JWT)
  - **Port:** `http://localhost:8000`
  - **API Docs:** `http://localhost:8000/docs` (Swagger UI)

- **AI/NLP Service (Python AI Engine):** OpenAI API, spaCy, Transformers, Langchain  
  - **Responsibilities:**
    - GitHub repo analysis and skill extraction
    - LinkedIn profile parsing and NLP
    - Creating profile summaries using OpenAI GPT
    - Skill scoring and categorization
  - **Integration:** Internal function call from Backend API

- **Database:** PostgreSQL  
  - **Responsibilities:**
    - User profiles
    - Cached social media data
    - Profile card metadata
    - Authentication tokens
  - **Port:** `localhost:5432`

- **Authentication Service:** FastAPI Security, PyJWT, OAuth2  
  - **Responsibilities:**
    - User registration and login
    - JWT token management
    - GitHub/LinkedIn OAuth integration
    - Role-based access control (RBAC)

## System Architecture Diagram

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Angular Web AppPort: 4200]
    end

    subgraph "API Gateway"
        API[FastAPI BackendPort: 8000]
    end

    subgraph "Service Layer"
        AUTH[Authentication ServiceJWT + OAuth2]
        AI[AI/NLP EngineOpenAI + spaCy]
        GITHUB[GitHub API Client]
        LINKEDIN[LinkedIn API Client]
        INSTA[Instagram API Client]
    end

    subgraph "Data Layer"
        DB[(PostgreSQLPort: 5432)]
        CACHE[(Redis CacheOptional)]
    end

    WEB -->|REST API| API
    
    API --> AUTH
    API --> AI
    API --> GITHUB
    API --> LINKEDIN
    API --> INSTA
    API --> DB
    API --> CACHE

    AI -->|Extract Skills| GITHUB
    AI -->|Parse Profile| LINKEDIN
    
    AUTH --> DB



```

## Data Flow

### Scenario 1: User Profile Creation
```mermaid
sequenceDiagram
    participant U as User (Web)
    participant API as FastAPI Backend
    participant AUTH as Auth Service
    participant GH as GitHub API
    participant AI as AI/NLP Engine
    participant DB as PostgreSQL

    U->>API: POST /api/auth/register
    API->>AUTH: Create User
    AUTH->>DB: Save User
    DB-->>AUTH: User Created
    AUTH-->>API: JWT Token
    API-->>U: 201 Created + Token

    U->>API: POST /api/profile/analyze{github_url, linkedin_url}
    API->>AUTH: Verify JWT
    AUTH-->>API: Valid User

    API->>GH: Fetch Repos & Commits
    GH-->>API: Repository Data

    API->>AI: Extract Skills & Experience
    AI->>AI: NLP ProcessingOpenAI API
    AI-->>API: Extracted Skills

    API->>DB: Save Profile Card
    DB-->>API: Profile Saved

    API-->>U: 200 OK + Profile Card
```

### Scenario 2: Viewing a Profile Card
```
User → Frontend → API → Database → API → Frontend → User
  1. User clicks on the profile card
  2. Frontend sends GET /api/profiles/{id} request to the API
  3. API fetches the profile card from the database
  4. Frontend visualizes the card
```

### Scenario 3: Smart Search/Filtering
```
User → Frontend → API → Database → AI (Optional) → API → Frontend → User
  1. User searches for “Python developer”
  2. API performs full-text search in the database
  3. AI service ranks results by relevance
  4. Frontend lists the results
```
---

## Teknolojiler ve Gerekçeleri

### Backend: Python + FastAPI
**Why it was chosen:**
- OpenAI resmi SDK native Python desteği
- NLP kütüphaneleri (spaCy, NLTK, Transformers) Python'da güçlü
- Async/await desteği (performans)
- Otomatik API dokümantasyonu (Swagger/OpenAPI)
- Type hints ile type safety (Pydantic)
- Hızlı prototipleme ve geliştirme
- AI/ML ekosistemi zengin

**Disadvantages:**
- Java kadar strict type system yok
- Thread-based concurrency sınırlı (GIL)

**Alternatives:** Java Spring Boot, Node.js Express
**Decision:** AI/NLP odaklı proje olduğu için Python optimal seçim

---

### Frontend: Angular 20
**Why it was chosen:**
- Enterprise-grade framework
- Strong typing (TypeScript)
- Modern reactive programming (Signals)
- Built-in form validation, routing, HTTP client
- TailwindCSS ile hızlı UI geliştirme
- Component-based architecture

  **Disadvantages:**
- Öğrenim eğrisi React'tan daha dik
- Bundle size biraz daha büyük

**Alternatives:** React, Vue.js, Svelte
**Decision:** Type safety, enterprise patterns ve Angular 20'nin modern özellikleri

---

### Database: PostgreSQL
**Why it was chosen:**
- Open source ve güvenilir
- JSONB desteği (flexible schema)
- Full-text search (profil araması için kritik)
- Complex queries ve indexing
- ACID compliance
- SQLAlchemy ORM ile mükemmel entegrasyon

**Disadvantages:**
- NoSQL kadar flexible değil

**Alternatives:** MongoDB, MySQL, SQLite
**Decision:** Structured data + full-text search ihtiyacı için ideal

---

### AI/ML: OpenAI API + spaCy
**Why it was chosen:**
- OpenAI GPT-4: En gelişmiş NLP modeli
- spaCy: Hızlı ve production-ready NLP
- Langchain: LLM orchestration
- Transformers: Hugging Face modelleri

**Disadvantages:**
- OpenAI API maliyeti (rate limiting gerekli)
- External dependency

**Alternatives:** Self-hosted LLM, Claude API, Gemini
**Decision:** OpenAI industry standard ve en iyi sonuçlar

---

### Authentication: JWT + OAuth 2.0
**Why it was chosen:**
- Stateless authentication (scalable)
- OAuth 2.0 industry standard
- Easy GitHub/LinkedIn login integration  
- Lightweight JWT tokens

**Disadvantages:**
- Token revocation kompleks olabilir

**Alternatives:** Session-based auth, Auth0, Keycloak
**Decision:** Basit, standart, cost-effective

---

---

## 🔒 Security Architecture

### Authentication Flow
```
1. User → Login with GitHub/LinkedIn
2. OAuth Provider → Authorization Code
3. Backend → Exchange code for access token
4. Backend → Create JWT token
5. Frontend → Store JWT in httpOnly cookie
6. Subsequent requests → JWT in Authorization header
```

### Data Security
- Password hashing: bcrypt
- HTTPS only (TLS/SSL)
- Rate limiting: 100 requests/hour per user
- Input validation: Pydantic models
- SQL injection prevention: SQLAlchemy ORM
- CORS policy: Whitelisted domains
- API key rotation: Monthly

---

## 📊 Performans ve Scalability

### Caching Strategy
```python
# Redis cache for GitHub API responses
@cache(ttl=3600)  # 1 hour
async def fetch_github_repos(username: str):
    # Expensive API call
    pass
```

### Database Indexing
```sql
-- Full-text search index
CREATE INDEX idx_profile_skills ON profiles 
USING GIN (to_tsvector('english', skills));

-- User lookup index
CREATE INDEX idx_users_email ON users (email);
```

### API Rate Limiting
```python
from slowapi import Limiter

limiter = Limiter(key_func=get_remote_address)

@app.get("/api/profiles")
@limiter.limit("10/minute")
async def list_profiles():
    pass
```

---

## 🚀 Deployment Architecture
```mermaid
graph LR
    subgraph "Production Environment"
        LB[Load BalancerNginx]
        API1[FastAPI Instance 1]
        API2[FastAPI Instance 2]
        DB[(PostgreSQLPrimary)]
        DBR[(PostgreSQLReplica)]
        REDIS[(Redis Cache)]
    end

    LB --> API1
    LB --> API2
    API1 --> DB
    API2 --> DB
    API1 --> REDIS
    API2 --> REDIS
    DB --> DBR
```

### Deployment Platforms
- **Backend:** Railway, Render, Fly.io
- **Frontend:** Vercel, Netlify
- **Database:** Railway PostgreSQL, Supabase
- **Mobile:** App Store, Google Play Store

---

## 📝 API Endpoint Yapısı

### Authentication
```
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/refresh
GET    /api/auth/me
```

### Profiles
```
POST   /api/profiles/analyze    # Create profile from social media
GET    /api/profiles            # List all profiles (with filters)
GET    /api/profiles/{id}       # Get single profile
PUT    /api/profiles/{id}       # Update profile
DELETE /api/profiles/{id}       # Delete profile
```

### Skills
```
GET    /api/skills              # List all skills
GET    /api/skills/trending     # Trending skills
```

### Search
```
GET    /api/search?q=python&skills=django,fastapi
```

For detailed API documentation, see: [API_SPECIFICATION.md](./API_SPECIFICATION.md)

---

## 🔄 CI/CD Pipeline
```mermaid
graph LR
    CODE[Code Push] --> GH[GitHub Actions]
    GH --> TEST[Run Tests]
    TEST --> BUILD[Build Docker Image]
    BUILD --> DEPLOY[Deploy to Railway]
    DEPLOY --> VERIFY[Health Check]
```

---

## 📚 Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Angular Documentation](https://angular.io/docs)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [OpenAI API Reference](https://platform.openai.com/docs)

---

## Future Improvements

- [ ] GraphQL API support
- [ ] Real-time notifications (WebSockets)
- [ ] Machine Learning model training pipeline
- [ ] Multi-language support (i18n)
- [ ] Advanced analytics dashboard
- [ ] Kubernetes deployment

---


</details>
