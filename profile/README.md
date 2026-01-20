## Hi there 👋

### PL — Opis projektu (po polsku)

Praca inżynierska

Celem pracy jest zaprojektowanie i wykonanie systemu umożliwiającego zarządzanie pracą drukarki 3D w pracowni studenckiej. System pozwoli na przydzielanie czasu pracy, tworzenie i obsługę kolejki zadań drukowania oraz kontrolę dostępu użytkowników. Baza użytkowników będzie tworzona automatycznie na podstawie danych importowanych z systemu USOS (np. poprzez plik CSV).

W ramach projektu opracowany zostanie także moduł bezpieczeństwa, który umożliwi monitorowanie stanu drukarki i wykrywanie potencjalnych zagrożeń (np. pozostawionych wydruków lub przeszkód na stole roboczym). System zostanie zintegrowany z aplikacją webową, co umożliwi zdalny podgląd stanu drukarki oraz zarządzanie procesem drukowania w granicach możliwości technicznych urządzenia.

Projekt łączy elementy systemów wbudowanych, aplikacji webowych i automatyki, tworząc kompleksowe rozwiązanie wspierające organizację pracy drukarki 3D w laboratorium studenckim.

---

### EN — Project description (in English)

Engineering project

The goal of this project is to design and implement a system for managing 3D printer usage in a university laboratory. The system will support booking and allocation of printing time, creating and managing a print job queue, and enforcing user access control. The user database will be populated automatically from the university information system (for example via CSV import).

As part of the project, a safety module will be developed to monitor the printer's status and detect potential hazards (for example, abandoned prints or obstacles on the build plate). The system will be integrated with a web application to allow remote monitoring and management of the printing process within the technical capabilities of the printer.

The project combines embedded systems, web applications and automation to provide a comprehensive solution that helps organise and operate 3D printing facilities in a student laboratory.

# AddiPi - System Zarządzania Drukarka 3D

![License](https://img.shields.io/badge/license-Private-red)
![Status](https://img.shields.io/badge/status-Active-green)
![TypeScript](https://img.shields.io/badge/TypeScript-5.2-blue)
![React](https://img.shields.io/badge/React-18.2-61dafb)
![Node.js](https://img.shields.io/badge/Node.js-18+-green)
![Python](https://img.shields.io/badge/Python-3.10+-blue)

**AddiPi** to zaawansowany, rozproszonymi architekturą mikroserwisów system do zarządzania drukarkami 3D. Projekt umożliwia użytkownikom przesyłanie plików G-code, monitorowanie stanu druku w czasie rzeczywistym, zarządzanie pracami druku oraz administracją użytkownikami poprzez nowoczesny interfejs webowy.

---

## 📋 Spis treści

- [Przegląd projektu](#-przegląd-projektu)
- [Architektura systemu](#-architektura-systemu)
- [Komponenty projektu](#-komponenty-projektu)
- [Technologie](#-technologie)
- [Wymagania](#-wymagania)
- [Instalacja i uruchomienie](#-instalacja-i-uruchomienie)
- [Struktura projektu](#-struktura-projektu)
- [Zmienne środowiskowe](#-zmienne-środowiskowe)
- [API i komunikacja](#-api-i-komunikacja)
- [Wdrażanie produkcyjne](#-wdrażanie-produkcyjne)
- [Rozwiązywanie problemów](#-rozwiązywanie-problemów)
- [Contributing](#-contributing)
- [Licencja](#-licencja)

---

## 🎯 Przegląd projektu

**AddiPi** to kompleksowe rozwiązanie dla zarządzania drukarka 3D w środowisku szkoły/laboratorium. System składa się z kilku niezależnych mikroserwisów komunikujących się ze sobą poprzez Azure Services (Service Bus, IoT Hub, Cosmos DB, Blob Storage) oraz frontendu webowego zbudowanego w React.

### Główne funkcjonalności:

- **🔐 Bezpieczna autentykacja** - JWT-based authentication z obsługą access/refresh tokenów
- **📁 Zarządzanie plikami** - Przesyłanie i przechowywanie plików G-code w Azure Blob Storage
- **📊 Monitorowanie stanu** - Śledzenie stanu druku w czasie rzeczywistym
- **📈 Zarządzanie kolejką** - Obsługa kolejki zadań druku poprzez Azure Service Bus
- **👥 Panel administracyjny** - Zarządzanie użytkownikami i wszystkimi pracami
- **🖨️ Kontrola drukarki** - Wysyłanie komend drukowania do urządzeń poprzez Azure IoT Hub
- **🎬 Wideo** - Obsługa stream'ów wideo z drukarek
- **🤖 Agent AI** - Zaawansowane funkcje zautomatyzowanego planowania druku

---

## 🏗️ Architektura systemu

```
┌──────────────────────────────────────────────────────────────────────┐
│                         Frontend (React)                             │
│                     AddiPi-Frontend (Port 3000)                      │
└────────┬─────────────────────────────────┬───────────────────────────┘
         │                                 │
         │ REST API Calls                  │ REST API Calls
         │                                 │
┌────────▼──────────────┐    ┌─────────────▼────────────┐
│  Auth Service         │    │  User Service            │
│  (Port 3001)          │    │  (Port 3002)             │
│  - Rejestracja        │    │  - Profil użytkownika    │
│  - Login/Logout       │    │  - Dane użytkownika      │
│  - JWT Tokens         │    │  - Role i uprawnienia    │
└────────┬──────────────┘    └──────────────┬───────────┘
         │                                  │
         └──────────────┬───────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
┌───────▼──────────┐ ┌──▼──────────┐ ┌──▼─────────────┐
│ Files Service    │ │ Queue       │ │ Printer Service│
│ (Port 5000)      │ │ Service     │ │ (Port 3050)    │
│ - Upload G-code  │ │ (Port 3070) │ │ - Scheduler    │
│ - Blob Storage   │ │ - Jobs      │ │ - IoT Commands │
│ - Service Bus    │ │ - Status    │ │ - Job Status   │
└──────────────────┘ └─────────────┘ └────────────────┘
                       │
        ┌──────────────┼──────────────┬─────────────┐
        │              │              │             │
   ┌────▼────┐   ┌─────▼────┐  ┌──────▼───┐  ┌──────▼──┐
   │ Service │   │ Cosmos   │  │ IoT Hub  │  │ Blob    │
   │  Bus    │   │   DB     │  │          │  │Storage  │
   └─────────┘   └──────────┘  └──────────┘  └─────────┘
        │                           |
┌───────▼──────────┐     ┌──────────▼───────┐
│ Video Service    │     │ Agent Service    │
│ (Port 3003)      │     │ (Python)         │
│ - Stream video   │     │ - AI features    │
│ - Recording      │     │ - Automation     │
└──────────────────┘     └──────────────────┘
```

---

## 🔧 Komponenty projektu

### 1. **AddiPi-Frontend**
Nowoczesny interfejs webowy zbudowany w React + TypeScript.

- **Technologia**: React 18, TypeScript, Vite, Tailwind CSS
- **Port**: 3000
- **Funkcjonalności**:
  - Rejestracja i logowanie użytkowników
  - Przesyłanie plików G-code
  - Monitorowanie stanu druku
  - Panel administracyjny
  - Dashboard z metrykami
  - Real-time aktualizacja statusu

[Więcej szczegółów →](AddiPi-Frontend/README.md)

### 2. **AddiPi-Auth-Service**
Mikroserwis odpowiedzialny za autentykację i autoryzację.

- **Technologia**: Node.js, Express, TypeScript, Azure Cosmos DB
- **Port**: 3001
- **Funkcjonalności**:
  - Rejestracja użytkowników
  - Login z JWT tokens (access + refresh)
  - Weryfikacja tokenów
  - Refresh tokenów
  - Bezpieczne hashowanie haseł (bcryptjs)

**Ograniczenie**: Domyślnie akceptuje tylko e-maile z domeny `@uwr.edu.pl` (można zmienić w konfiguracji)

[Więcej szczegółów →](AddiPi-Auth-Service/README.md)

### 3. **AddiPi-User-Service**
Serwis zarządzania profilem i danymi użytkowników.

- **Technologia**: Node.js, TypeScript
- **Port**: 3002
- **Funkcjonalności**:
  - Zarządzanie profilami użytkowników
  - Role i uprawnienia
  - Edycja danych użytkownika

[Więcej szczegółów →](AddiPi-User-Service/README.md)

### 4. **AddiPi-Files-Service**
Lekki mikroserwis do obsługi przesyłania i przechowywania plików.

- **Technologia**: Python, Flask
- **Port**: 5000
- **Funkcjonalności**:
  - Przesyłanie plików G-code
  - Przechowywanie w Azure Blob Storage
  - Powiadomienia poprzez Azure Service Bus
  - Walidacja plików
  - Ograniczenia rozmiaru (domyślnie 50 MB)

**Endpoint API**:
- `POST /upload` - Przesyłanie pliku
- `GET /health` - Health check
- `GET /files/recent` - Lista ostatnich plików

[Więcej szczegółów →](AddiPi-Files-Service/README.md)

### 5. **AddiPi-Queue-Service**
Serwis zarządzania kolejką zadań druku.

- **Technologia**: Node.js, TypeScript, Express
- **Port**: 3070
- **Funkcjonalności**:
  - Nasłuchiwanie zdarzeń z Azure Service Bus
  - Przechowywanie zadań w Cosmos DB
  - HTTP API do przeglądania kolejki
  - Zarządzanie statusem zadań
  - Notyfikacje o nowych zadaniach

[Więcej szczegółów →](AddiPi-Queue-Service/README.md)

### 6. **AddiPi-Printer-Service**
Serwis obsługi drukarek i wykonywania zadań druku.

- **Technologia**: Node.js, TypeScript, Express
- **Port**: 3050
- **Funkcjonalności**:
  - Planowanie zadań druku (cron)
  - Wysyłanie komend do urządzeń via Azure IoT Hub
  - Zarządzanie statusem pracy
  - Aktualizacja stanu w Cosmos DB
  - Health checks

**Planner**: Uruchomiany co minutę, sprawdza zaplanowane zadania

[Więcej szczegółów →](AddiPi-Printer-Service/README.md)

### 7. **AddiPi-Video-Service**
Serwis do obsługi video z drukarek.

- **Technologia**: Node.js, TypeScript
- **Port**: 3003
- **Funkcjonalności**:
  - Stream video z drukarek
  - Nagrywanie procesu druku
  - Integracja z Azure Storage

[Więcej szczegółów →](AddiPi-Video-Service/README.md)

### 8. **AddiPi-Agent**
Agent AI do zaawansowanych funkcji automatyzacji.

- **Technologia**: Python
- **Funkcjonalności**:
  - Inteligentne planowanie zadań
  - Optimizacja kolejki druku
  - Analiza danych druku

[Więcej szczegółów →](AddiPi-Agent/README.md)

### 9. **AddiPi-Infrastructure**
Konfiguracja infrastruktury i orchestracji serwisów.

- **Narzędzia**: Docker, Docker Compose
- **Zawiera**: docker-compose.yml, skrypty wdrażające, dokumentacja

[Więcej szczegółów →](AddiPi-Infrastructure/README.md)

---

## 🛠 Technologie

### Frontend
- **React 18.2** - Biblioteka UI
- **TypeScript 5.2** - Statyczne typowanie
- **Vite 5.1** - Bundler
- **Tailwind CSS 3.4** - Styling
- **React Router 6.22** - Routing
- **Zustand 4.5** - State management
- **Axios 1.6** - HTTP client

### Backend (Node.js services)
- **Express 5.1** - Web framework
- **TypeScript 5.2** - Statyczne typowanie
- **Azure SDK**:
  - `@azure/cosmos` - Cosmos DB
  - `azure-iot-device` - IoT Hub
  - `azure-storage-blob` - Blob Storage
  - `azure-servicebus` - Service Bus
- **JWT** - Token-based auth
- **bcryptjs** - Password hashing

### Backend (Python services)
- **Flask 2.3** - Web framework
- **Azure SDK**:
  - `azure-storage-blob` - Blob Storage
  - `azure-servicebus` - Service Bus
  - `azure-iot-device` - IoT Hub
- **python-dotenv** - Environment variables

### Infrastruktura
- **Docker** - Containerization
- **Docker Compose** - Orchestracja lokalnie
- **Azure Services**:
  - Azure Cosmos DB (NoSQL)
  - Azure Service Bus (Messaging)
  - Azure IoT Hub (Device communication)
  - Azure Blob Storage (File storage)
  - Azure Container Registry (Images)

---

## 📦 Wymagania

### Globalne
- **Git** - Kontrola wersji
- **Docker** >= 20.x
- **Docker Compose** >= 2.x

### Local Development

#### Node.js Services
- **Node.js** >= 18.x (zalecane 20.x)
- **npm** >= 9.x lub **yarn** >= 1.22.x

#### Python Services
- **Python** >= 3.10
- **pip** >= 21.x

### Azure Services (production/staging)
- **Azure Subscription**
- **Azure Cosmos DB** (SQL API)
- **Azure Service Bus**
- **Azure IoT Hub**
- **Azure Blob Storage**
- **Azure Container Registry** (opcjonalnie)

---

## 🚀 Instalacja i uruchomienie

### Opcja 1: Docker Compose (zalecane dla całego systemu)

1. **Klonowanie repozytorium**:
```powershell
git clone https://github.com/AddiPii/AddiPi.git
cd AddiPi
```

2. **Konfiguracja zmiennych środowiskowych**:
```powershell
cd AddiPi-Infrastructure
# Utwórz lub zmodyfikuj plik .env z wymaganymi zmiennymi
# (patrz sekcja Zmienne środowiskowe)
cp .env.example .env
# Edytuj .env swoimi wartościami
```

3. **Uruchomienie wszystkich serwisów**:
```powershell
docker-compose up --build
```

Lub w tle:
```powershell
docker-compose up -d --build
docker-compose logs -f
```

4. **Dostęp do aplikacji**:
- Frontend: http://localhost:3000
- Auth Service: http://localhost:3001
- User Service: http://localhost:3002
- Files Service: http://localhost:5000
- Queue Service: http://localhost:3070
- Printer Service: http://localhost:3050
- Video Service: http://localhost:3003

5. **Zatrzymanie serwisów**:
```powershell
docker-compose down
```

### Opcja 2: Lokalne uruchomienie poszczególnych serwisów

#### Frontend
```powershell
cd AddiPi-Frontend
npm install
npm run dev
# Dostępne na http://localhost:5173 (Vite dev server)
```

#### Auth Service
```powershell
cd AddiPi-Auth-Service
npm install
npm run build
npm start
```

#### Files Service
```powershell
cd AddiPi-Files-Service
python -m pip install -r requirements.txt
python app.py
```

#### Queue Service
```powershell
cd AddiPi-Queue-Service
npm install
npm start
```

#### Printer Service
```powershell
cd AddiPi-Printer-Service
npm install
npm run build
npm start
```

---

## 📁 Struktura projektu

```
AddiPi/
├── AddiPi-Frontend/              # React UI aplikacja
│   ├── src/
│   │   ├── components/           # Komponenty React
│   │   ├── pages/                # Strony (Dashboard, Admin, Login)
│   │   ├── services/             # Serwisy API (Axios)
│   │   ├── hooks/                # Custom React hooks
│   │   ├── store/                # Zustand store
│   │   ├── types/                # TypeScript interfejsy
│   │   └── utils/                # Utility functions
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   └── README.md
│
├── AddiPi-Auth-Service/          # JWT Authentication
│   ├── src/
│   │   ├── controllers/          # Logika biznesowa
│   │   ├── routes/               # Express routes
│   │   ├── config/               # Konfiguracja
│   │   ├── helpers/              # Helper functions
│   │   └── services/             # Cosmos DB, JWT
│   ├── package.json
│   ├── Dockerfile
│   └── README.md
│
├── AddiPi-User-Service/          # User Management
│   ├── src/
│   ├── package.json
│   ├── Dockerfile
│   └── README.md
│
├── AddiPi-Files-Service/         # G-code Upload & Storage
│   ├── app.py                    # Flask app
│   ├── controllers/              # Handler functions
│   ├── routes/                   # Flask blueprints
│   ├── middleware/               # Auth middleware
│   ├── config/
│   ├── requirements.txt
│   ├── Dockerfile
│   └── README.md
│
├── AddiPi-Queue-Service/         # Job Queue Management
│   ├── index.js                  # Entry point
│   ├── listeners/                # Service Bus listeners
│   ├── controllers/              # Route handlers
│   ├── services/                 # Cosmos, Service Bus clients
│   ├── package.json
│   ├── Dockerfile
│   └── README.md
│
├── AddiPi-Printer-Service/       # Print Job Execution
│   ├── src/
│   │   ├── index.ts              # Entry point
│   │   ├── pi-device.ts          # Device communication
│   │   ├── controllers/          # API handlers
│   │   ├── services/             # Cosmos, IoT services
│   │   └── middleware/           # Express middleware
│   ├── package.json
│   ├── Dockerfile
│   └── README.md
│
├── AddiPi-Video-Service/         # Video Stream Management
│   ├── src/
│   ├── package.json
│   ├── Dockerfile
│   └── README.md
│
├── AddiPi-Agent/                 # AI Agent
│   ├── src/
│   │   ├── app.py                # Main app
│   │   ├── agent/                # Agent logic
│   │   ├── config/               # Configuration
│   │   └── utils/                # Utilities
│   ├── requirements.txt
│   ├── Dockerfile
│   └── README.md
│
├── AddiPi-Infrastructure/        # Docker & Deployment
│   ├── docker-compose.yml        # Local orchestration
│   ├── addipi-pod.yml            # Kubernetes config
│   ├── deploy-files-service.sh   # Deployment scripts
│   ├── infra.sh
│   ├── cosmos.sh
│   ├── .env.example
│   └── README.md
│
├── AddiPi-Test-Frontend/         # Test Frontend
│
└── README.md                      # Ten plik
```

---

## 🔑 Zmienne środowiskowe

Utwórz plik `.env` w `AddiPi-Infrastructure/` z następującymi zmiennymi:

### Azure Services

```env
# Azure Cosmos DB
COSMOS_ENDPOINT=https://<account>.documents.azure.com:443/
COSMOS_KEY=<primary-key>

# Azure Service Bus
SERVICE_BUS_CONN=Endpoint=sb://<namespace>.servicebus.windows.net/;SharedAccessKeyName=<policy>;SharedAccessKey=<key>
QUEUE_NAME=print-queue

# Azure Storage Blob
STORAGE_CONN=DefaultEndpointsProtocol=https;AccountName=<name>;AccountKey=<key>;EndpointSuffix=core.windows.net

# Azure IoT Hub
IOT_CONN_STRING=HostName=<hub>.azure-devices.net;SharedAccessKeyName=<policy>;SharedAccessKey=<key>
IOT_HUB_SERVICE_CS=HostName=<hub>.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey=<key>
```

### JWT Configuration

```env
# Auth Service
JWT_SECRET=<long-random-secret-key>
JWT_REFRESH_SECRET=<long-random-refresh-secret>
JWT_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d
```

### Service Ports

```env
PORT=3000
PRINTER_PORT=3050
QUEUE_PORT=3070
FILES_PORT=5000
USER_PORT=3002
VIDEO_PORT=3003
```

### File Upload Configuration

```env
ALLOWED_EXTENSIONS=.gcode
MAX_UPLOAD_SIZE=52428800  # 50MB
STRICT_CONTENT_CHECK=0
```

### Email Configuration (Auth Service)

```env
SMTP_HOST=<mail-server>
SMTP_PORT=587
SMTP_USER=<email>
SMTP_PASSWORD=<password>
EMAIL_DOMAIN=@uwr.edu.pl  # Allowlist domain
```

---

## 🔌 API i komunikacja

### Frontend ↔ Backend

Frontend komunikuje się z backend serwisami poprzez REST API (HTTP/HTTPS):

```
Frontend (React) 
  └─→ Axios HTTP Client
      ├─→ Auth Service (3001) - /auth/login, /auth/register, /auth/verify
      ├─→ User Service (3002) - /user/profile, /user/update
      ├─→ Files Service (5000) - /upload, /files/recent
      ├─→ Queue Service (3070) - /api/jobs, /api/queue
      ├─→ Printer Service (3050) - /printer/health, /printer/jobs
      └─→ Video Service (3003) - /stream, /status
```

### Backend ↔ Backend

Backend serwisy komunikują się poprzez:

1. **Azure Service Bus** - Asynchroniczna komunikacja (events, messaging)
2. **Azure Cosmos DB** - Wspólny data store
3. **HTTP REST** - Direct API calls (z retry logic)

```
Files Service 
  └─→ POST /upload
      └─→ Azure Service Bus (print-queue)
          └─→ Queue Service (nasłuchuje)
              └─→ Cosmos DB (jobs collection)
                  └─→ Printer Service (scheduler)
                      └─→ Azure IoT Hub
                          └─→ Device (Raspberry Pi z G-code reader)
```

---

## 🛡️ Bezpieczeństwo

### Autentykacja
- **JWT (JSON Web Tokens)** - Access tokens (15 minut) i refresh tokens (7 dni)
- **Access Token** - Przesyłany w `Authorization: Bearer <token>` header
- **Refresh Token** - Przechowywany w HttpOnly cookie (zalecane) lub localStorage

### Middleware
- `require_auth` - Weryfikacja JWT na każdy request do zabezpieczonych endpointów
- `require_admin` - Dodatkowa autoryzacja dla admin endpoints
- CORS - Ograniczenie dostępu z obcych domenach

### Hasła
- **bcryptjs** - Bezpieczne haszowanie haseł (salt rounds: 10)
- **Validator** - Walidacja pola email i hasła

### Best Practices
- Użyj silnych sekretów dla JWT (minimum 32 znaki)
- Przechowuj Azure keys w Azure Key Vault (nie w .env)
- Włącz HTTPS w produkcji
- Ustaw odpowiednie cookie attributes (Secure, HttpOnly, SameSite)
- Regularnie rotuj tokeny refresh

---

## 📈 Wdrażanie produkcyjne

### Azure Container Instances (ACI)

Każdy serwis ma `Dockerfile` do konteneryzacji:

```powershell
# Build image
docker build -t addipi-auth-service:latest ./AddiPi-Auth-Service

# Push to Azure Container Registry
az acr build --registry <registry-name> --image addipi-auth-service:latest ./AddiPi-Auth-Service

# Deploy to ACI
az container create --resource-group <rg> \
  --name addipi-auth \
  --image <registry>.azurecr.io/addipi-auth-service:latest \
  --ports 3001 \
  --environment-variables KEY=value \
  --restart-policy Always
```

### Kubernetes (AKS)

Dostępne są manifesty Kubernetes:

```powershell
# Deploy do AKS
kubectl apply -f AddiPi-Infrastructure/addipi-pod.yml
```

### Monitoring

Każdy serwis eksponuje `/health` endpoint:

```powershell
curl http://localhost:3001/health
curl http://localhost:5000/health
curl http://localhost:3070/health
```

---

## 🔧 Rozwiązywanie problemów

### Serwis nie startuje - Docker Compose

1. **Sprawdź logi**:
```powershell
docker-compose logs -f <service-name>
```

2. **Weryfikuj zmienne środowiskowe**:
```powershell
docker-compose config  # Wyświetla resolved config
```

3. **Przebuduj images**:
```powershell
docker-compose down
docker system prune -a
docker-compose up --build
```

### Błędy konektywności Azure

1. **Sprawdź connection strings**:
```powershell
# Cosmos DB
$env:COSMOS_ENDPOINT  # Powinno być https://...documents.azure.com:443/

# Service Bus
$env:SERVICE_BUS_CONN  # Powinno zaczynać się z "Endpoint=sb://"
```

2. **Testuj konekt do Azure**:
```powershell
# Cosmos DB
curl -X POST "https://<account>.documents.azure.com/dbs" \
  -H "Authorization: type=master&ver=1.0&sig=<key>"

# Service Bus (wymaga azcli)
az servicebus queue peek --resource-group <rg> \
  --namespace-name <ns> --name print-queue
```

### Frontend nie łączy się z backend

1. **Sprawdź CORS** (backend powinien mieć `cors()` middleware)
2. **Weryfikuj adresy IP/hostname**:
   - LocalHost: `http://localhost:PORT`
   - Docker: użyj service names z docker-compose.yml (np. `http://auth:3001`)
3. **Sprawdź firewall/bezpieczeństwo sieciowe**

### JWT Token expiration

- **Access Token**: 15 minut - automatycznie refresh za pomocą refresh tokena
- **Refresh Token**: 7 dni - wymaga nowego logowania po wygaśnięciu
- Upewnij się że backend wysyła prawidłowe `exp` claim w tokenach

---

## 📚 Dokumentacja poszczególnych serwisów

Każdy serwis ma dedykowany README:

- [AddiPi-Frontend](AddiPi-Frontend/README.md) - React UI z komponentami i state management
- [AddiPi-Auth-Service](AddiPi-Auth-Service/README.md) - JWT, login, rejestracja
- [AddiPi-User-Service](AddiPi-User-Service/README.md) - Profile użytkowników
- [AddiPi-Files-Service](AddiPi-Files-Service/README.md) - Upload G-code, Blob Storage
- [AddiPi-Queue-Service](AddiPi-Queue-Service/README.md) - Job queue, Service Bus listener
- [AddiPi-Printer-Service](AddiPi-Printer-Service/README.md) - Scheduler, IoT commands
- [AddiPi-Video-Service](AddiPi-Video-Service/README.md) - Video streaming
- [AddiPi-Agent](AddiPi-Agent/README.md) - AI agent, automation
- [AddiPi-Infrastructure](AddiPi-Infrastructure/README.md) - Docker, deployment scripts

---

## 👥 Contributing

### Development Workflow

1. **Utwórz feature branch**:
```bash
git checkout -b feature/my-feature
```

2. **Commituj zmiany**:
```bash
git commit -m "feat: Add new feature"
git push origin feature/my-feature
```

3. **Otwórz Pull Request** na GitHub

### Code Standards

- **TypeScript**: Strict mode, proper typing
- **Python**: PEP 8, type hints
- **Commits**: Conventional Commits (feat:, fix:, docs:, etc.)
- **Branches**: lowercase, kebab-case (feature/add-printer-support)

### Testing

Każdy serwis powinien mieć testy jednostkowe i integracyjne:

```powershell
# Frontend
npm test

# Python services
pytest

# Node services
npm test
```

---

## 📝 Licencja

Projekt jest zamknięty (Private) i dostępny tylko dla autoryzowanych użytkowników.

```
Copyright © 2024-2026 AddiPi Team
All rights reserved.
```

---

## 📞 Kontakt i Support

- **Issues**: Zgłaszaj błędy na GitHub Issues
- **Discussions**: Dyskusje o nowych funkcjonalnościach
- **Email**: [contact@addipi.local]

---

## 🎯 Mapa rozwoju projektu (Roadmap)

### v1.1 (Planowany)
- [ ] Optymalizacja UI mobilnego
- [ ] Wsparcie dla wielu drukarek jednocześnie
- [ ] Advanced job scheduling (AI-based)
- [ ] Email notifications
- [ ] Two-Factor Authentication (2FA)

### v1.2 (Przyszłość)
- [ ] Real-time video streaming (WebRTC)
- [ ] Machine learning dla predykcji czasu druku
- [ ] Mobile app (React Native)
- [ ] GraphQL API (obok REST)

---

**Ostatnia aktualizacja**: 20 stycznia 2026  
**Status**: Aktywnie rozwijany ✅


