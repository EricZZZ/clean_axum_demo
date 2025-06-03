# **Rust Axum Clean Demo**

A **modern, clean-architecture Rust API server template** built with Axum and SQLx. It features domain-driven design, repository patterns, JWT authentication, file uploads, Swagger documentation, and comprehensive testing.

## ✨ Features

- Clean Architecture & layered domain separation
- Modular Axum HTTP server
- SQLx with compile-time query checks
- JWT authentication & protected routes
- File upload and secure asset serving
- Swagger UI documentation (utoipa)
- OpenTelemetry distributed tracing and metrics instrumentation

## 🛠 Getting Started

### Prerequisites

- Rust (latest stable)
- MySQL or MariaDB
- Docker & Docker Compose (optional)

### Quickstart

Choose your preferred setup:

- **Using Docker Compose:**

  ```bash
  docker-compose up --build
  ```

  To stop and clean up:

  ```bash
  docker-compose down --rmi all
  ```

- **Manual Setup:**
  1. **Create database tables:**  
     From `db-seed`:
     ```bash
     mysql -u <user> -p <database> < db-seed/01-tables.sql
     mysql -u <user> -p <database> < db-seed/02-seed.sql
     ```
  2. **Configure environment:**  
     Edit `.env`:
     ```env
     DATABASE_URL=mysql://user:password@localhost/clean_axum_demo
     JWT_SECRET_KEY=your_super_secret_key
     SERVICE_PORT=8080
     OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
     OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
     ```
  3. **Prepare SQLx (offline mode with validation):**
     ```bash
     cargo sqlx prepare --check
     ```
  4. **Run locally:**
     ```bash
     cargo run
     ```

### Usage

- **Authenticate & call protected API:**
  1. Login:
     ```bash
     curl -X POST http://localhost:8080/auth/login \
       -H "Content-Type: application/json" \
       -d '{"client_id":"apitest01","client_secret":"test_password"}'
     ```
  2. Use the returned `token`:
     ```bash
     curl http://localhost:8080/user -H "Authorization: Bearer $token"
     ```
- **API docs:**  
  Open [http://localhost:8080/docs](http://localhost:8080/docs) in your browser for the Swagger UI.

  Access protected endpoints:

  - Authenticate by sending a `POST` request to `/auth/login` (e.g., via Swagger UI or curl).

    ```json
    {
      "client_id": "apitest01",
      "client_secret": "test_password"
    }
    ```

    - Copy the returned JWT token.

  - In Swagger UI, click the 🔒 Authorize button and paste `<jwt>` to authorize requests.

### 📦 Project Structure

Recommended layout:

```plaintext
├── src

│   ├── <domain>/             # Replace with: auth, user, device, file, etc.
│   │   ├── mod.rs            # Module entry point
│   │   ├── domain/           # Domain logic: models, traits
│   │   │   ├── mod.rs
│   │   │   ├── model.rs
│   │   │   ├── repository.rs
│   │   │   └── service.rs
│   │   ├── handlers.rs       # HTTP handlers
│   │   ├── routes.rs         # Route definitions
│   │   ├── queries.rs        # SQLx query logic
│   │   ├── dto.rs            # Data Transfer Objects
│   │   └── services.rs        # Infrastructure-layer service implementations

│   ├── common/               # Shared components/utilities
│   │   ├── mod.rs
│   │   ├── app_state.rs          # Defines AppState struct for dependency injection
│   │   ├── bootstrap.rs          # Initializes services and constructs AppState
│   │   ├── config.rs             # Loads configuration from environment variables
│   │   ├── dto.rs                # Shared/global DTOs used across domains
│   │   ├── error.rs              # Defines AppError enum and error mappers
│   │   ├── hash_util.rs          # Hashing utilities (e.g., bcrypt)
│   │   ├── jwt.rs                # JWT encoding/decoding and validation
│   │   ├── opentelemetry.rs      # OpenTelemetry setup
│   │   └── ts_format.rs          # Custom timestamp formatting for serialization
│   ├── lib.rs               # Declares top-level modules like app, auth, user, etc.
│   ├── app.rs               # Axum router and middleware setup
│   ├── main.rs              # Application entry point
│   ├── .env                 # Environment variables for local development
│   ├── .env.test            # Environment overrides for test environment (e.g., test DB)
└── tests/                     # Integration and API tests
    ├── asset/                 # Test file assets
    │   ├── cat.png
    │   └── mario_PNG52.png
    ├── test_auth_routes.rs
    ├── test_device_routes.rs
    ├── test_helpers.rs       # Shared setup and utility functions for integration tests
    └── test_user_routes.rs
```

> When adding a new domain module, be sure to register it in the following files:
>
> - `src/lib.rs`
> - `src/app.rs`
> - `src/common/app_state.rs`
> - `src/common/bootstrap.rs`

Domain modules (`domain`, `dto`, `handlers`, `routes`, `queries`, `services`, etc.) can be automatically generated using [domain_codegen](https://github.com/sukjaelee/domain_codegen).

### 📦 API Response Format

All endpoints return a consistent JSON envelope:

```json
{
  "status": 200,
  "message": "success",
  "data": { ... }
}
```

These are implemented as:

- `ApiResponse<T>` – standard generic response wrapper used in most endpoints
- `RestApiResponse<T>` – wrapper around `ApiResponse<T>` for Axum's `IntoResponse` trait

See their definitions in `common/dto.rs`.

### 🧪 Environment Configuration

Configure via `.env` at the project root.  
Set DB URL, JWT secret, service port, and asset settings.  
Example:

```env
DATABASE_URL=mysql://user:pass@localhost/test_db
```

### 🧠 Domain-Driven Design

- Domain models are plain Rust structs/enums
- Business logic resides in domain services or model methods
- Core logic is free from framework or DB coupling

### 🔄 Use Case Isolation & Dependency Inversion

- Services/use cases encapsulate operations
- Repository traits injected via `Arc<T>`
- Infrastructure implements traits for easy mocking and testing

### 🔌 Infrastructure Layer

- SQLx for DB access with compile-time checked queries
- UUIDs stored as `CHAR(36)` (MySQL/MariaDB)
- Queries reside in each domain's `db/` or `repository.rs`

### 🧭 QueryBuilder vs. Static Queries

Use `QueryBuilder` for dynamic or batch SQL.  
Prefer `sqlx::query!` macros for static, type-checked queries.

### 🧱 Database Schema

See `db-seed/` for table definitions and sample data.  
Below is the Entity Relationship Diagram (ERD) illustrating the database structure:

![ER Diagram](./ERD.png)

### 🌐 Interface Layer (Axum)

- Route handlers accept DTOs, invoke domain logic, and return serialized responses
- Each domain owns its `routes.rs` and `handlers.rs`
- **File upload:** Endpoints support asynchronous multipart upload and validation
- **Protected file serving:** Secure endpoints validate user permissions and sanitize file paths

### 🧾 DTOs & Validation

- Request/response DTOs live in each domain's `dto.rs`
- Explicit mapping between DTOs and domain models
- Use `serde` and optionally the [`validator`](https://docs.rs/validator) crate for input validation

### 📚 API Documentation

- Swagger UI available at `/docs` (utoipa)
- DTOs and endpoints annotated for OpenAPI

### ✅ Testing

- Unit tests cover domain logic and core components
- Integration tests exercise HTTP endpoints and database interactions
- Use `#[tokio::test]` and `tower::ServiceExt` for HTTP simulation

### 🚨 Error Handling

- Centralized `AppError` enum implements `IntoResponse`
- Errors mapped to appropriate HTTP status codes and JSON structure, e.g.:

```json
{
  "status": 400,
  "message": "Invalid request data",
  "data": null
}
```

### OpenTelemetry Feature

This project supports distributed tracing, logging, and metrics via OpenTelemetry. To enable and use this feature, follow these steps:

1. Run Jaeger (OTLP-compatible collector):

   ```bash
   docker run --rm --name jaeger \
     -e COLLECTOR_OTLP_ENABLED=true \
     -p 16686:16686 \
     -p 4317:4317 \
     -p 4318:4318 \
     -p 5778:5778 \
     -p 9411:9411 \
     jaegertracing/jaeger:2.6.0
   ```

   Access the Jaeger UI at [http://localhost:16686](http://localhost:16686).

2. Enable OpenTelemetry when running or building the service:

   - To run with OpenTelemetry:
     ```bash
     cargo run --features opentelemetry
     ```
   - To build with OpenTelemetry:
     ```bash
     cargo build --features opentelemetry
     ```

For more information, see the [Jaeger Getting Started guide](https://www.jaegertracing.io/docs/2.6/getting-started/).

## 🚧 Roadmap & Future Enhancements

- **Hexagonal architecture:** Separate into domain, infra, app, and web crates for improved decoupling and testability
- **PostgreSQL migration:** Native UUID support, advanced types, and enhanced scalability
- **gRPC support:** Enable machine-to-machine APIs
- **RBAC:** Role-based access controls
- **Expanded documentation:** Complete schema and deeper infrastructure insights

## 🤝 Contributing

Contributions are welcome! Feel free to open issues, suggest improvements, or submit pull requests.

See the roadmap above for ideas or create your own. Let's build something great together 🚀

## 📄 License

MIT License. See [LICENSE](./LICENSE) for details.

## 🔗 Useful Links

- [Axum](https://docs.rs/axum)
- [SQLx](https://docs.rs/sqlx)
- [Utoipa (OpenAPI)](https://docs.rs/utoipa)
- [Tokio](https://tokio.rs/)
- [Validator (crate)](https://docs.rs/validator)
