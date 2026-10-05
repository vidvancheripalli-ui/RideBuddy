# RideBuddy V1 — System Architecture

**Version:** 0.1
**Status:** Approved
**Architecture:** Modular Monolith

---

## 1. Architecture Overview

RideBuddy will use a modular monolithic backend with a separate frontend application.

```text
                         ┌──────────────────┐
                         │      Users       │
                         └────────┬─────────┘
                                  │
                                HTTPS
                                  │
                         ┌────────▼─────────┐
                         │     Next.js      │
                         │    Frontend      │
                         │     Vercel       │
                         └────────┬─────────┘
                                  │
                         REST / WebSocket
                                  │
                         ┌────────▼─────────┐
                         │     FastAPI      │
                         │     Backend      │
                         │    Cloud Run     │
                         └────────┬─────────┘
                                  │
                         ┌────────▼─────────┐
                         │    Supabase      │
                         │ PostgreSQL +     │
                         │    PostGIS       │
                         └──────────────────┘
```

---

## 2. Architectural Philosophy

RideBuddy will use a **modular monolith** rather than a microservices architecture.

The backend will consist of logically separated modules within a single FastAPI application.

```text
backend/
└── app/
    ├── auth/
    ├── users/
    ├── colleges/
    ├── vehicles/
    ├── rides/
    ├── matching/
    ├── notifications/
    ├── tracking/
    └── ratings/
```

This provides clear domain boundaries without introducing the operational complexity of multiple independently deployed services.

Individual modules may be extracted into independent services in the future if actual scalability or organizational requirements justify it.

---

## 3. Frontend

### Technology

* Next.js
* TypeScript
* React
* Tailwind CSS
* shadcn/ui

### Hosting

Vercel.

### Responsibilities

The frontend is responsible for:

* User interface
* Client-side navigation
* Form handling
* Displaying ride information
* Map visualization
* Ride request interactions
* Driver/rider dashboards
* Realtime UI updates
* Client-side state management

The frontend must not directly access the production database.

All business operations must go through the backend API.

---

## 4. Backend

### Technology

* Python
* FastAPI

### Hosting

Google Cloud Run.

### Responsibilities

The backend is responsible for:

* Authentication
* Authorization
* User management
* College verification
* Vehicle verification
* Ride management
* Ride requests
* Ride lifecycle
* Geospatial matching
* Location privacy
* Realtime communication
* Notifications
* Ratings
* Validation
* Business rules
* Database access

The backend is the authoritative layer for all business rules.

The frontend must never be trusted to enforce security-sensitive rules.

---

## 5. Backend Module Structure

The initial backend will be organized by domain.

```text
app/
│
├── auth/
│
├── users/
│
├── colleges/
│
├── vehicles/
│
├── rides/
│
├── matching/
│
├── tracking/
│
├── notifications/
│
└── ratings/
```

Each module should contain its own:

* API routes
* schemas
* service/business logic
* database operations where appropriate
* tests

Cross-module dependencies should be kept explicit and minimal.

---

## 6. API Architecture

Normal application operations will use REST APIs.

Example:

```text
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
POST   /api/v1/auth/logout

GET    /api/v1/users/me

GET    /api/v1/colleges

POST   /api/v1/vehicles
GET    /api/v1/vehicles
POST   /api/v1/vehicles/{vehicle_id}/verify

POST   /api/v1/rides
GET    /api/v1/rides/{ride_id}
GET    /api/v1/rides/search

POST   /api/v1/rides/{ride_id}/requests

POST   /api/v1/requests/{request_id}/accept
POST   /api/v1/requests/{request_id}/reject
POST   /api/v1/requests/{request_id}/cancel
```

The exact API contract will be defined after the domain model is designed.

All APIs will be versioned under:

```text
/api/v1/
```

---

## 7. Realtime Architecture

WebSockets will be used only for functionality that genuinely requires realtime communication.

Initial use cases:

* Driver live location
* Ride status changes
* Realtime notifications where required

Conceptual flow:

```text
Driver
   │
   │ GPS update
   ▼
WebSocket
   │
   ▼
FastAPI
   │
   │ Authorization
   ▼
Connected rider(s)
```

The backend must verify that a user is authorized to receive location information before broadcasting it.

---

## 8. Database

RideBuddy will use:

**PostgreSQL + PostGIS**

hosted through Supabase.

PostgreSQL will provide:

* Relational data model
* Transactions
* Constraints
* Referential integrity
* Indexing
* Reliable persistence

PostGIS will provide:

* Geographic points
* Geographic distances
* Spatial filtering
* Spatial indexes
* Route geometry
* Future route-overlap calculations

---

## 9. Geospatial Data

Locations will be represented using appropriate geographic types rather than only text addresses.

Conceptually:

```text
POINT(latitude, longitude)
```

Routes may later be represented using:

```text
LINESTRING(...)
```

Spatial queries will be performed through PostGIS.

Examples of future operations:

```text
Find rides near pickup point
Calculate distance between locations
Find candidate routes
Calculate route overlap
Estimate pickup/destination detour
```

The exact geospatial schema will be defined during database design.

---

## 10. Location Privacy

RideBuddy will distinguish between private and user-visible location information.

### Exact location

May be used internally when required.

Example:

```text
Driver's current GPS position
```

### Public/participant pickup location

A designated pickup point or sufficiently coarse location.

Example:

```text
Miyapur Metro Station
```

### Home locality

A coarse geographic area.

Example:

```text
Kukatpally
```

Exact residential addresses must not be exposed to other RideBuddy users.

Live driver location is available only to authorized participants of the corresponding active ride.

---

## 11. Authentication

Authentication will be handled by the FastAPI backend.

Conceptual flow:

```text
User
 │
 ▼
Next.js
 │
 │ Login request
 ▼
FastAPI
 │
 │ Verify credentials
 ▼
Authentication
 │
 ▼
Access + Refresh credentials
```

The exact token storage mechanism will be finalized during authentication design.

The preferred approach is to use secure, HttpOnly cookies where appropriate to minimize exposure of authentication credentials to client-side JavaScript.

---

## 12. Authorization

Authentication answers:

> Who is this user?

Authorization answers:

> What is this user allowed to do?

Examples:

```text
Unverified student
    ↓
Can search rides

College-verified student
    ↓
Can access additional verified features

College + vehicle verified
    ↓
Can post rides

Ride participant
    ↓
Can access active ride tracking

Ride owner
    ↓
Can manage their posted ride
```

Authorization rules will be enforced by FastAPI rather than relying on frontend restrictions.

---

## 13. Ride and Request Separation

A ride and a request to join that ride are separate domain concepts.

Example:

```text
User
 │
 └── creates ──→ Ride
                    │
                    ├── RideRequest ← User
                    ├── RideRequest ← User
                    └── RideRequest ← User
```

A ride represents the driver's planned journey.

A ride request represents a student's request to join that journey.

This separation allows the system to support:

* Multiple riders
* Individual request states
* Driver approval/rejection
* Seat limits
* Cancellation
* Request history

---

## 14. Ride Lifecycle

The initial conceptual lifecycle is:

```text
POSTED
   ↓
REQUESTED
   ↓
ACCEPTED
   ↓
ACTIVE
   ↓
COMPLETED
```

Possible alternative transitions include:

```text
REQUESTED → REJECTED
REQUESTED → CANCELLED
ACCEPTED  → CANCELLED
```

The final state machine and transition constraints will be formally defined during domain design.

---

## 15. Deployment Architecture

```text
                         GitHub
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
             Vercel              Google Cloud
                │                  Cloud Run
                ▼                     │
             Next.js              FastAPI
                                      │
                                      ▼
                                  Supabase
                               PostgreSQL +
                                  PostGIS
```

Initial deployments may be performed manually.

Automated CI/CD will be introduced after the application has a stable build and test process.

---

## 16. Environment Management

Environment-specific configuration must not be committed to GitHub.

Examples include:

```text
DATABASE_URL
JWT_SECRET
MAP_API_KEY
SUPABASE credentials
Cloud credentials
```

Local development will use environment files that are excluded from Git.

A safe template will be maintained:

```text
.env.example
```

containing variable names but not secret values.

---

## 17. Initial Infrastructure

The initial system will intentionally avoid unnecessary infrastructure.

### Included

```text
Next.js
FastAPI
PostgreSQL
PostGIS
Vercel
Cloud Run
Supabase
```

### Not initially required

```text
Redis
Kafka
Celery
Kubernetes
Elasticsearch
Microservices
Dedicated notification service
Dedicated message broker
```

These technologies may be introduced when a concrete requirement or performance bottleneck justifies them.

---

## 18. Architectural Evolution

The system should be designed so individual components can evolve without requiring a complete rewrite.

Potential future evolution:

```text
                 Current
                   │
          Modular Monolith
                   │
                   ▼
          Identify bottleneck
                   │
                   ▼
         Extract specific module
                   │
                   ▼
            Independent service
```

Microservices should therefore be treated as an optimization for a demonstrated need, not as a starting requirement.

---

## 19. Architecture Decision Summary

| Area             | Decision             |
| ---------------- | -------------------- |
| Architecture     | Modular Monolith     |
| Frontend         | Next.js + TypeScript |
| Frontend Hosting | Vercel               |
| Backend          | Python + FastAPI     |
| Backend Hosting  | Google Cloud Run     |
| API              | REST                 |
| Realtime         | WebSockets           |
| Database         | PostgreSQL           |
| Geospatial       | PostGIS              |
| Database Hosting | Supabase             |
| Authentication   | Backend-managed      |
| Authorization    | Backend-enforced     |
| Cache            | Not initially        |
| Queue            | Not initially        |
| Microservices    | Not initially        |

---

## 20. Guiding Principle

RideBuddy will follow:

> **Production-oriented architecture with controlled scope.**

The system will prioritize:

* Correctness
* Security
* Maintainability
* Testability
* Observability
* Clear domain boundaries
* Scalability where justified

while avoiding premature infrastructure complexity.
