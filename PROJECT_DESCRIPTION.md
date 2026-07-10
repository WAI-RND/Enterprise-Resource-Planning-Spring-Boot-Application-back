# SecureOps — Enterprise Resource Planning & Security Operations Platform

---

## 1. Short Summary (CV-ready)

Full-stack Enterprise Resource Planning (ERP) platform built with Spring Boot 3.4 and React 19, featuring real-time WebSocket chat, a security threat detection engine analyzing 11 attack vectors from audit logs, and modular business domains (HR, Sales, Calendar, Admin). Deployed as a Dockerized three-container stack (PostgreSQL, Spring Boot, Nginx-served React SPA) with JWT authentication, role-based access control, and a comprehensive REST API spanning 80+ endpoints.

---

## 2. Problem Statement

Small-to-medium enterprises need a unified platform to manage core business operations — HR, sales pipelines, internal communication, task scheduling, and security monitoring — without juggling disconnected tools. SecureOps consolidates these functions into a single web application with centralized authentication, audit logging, and real-time threat detection.

---

## 3. Key Features / Functionalities

| Module | Capabilities |
|--------|-------------|
| **Authentication & Authorization** | JWT-based stateless auth, BCrypt password hashing, brute-force lockout (5 attempts / 15-min window), Google reCAPTCHA v3, admin-gated user approval workflow, password reset via email verification codes |
| **Security Dashboard** | Log-based threat detection for 11 attack categories (SQL injection, XSS, path traversal, RCE, LFI, brute-force, DDoS, malware, privilege escalation, data exfiltration, unauthorized access); real-time alert summary with severity classification; top-attacking-IP analysis; exportable threat reports |
| **Real-Time Chat** | STOMP-over-WebSocket messaging with SockJS fallback, JWT-authenticated WebSocket connections, text and file message types (JPA JOINED inheritance), typing indicators, read receipts, conversation archiving |
| **Sales Pipeline** | Full quote-to-order-to-invoice lifecycle with automatic entity conversion, line-item management, client and product catalogs, payment tracking, client-side PDF generation (jsPDF), and analytics reports (revenue summary, per-product sales, client spending, employee performance) |
| **HR Management** | Employee CRUD with address, status (Active/Inactive/On Leave), task assignment with priority levels and overdue detection |
| **Calendar & Tasks** | Four calendar views (Month/Week/Day/Year), global and user-scoped events, task assignment to multiple users, event conflict detection, admin-only event creation |
| **Admin Panel** | Pending-user approvals, role assignment, audit log viewer with user/action/date filtering, security event management |
| **File Management** | Multipart upload (50 MB limit), UUID-based filename storage, path-traversal prevention, download and inline-view endpoints |

---

## 4. Technical Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Client Browser                      │
│  React 19 SPA (Vite build → Nginx static serve)        │
│  Context API + Hooks state │ Axios HTTP │ STOMP WS     │
└──────────────┬──────────────────────┬───────────────────┘
               │ REST / JSON          │ WebSocket (STOMP)
               ▼                      ▼
┌──────────────────────────────────────────────────────────┐
│  Nginx Reverse Proxy (port 80)                           │
│  /api/* → backend:8080   |   /ws/* → backend:8080 (WS)  │
│  /* → SPA index.html (client-side routing fallback)      │
└──────────────┬──────────────────────┬────────────────────┘
               │                      │
               ▼                      ▼
┌──────────────────────────────────────────────────────────┐
│  Spring Boot 3.4.5 (Java 21)                             │
│                                                          │
│  ┌─────────────────┐  ┌──────────────┐  ┌────────────┐  │
│  │  Controllers     │  │  WebSocket   │  │  Security  │  │
│  │  (REST API)      │  │  (STOMP)     │  │  (JWT +    │  │
│  │  10 core + 8     │  │  Real-time   │  │   RBAC)    │  │
│  │  sales/HR ctrl   │  │  messaging   │  │            │  │
│  └────────┬─────────┘  └──────┬───────┘  └────────────┘  │
│           │                   │                           │
│  ┌────────▼───────────────────▼──────────────────┐       │
│  │            Service Layer (18 services)          │       │
│  │  Auth · User · Chat · Message · Alerts · Log   │       │
│  │  TaskEvent · File · Email · Recaptcha           │       │
│  │  + Sales (7) · HR (5)                           │       │
│  └────────────────────┬──────────────────────────┘       │
│                       │                                   │
│  ┌────────────────────▼──────────────────────────┐       │
│  │         Repository Layer (Spring Data JPA)      │       │
│  │  18 repositories · Custom JPQL/native queries   │       │
│  └────────────────────┬──────────────────────────┘       │
└───────────────────────┼──────────────────────────────────┘
                        │ JDBC
                        ▼
            ┌───────────────────────┐
            │  PostgreSQL 16        │
            │  (Docker volume)      │
            └───────────────────────┘
```

**Deployment:** Docker Compose orchestrating three services (`db`, `backend`, `frontend`) with health-check dependencies and persistent volume for PostgreSQL data.

---

## 5. Technologies Used

### Backend
| Technology | Version | Role |
|-----------|---------|------|
| Java | 21 | Language runtime |
| Spring Boot | 3.4.5 | Application framework |
| Spring Security | 6.x | Authentication & authorization |
| Spring Data JPA / Hibernate | — | ORM & data access |
| Spring WebSocket (STOMP) | — | Real-time messaging |
| Spring Mail | — | Email notifications |
| Spring Actuator | — | Health & metrics endpoints |
| PostgreSQL | 16 | Relational database |
| JJWT | 0.11.5 | JWT token handling |
| Lombok | 1.18.30 | Boilerplate reduction |
| MapStruct | 1.5.5 | DTO ↔ Entity mapping |
| SpringDoc OpenAPI | 2.2.0 | Swagger API documentation |
| Logback | — | Structured logging |

### Frontend
| Technology | Version | Role |
|-----------|---------|------|
| React | 19.0.0 | UI library |
| React Router DOM | 7.5.3 | Client-side routing |
| Vite | 6.2.0 | Build tooling & dev server |
| Tailwind CSS | 4.1.5 | Utility-first styling |
| Axios | 1.9.0 | HTTP client |
| @stomp/stompjs + SockJS | 7.1.1 / 1.6.1 | WebSocket client |
| Recharts + Chart.js | 2.15.3 / 4.4.1 | Data visualization |
| Framer Motion | 12.16.0 | Animations |
| React Hook Form + Yup | 7.56.2 / 1.6.1 | Form state & validation |
| jsPDF + autoTable | 3.0.1 / 5.0.2 | Client-side PDF generation |
| @tanstack/react-query | 5.21.7 | Server state caching |
| @tanstack/react-table | 8.11.8 | Headless data tables |
| Ant Design | 5.14.1 | UI component library |
| date-fns | 3.3.1 | Date utilities |
| react-google-recaptcha | 3.1.0 | Bot protection |

### Infrastructure
| Technology | Role |
|-----------|------|
| Docker / Docker Compose | Containerized deployment |
| Nginx | Reverse proxy & static file server |
| PostgreSQL 16 (Alpine) | Persistence with health-check readiness |

---

## 6. Important Algorithms & Logic

### Threat Detection Engine (`AlertsService`)
Analyzes audit logs via regex pattern matching across 11 attack categories:
- **Brute-force detection** — flags IPs with repeated failed-login entries above a configurable threshold within a time window.
- **SQL injection / XSS / Path traversal / RCE / LFI** — scans log `details` and `action` fields for known malicious patterns (e.g., `UNION SELECT`, `<script>`, `../`, command-injection signatures).
- **DDoS detection** — identifies abnormally high request volumes from single IPs.
- **Privilege escalation & unauthorized access** — detects role-change attempts and access to forbidden resources.
- Alerts are classified by severity (HIGH / MEDIUM / LOW) and enriched with timestamp, IP, and affected-user metadata.

### Brute-Force Lockout (`AuthController`)
In-memory `ConcurrentHashMap` tracks failed login attempts per IP address. After 5 consecutive failures the IP is locked out for 15 minutes. Successful logins clear the counter.

### Quote → Order → Invoice Lifecycle (Sales Module)
Automated entity conversion pipeline: a `Quote` (DRAFT → SENT → ACCEPTED) can be converted into an `Order` via a dedicated endpoint, which copies line items and recalculates totals; an `Order` (CONFIRMED → SHIPPED → DELIVERED) generates an `Invoice` with payment-due tracking and PDF export.

### JWT Authentication Pipeline
Request → `JwtAuthenticationFilter` extracts Bearer token → `JwtTokenProvider` validates HMAC-SHA256 signature and expiration → extracts username and authorities from claims → `CustomUserDetailsService` verifies user is active and approved → sets `SecurityContextHolder` → request proceeds to endpoint authorization.

### WebSocket Authentication
STOMP CONNECT frames carry the JWT in a custom header. `WebSocketChannelInterceptor` validates the token and binds the authenticated principal to the WebSocket session, ensuring all subsequent messages are authorized.

### Calendar Event Conflict Detection (Frontend)
The `CalendarContext` reducer checks for overlapping time ranges when creating or editing events, preventing double-booking within the same user's schedule.

---

## 7. API & Data Flow

### Authentication Flow
```
Client                          Backend                         DB
  │── POST /api/auth/login ────▶│                               │
  │   {email, password,          │── brute-force check (IP) ──▶ │
  │    recaptchaResponse}        │◀─ validate reCAPTCHA         │
  │                              │── authenticate ───────────▶  │
  │                              │◀─ User (active + approved)   │
  │◀─ {token, user} ───────────│── generate JWT (24 h)         │
  │                              │── log LOGIN event ────────▶  │
```

### Real-Time Chat Flow
```
Client A                    WebSocket Broker              Client B
  │── CONNECT (JWT) ────────▶│                               │
  │── SUBSCRIBE /topic/chat/1│                               │
  │                           │◀── SUBSCRIBE /topic/chat/1 ──│
  │── SEND /app/chat/1/ ────▶│                               │
  │   sendMessage {content}   │── persist Message to DB       │
  │                           │── BROADCAST /topic/chat/1 ──▶│
  │◀── MESSAGE ──────────────│                               │
```

### Sales Pipeline Flow
```
Create Client → Create Product(s)
       │                │
       ▼                ▼
   Create Quote (with QuoteItems referencing Products)
       │
       ▼  POST /quotes/{id}/convert-to-order
   Create Order (copies line items, calculates totals)
       │
       ▼  POST /orders/{id}/invoice
   Create Invoice (payment due date, status tracking)
       │
       ▼  PUT /invoices/{id}/mark-paid
   Invoice PAID → GET /invoices/{id}/pdf (client-side PDF export)
```

---

## 8. Challenges Solved

| Challenge | Solution |
|-----------|----------|
| **Stateless auth over both REST and WebSocket** | JWT validated in HTTP filter for REST; custom STOMP channel interceptor validates the same token on WebSocket CONNECT, unifying auth across protocols |
| **Polymorphic message storage** | JPA JOINED inheritance strategy with a discriminator column lets `TextMessage` and `FileMessage` share a single query interface while storing type-specific fields in separate tables |
| **Brute-force protection without external dependencies** | In-memory `ConcurrentHashMap<String, AttemptInfo>` keyed by IP; no Redis or external cache required for single-instance deployments |
| **File upload security** | UUID-based renaming prevents filename collisions and eliminates directory-traversal attacks; size validation enforced at both Nginx and Spring levels |
| **Multi-module domain separation** | Sales and HR modules are isolated into their own package trees (entity, controller, service, repository, DTO, exception), enabling independent development and clear bounded contexts |
| **Real-time UI consistency** | WebSocket message deduplication via `useRef`-tracked processed-message IDs prevents duplicate renders; optimistic UI updates provide instant feedback before server confirmation |
| **Admin approval workflow** | New users enter a PENDING state; only admin-approved users can authenticate, implemented as a status check inside `CustomUserDetailsService.loadUserByUsername()` |

---

## 9. Quantifiable Metrics (Code-Based)

| Metric | Value |
|--------|-------|
| REST API endpoints | **80+** across 18 controllers |
| Database entities | **18** (7 core + 4 HR + 7 Sales) with JPA relationships |
| Backend services | **30** (interfaces + implementations) |
| Frontend pages | **15+** with role-based routing |
| React components | **60+** organized across 8 module folders |
| Threat detection categories | **11** distinct attack vectors analyzed |
| WebSocket channels | Real-time chat + typing indicators + read receipts |
| Supported calendar views | **4** (Month / Week / Day / Year) |
| Sales entity lifecycle stages | **5** statuses per entity (Quote, Order, Invoice) |
| Docker services | **3** (PostgreSQL, Spring Boot, Nginx) |
| Java version | **21** (LTS) with Spring Boot **3.4.5** |
| Frontend dependencies | **28** production packages |
