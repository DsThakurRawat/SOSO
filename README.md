# SOSO

A high-performance, production-ready, production-grade social media REST API backend built in Go.

Developed and maintained by Divyansh Rawat (https://github.com/DsThakurRawat).

---

## Tech Stack and Integrations

* Language: Go (Golang 1.22+)
* Database: PostgreSQL (v16.3) for relational data persistence
* Caching: Redis (v6.2) for read-scaling (e.g., user profiles, feeds)
* HTTP Routing: Chi Router (go-chi/chi/v5)
* Logging: Structured, high-performance logging with Zap Logger (uber-go/zap)
* Request Validation: Go Playground Validator (go-playground/validator/v10)
* Authentication: JWT authentication (golang-jwt/jwt/v5) and password hashing via BCrypt
* API Documentation: OpenAPI / Swagger UI (swaggo/swag)
* Email Delivery: SendGrid (Production) and Mailtrap (Development/Testing)
* Dev and Hot Reloading: Air (watches files and rebuilds instantly)
* Database Migrations: Golang-Migrate CLI

---

## Scalability and Capacity (1 Lakh / 100k DAU)

With the current out-of-the-box configuration, this application is theoretically capable of serving 1 Lakh (100,000) Daily Active Users (DAU).

### Performance Breakdown:
* Throughput: 100k DAU performing normal activity translates to an average of ~115 Requests Per Second (RPS).
* Database Connection Pool: Configured with a maxOpenConns limit of 30. Assuming simple query operations take ~10ms, the DB pool alone can sustain up to 3,000 queries per second, leaving a massive head-room for peak active hours.
* Caching: Heavy read endpoints (like fetching user profiles) bypass PostgreSQL and query Redis directly. Since Redis operates in-memory and can handle 100,000+ operations/sec, read latency remains sub-millisecond.
* Rate Limiting: Users are restricted to 20 requests every 5 seconds (average of 4 requests/sec per IP) by default, protecting the system from brute-force login requests or DDoS attempts.

---

## Architecture Layout

```text
SOSO/
├── cmd/
│   ├── api/          # REST API server entrypoint (routing, HTTP server setup, controllers)
│   └── migrate/      # CLI tool for database migrations (schema setup and versioning)
├── internal/         # Core application logic (encapsulated)
│   ├── auth/         # JWT generation, validation, and password hashing (BCrypt)
│   ├── db/           # PostgreSQL connection pool configuration
│   ├── env/          # Environment configuration loader
│   ├── mailer/       # Email delivery integration (SendGrid and Mailtrap)
│   ├── ratelimiter/  # API rate-limiting middleware (prevents DDoS and brute-force abuse)
│   └── store/        # Data Layer: Implements the Repository Pattern for PostgreSQL and Redis
│       └── cache/    # Redis caching (caches heavy read operations like User profiles)
├── docs/             # Swagger / OpenAPI documentation files
└── web/              # Minimal Next.js frontend scaffold (main focus is the Go backend)
```

---

## Getting Started

### 1. Prerequisites
Make sure you have the following installed:
* Go (v1.22+)
* Docker and Docker Compose

### 2. Start Infrastructural Services
Spin up PostgreSQL, Redis, and Redis Commander containers:
```bash
docker-compose up -d
```
* PostgreSQL runs on port 5432
* Redis runs on port 6379
* Redis Commander (Web GUI for Redis) runs on http://localhost:8081

### 3. Run Database Migrations
To set up your database schema, apply migrations:
```bash
make migrate-up
```

### 4. Start the Application
For development with hot-reloading:
```bash
air
```
Alternatively, run without hot-reload:
```bash
go run cmd/api/main.go
```

---

## Testing

To test the built-in rate limiter under load, use autocannon:
```bash
npx autocannon -r 22 -d 1 -c 1 --renderStatusCodes http://localhost:8080/v1/health
```