<div align="center">

<h1>⚡ TVS EV Subscription Platform</h1>

<p>
  <strong>A production-grade, cloud-native backend for managing EV charging subscriptions, payments, feature usage, and automated notifications — built with Spring Boot microservices.</strong>
</p>

<p>
  <img src="https://img.shields.io/badge/Java-17-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white" />
  <img src="https://img.shields.io/badge/Spring_Boot-3.5.6-6DB33F?style=for-the-badge&logo=springboot&logoColor=white" />
  <img src="https://img.shields.io/badge/Spring_Cloud-2025.0.0-6DB33F?style=for-the-badge&logo=spring&logoColor=white" />
  <img src="https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Razorpay-Payment-02042B?style=for-the-badge&logo=razorpay&logoColor=white" />
</p>

<p>
  <img src="https://img.shields.io/badge/Architecture-Microservices-blueviolet?style=flat-square" />
  <img src="https://img.shields.io/badge/Auth-JWT%20%2B%20BCrypt-orange?style=flat-square" />
  <img src="https://img.shields.io/badge/Discovery-Eureka-green?style=flat-square" />
  <img src="https://img.shields.io/badge/Gateway-Spring%20Cloud%20Gateway-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/PDF-iText%207-red?style=flat-square" />
  <img src="https://img.shields.io/badge/CI-GitLab%20CI-FC6D26?style=flat-square&logo=gitlab" />
</p>

</div>

---

## 📋 Table of Contents

- [About the Project](#-about-the-project)
- [Architecture](#-architecture)
- [Services Overview](#-services-overview)
- [Tech Stack](#-tech-stack)
- [Key Features](#-key-features)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Database Setup](#database-setup)
  - [Running Locally](#running-locally)
  - [Running with Docker](#running-with-docker)
- [API Overview](#-api-overview)
- [End-to-End Flow](#-end-to-end-flow)
- [Project Structure](#-project-structure)
- [Environment & Configuration](#-environment--configuration)
- [Documentation](#-documentation)
- [License](#-license)

---

## 🔍 About the Project

The **TVS EV Subscription Platform** is a fully functional microservices-based backend that powers an electric vehicle subscription service. It enables EV owners to:

- **Register** their vehicle (validated against an RTO reference table) and create an account
- **Browse** subscription plans with bundled features (e.g., charging credits, roadside assistance)
- **Pay** for a plan via Razorpay's payment gateway (order creation + HMAC-SHA256 signature verification)
- **Receive** a PDF invoice via email automatically upon successful payment
- **Track** feature-by-feature usage credits and consume them as they ride
- **Auto-renew** subscriptions and receive expiry notifications via scheduled email jobs

All traffic flows through a **JWT-secured API Gateway** that validates tokens and enforces role-based access before routing to the correct downstream service. **Eureka** provides service discovery across the ecosystem.

---

## 🏗 Architecture

```
                          ┌──────────────────────────────────────────┐
                          │         Netflix Eureka Server             │
                          │           :8761  (Discovery)              │
                          └────────────────┬─────────────────────────┘
                                           │  Service Registry
                     ┌─────────────────────▼──────────────────────────┐
  Client / Mobile ──►│         Spring Cloud API Gateway               │
   (React / App)     │           :8083  (JWT Auth + Routing)          │
                     └──┬──────┬──────┬──────┬──────┬────────────────┘
                        │      │      │      │      │
              ┌─────────▼┐  ┌──▼───┐ ┌▼─────┴┐ ┌───▼──────┐  ┌──────────────┐
              │  User    │  │ Plan │ │Feature│ │  Order   │  │  Payment     │
              │ Service  │  │Svc   │ │  Svc  │ │  Service │  │  Service     │
              │  :9003   │  │:8081 │ │ :9001 │ │  :9004   │  │  :9008       │
              └──────────┘  └──────┘ └───────┘ └────┬─────┘  └──────┬───────┘
                                                     │               │ Razorpay API
                                                     │               │
                               ┌─────────────────────▼───────────────▼──────────┐
                               │              Usage Manager Service              │
                               │    :9007  (Subscriptions + Feature Credits)     │
                               └──────────────────────┬──────────────────────────┘
                                                       │
                               ┌───────────────────────▼──────────────────────────┐
                               │          Email Notification Service               │
                               │   :9090  (JavaMail + Schedulers + Attachments)    │
                               └──────────────────────────────────────────────────┘

                    Each service owns its own MySQL database (DB-per-service pattern)
```

---

## 🧩 Services Overview

| # | Service | Port | Database | Responsibility |
|---|---------|------|----------|----------------|
| 1 | **Eureka Server** | `8761` | — | Service registry & discovery (Netflix Eureka) |
| 2 | **API Gateway** | `8083` | — | JWT validation, role-based routing, request forwarding |
| 3 | **User Service** | `9003` | `userdb` | Registration, login (JWT issuance), OTP password reset, vehicle management |
| 4 | **Plan Service** | `8081` | `project` | Subscription plan CRUD — plans bundle multiple features |
| 5 | **Feature Service** | `9001` | `project` | Feature definitions (metered units, pricing, descriptions) |
| 6 | **Order Service** | `9004` | `userdb` | Plan assignment, payment verification, PDF invoice generation + email |
| 7 | **Payment Service** | `9008` | `paymentdb` | Razorpay order creation, HMAC-SHA256 signature verification, payment logging |
| 8 | **Usage Manager Service** | `9007` | `usagemanagerdb` | User subscriptions, per-feature credit tracking, auto-renew cron job |
| 9 | **Email Notification Service** | `9090` | — | Plain + HTML + attachment emails via Gmail SMTP, plan expiry scheduler |

---

## 🛠 Tech Stack

### Core
| Layer | Technology |
|-------|-----------|
| Language | Java 17 |
| Framework | Spring Boot 3.5.6 |
| Service Mesh | Spring Cloud 2025.0.0 |
| Service Discovery | Netflix Eureka |
| API Gateway | Spring Cloud Gateway (Reactive) |
| ORM | Spring Data JPA + Hibernate |
| Database | MySQL 8.0 (DB-per-service) |
| Security | Spring Security + JWT (jjwt 0.11.5) + BCrypt |
| HTTP Clients | WebClient (reactive) + RestTemplate |

### Integrations & Utilities
| Capability | Technology |
|-----------|-----------|
| Payment Gateway | Razorpay Java SDK |
| PDF Generation | iText 7 (kernel + layout + io) |
| Email | Spring Boot Mail + Gmail SMTP + MimeMessage |
| Validation | Spring Validation (`@Valid`, `@NotNull`, `@Email`, …) |
| Boilerplate reduction | Lombok (`@Data`, `@Builder`, `@AllArgsConstructor`) |
| Scheduling | `@Scheduled` + `@EnableScheduling` cron jobs |
| Build | Apache Maven 3.9 |
| Containerization | Docker (multi-stage builds) |
| CI/CD | GitLab CI (`.gitlab-ci.yml` per service) |

---

## ✨ Key Features

### 🔐 Security & Authentication
- **JWT (HS256)** issued by User Service on login — 24-hour expiry
- **API Gateway** validates the JWT on every request using a custom `GlobalFilter`; no downstream service needs its own security filter
- **BCrypt** password hashing (Spring Security `PasswordEncoder`)
- **OTP-based password reset** — 6-digit OTP emailed via notification service, TTL-validated in DB
- **Role-based access control** — `ROLE_USER` and `ROLE_ADMIN` enforced at gateway route level

### 💳 Payment Flow
- Creates a **Razorpay order** with amount + currency, returns `order_id` to the frontend
- After Razorpay checkout, the frontend sends `razorpay_payment_id` + `razorpay_order_id` + `razorpay_signature`
- Backend verifies the **HMAC-SHA256 signature** (`razorpay_order_id|razorpay_payment_id` signed with `key_secret`) before marking payment `SUCCESS`
- Zero money moves unless signature matches — prevents payment bypass attacks

### 📄 Invoice & Notifications
- On plan assignment, Order Service generates a **PDF invoice** using iText 7 (user name, plan, pricing, date, order ID)
- Invoice is sent as an **email attachment** via Email Notification Service
- `PlanExpiryScheduler` runs daily (`@Scheduled`) to warn users whose plan expires within 3 days

### 📊 Usage Tracking
- Usage Manager maintains a `UserSubscription` and one `PlanUsage` row per (user, feature)
- Each API call to `consumeUnits` decrements the credit balance and writes a `FeatureUsageHistory` audit row
- Auto-renew cron job resets credits and extends `endDate` by 12 months when a subscription expires

### 🚗 Vehicle Validation
- RTO vehicle numbers are pre-seeded in a `vehicle_no` reference table
- Registration rejects any vehicle number not present in this table (domain integrity)
- A user can own multiple `Vehicle` entries (one-to-many relationship)

---

## 🚀 Getting Started

### Prerequisites

| Tool | Version |
|------|---------|
| Java JDK | 17+ |
| Apache Maven | 3.9+ |
| MySQL | 8.0+ |
| Docker (optional) | 20+ |

### Database Setup

Create the required databases in MySQL:

```sql
CREATE DATABASE userdb;
CREATE DATABASE paymentdb;
CREATE DATABASE usagemanagerdb;
CREATE DATABASE project;      -- shared by plan-service & feature-service
```

All tables are auto-created by Hibernate (`ddl-auto=update`) on first startup. You only need to create the databases themselves.

> **Seed RTO data** — manually insert known vehicle numbers into `userdb.vehicle_no` so users can register:
> ```sql
> USE userdb;
> INSERT INTO vehicle_no (vehicle_number) VALUES ('MH12AB1234'), ('KA01XY9999'), ('TN07CD5678');
> ```

### Running Locally

Start services **in this exact order** (each depends on the previous being available):

```bash
# 1. Eureka Server — service registry (must be first)
cd tvs-eureka-server-service-main
./mvnw spring-boot:run

# 2. API Gateway — all client traffic enters here
cd ../tvs-api-gateway-service-main
./mvnw spring-boot:run

# 3. User Service — auth, registration, vehicles
cd ../tvs-user-microservice-main
./mvnw spring-boot:run

# 4. Plan Service — subscription plans catalog
cd ../plan-service
./mvnw spring-boot:run

# 5. Feature Service — feature definitions
cd ../tvs-feature-main
./mvnw spring-boot:run

# 6. Payment Service — Razorpay integration
cd ../tvs-payment-service-main
./mvnw spring-boot:run

# 7. Order Service — plan assignment, PDF invoice
cd ../tvs-order-service-main
./mvnw spring-boot:run

# 8. Usage Manager Service — credits, auto-renew
cd ../tvs-usagemanager-service-main/tvs-usagemanager-service-main
./mvnw spring-boot:run

# 9. Email Notification Service — SMTP, schedulers
cd ../../../email-notification-service
./mvnw spring-boot:run
```

Verify all services are registered at: **http://localhost:8761**

### Running with Docker

Each service includes a **multi-stage Dockerfile** (Maven build stage → JRE runtime stage):

```bash
# Build and run a single service (example: User Service)
cd tvs-user-microservice-main
docker build -t tvs-user-service:latest .
docker run -p 9003:9003 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://host.docker.internal:3306/userdb \
  -e SPRING_DATASOURCE_USERNAME=root \
  -e SPRING_DATASOURCE_PASSWORD=Pass@123 \
  tvs-user-service:latest
```

> **Note:** A root-level `docker-compose.yml` would orchestrate all services together. See [`docs/03-run-and-commands.md`](docs/03-run-and-commands.md) for the full compose setup guide.

---

## 📡 API Overview

All requests go through the **API Gateway at `:8083`**. Include `Authorization: Bearer <token>` for protected endpoints.

### Authentication (User Service)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/auth/register` | Public | Register new user (vehicle validation) |
| `POST` | `/api/auth/login` | Public | Login → returns JWT |
| `POST` | `/api/auth/send-otp` | Public | Send OTP to email for password reset |
| `POST` | `/api/auth/verify-otp` | Public | Verify OTP |
| `POST` | `/api/auth/reset-password` | Public | Reset password with valid OTP |

### User Management
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/api/users/{id}` | `ROLE_USER` | Get user profile |
| `PUT` | `/api/users/{id}` | `ROLE_USER` | Update profile |
| `GET` | `/api/users/{id}/vehicles` | `ROLE_USER` | List user vehicles |
| `POST` | `/api/users/{id}/vehicles` | `ROLE_USER` | Add vehicle |

### Plans & Features
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/api/plans` | Public | List all plans |
| `GET` | `/api/plans/{id}` | Public | Plan details |
| `POST` | `/api/plans` | `ROLE_ADMIN` | Create plan |
| `GET` | `/api/features` | Public | List all features |
| `POST` | `/api/features` | `ROLE_ADMIN` | Create feature |

### Payments
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/payments/create` | `ROLE_USER` | Create Razorpay order |
| `POST` | `/api/payments/verify` | `ROLE_USER` | Verify signature → mark SUCCESS |
| `GET` | `/api/payments/status/{orderId}` | `ROLE_USER` | Get payment status |

### Orders
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/orders/{userId}/assign/{planId}` | `ROLE_USER` | Assign plan (payment verified), send PDF invoice |
| `GET` | `/api/orders/{userId}` | `ROLE_USER` | Get user's orders |

### Usage Manager
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/subscriptions/create` | `ROLE_USER` | Initialize subscription + credits |
| `POST` | `/api/subscriptions/consume` | `ROLE_USER` | Consume feature units |
| `GET` | `/api/subscriptions/usage/{userId}` | `ROLE_USER` | Get current credits |
| `GET` | `/api/subscriptions/history/{userId}` | `ROLE_USER` | Usage history |

### Notifications
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/notifications/send-email` | Internal | Send plain email |
| `POST` | `/api/notifications/send-email-with-attachment` | Internal | Send email + PDF |

> Full endpoint reference with request/response payloads: [`docs/41-api-reference.md`](docs/41-api-reference.md)

---

## 🔄 End-to-End Flow

Here is the complete **paid plan subscription journey**:

```
User                  Gateway            User-Svc     Payment-Svc      Order-Svc     Email-Svc
 │                       │                  │              │                │              │
 │─── POST /auth/login ──►│                  │              │                │              │
 │                        │──► validates ────►│              │                │              │
 │                        │◄── JWT token ─────│              │                │              │
 │◄── {token} ────────────│                  │              │                │              │
 │                        │                  │              │                │              │
 │─── POST /payments/create (Bearer token) ──►│              │                │              │
 │                        │──── JWT valid? ───►              │                │              │
 │                        │──────────────────────────────────►│               │              │
 │                        │                  │           Razorpay API         │              │
 │◄── {razorpay_order_id} ─────────────────────────────────────               │              │
 │                        │                  │              │                │              │
 │  [User completes Razorpay checkout in browser]           │                │              │
 │                        │                  │              │                │              │
 │─── POST /payments/verify ──────────────────────────────────►│              │              │
 │                        │                  │           HMAC verify          │              │
 │◄── {status: SUCCESS} ───────────────────────────────────────               │              │
 │                        │                  │              │                │              │
 │─── POST /orders/{userId}/assign/{planId} ──►│              │                │              │
 │                        │──────────────────────────────────────────────────►│              │
 │                        │                  │              │         check payment status   │
 │                        │                  │              │◄───────────────│              │
 │                        │                  │         {SUCCESS}              │              │
 │                        │                  │────UserPlanOrder persisted──────              │
 │                        │                  │────PDF invoice generated (iText)──────────────►│
 │                        │                  │                                │  send email  │
 │◄── {orderId, planName, PDF sent} ──────────────────────────────────────────│              │
```

> 9 more detailed flow diagrams (OTP reset, usage consumption, auto-renew, etc.) in [`docs/04-end-to-end-flows.md`](docs/04-end-to-end-flows.md)

---

## 📁 Project Structure

```
EV-SUBSCRIPTION/
├── tvs-eureka-server-service-main/     # Service discovery (port 8761)
├── tvs-api-gateway-service-main/       # JWT gateway (port 8083)
├── tvs-user-microservice-main/         # Users, auth, OTP, vehicles (port 9003)
├── plan-service/                       # Subscription plans (port 8081)
├── tvs-feature-main/                   # Feature definitions (port 9001)
├── tvs-order-service-main/             # Orders, PDF invoice (port 9004)
├── tvs-payment-service-main/           # Razorpay integration (port 9008)
├── tvs-usagemanager-service-main/      # Usage credits, auto-renew (port 9007)
│   └── tvs-usagemanager-service-main/  # (nested — actual source)
├── email-notification-service/         # Email + schedulers (port 9090)
└── docs/                               # 📚 Full study documentation
    ├── README.md                       # Docs index
    ├── 00-overview.md                  # Business & actors
    ├── 01-architecture.md              # Microservice topology
    ├── 02-tech-stack.md                # Libraries & versions
    ├── 03-run-and-commands.md          # All run commands
    ├── 04-end-to-end-flows.md          # Sequence diagrams
    ├── 40-db-schema.md                 # All DB tables
    ├── 41-api-reference.md             # API catalog
    ├── 50-known-gaps-and-improvements.md
    ├── 99-interview-qa.md              # 60+ Q&A
    ├── services/                       # Per-service deep dives (9 files)
    └── concepts/                       # Spring/Java concept explainers (15 files)
```

Each service follows the **same internal structure**:
```
<service>/
├── Dockerfile
├── pom.xml
└── src/main/java/com/tvs/
    ├── <Service>Application.java      # @SpringBootApplication entry point
    ├── controller/                    # @RestController — HTTP layer
    ├── service/                       # Business logic
    ├── repository/                    # Spring Data JPA interfaces
    ├── entity/                        # @Entity JPA models
    ├── dto/                           # Request/Response POJOs (Lombok)
    ├── config/                        # Security, WebClient, CORS beans
    └── exception/                     # @ControllerAdvice + custom exceptions
```

---

## ⚙ Environment & Configuration

All configuration lives in `src/main/resources/application.properties` per service. Key settings:

| Service | Key Property | Default |
|---------|-------------|---------|
| All | `spring.jpa.hibernate.ddl-auto` | `update` |
| User Service | `jwt.secret` | `MySuperSecureJwtSecretKey...` |
| User Service | `jwt.expiration` | `86400000` (24 h) |
| API Gateway | `jwt.secret` | *(must match User Service)* |
| Payment Service | `razorpay.key.id` | `rzp_test_...` |
| Payment Service | `razorpay.key.secret` | `Qudx...` |
| All Email-sending | `spring.mail.username` | `projectemailod@gmail.com` |
| Eureka clients | `eureka.client.service-url.defaultZone` | `http://localhost:8761/eureka/` |

> ⚠️ **For production:** externalize secrets via environment variables or a secrets manager (Vault / AWS Secrets Manager). Never commit real credentials.

---



---

## 📄 License

This project is built for learning and demonstration purposes. All rights reserved © Harsh Rawat.

---

<div align="center">

**Built with ❤️ using Spring Boot · Java 17 · MySQL · Razorpay · iText · Docker**

</div>

