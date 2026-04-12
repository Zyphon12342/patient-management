# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

All services use Maven with Java 21. Run from within each service directory:

```bash
./mvnw clean package          # Build and package JAR
./mvnw clean package -DskipTests  # Build without running tests
./mvnw test                   # Run unit tests
./mvnw test -Dtest=ClassName  # Run a single test class
```

Integration tests (in `integration-tests/`):
```bash
./mvnw test                   # Requires all services running on their configured ports
./mvnw test -Dtest=AuthIntegrationTest  # Run a single integration test class
```

## Infrastructure Setup (Docker)

Services depend on PostgreSQL and Kafka. No docker-compose.yml exists — start infrastructure manually using Docker with a shared `internal` network:

```bash
docker network create internal

# Auth DB (port 5001)
docker run -d --name auth-db --network internal -p 5001:5432 \
  -e POSTGRES_USER=admin_user -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=auth_service_db postgres:latest

# Patient DB (port 5000)
docker run -d --name patient-db --network internal -p 5000:5432 \
  -e POSTGRES_USER=admin_user -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=patient_service_db postgres:latest

# Kafka in KRaft mode (port 9094 external, 9092 internal)
docker run -d --name kafka --network internal -p 9094:9094 apache/kafka:latest
```

IntelliJ run configurations are in `dockerFileConfig/` for all services.

## Architecture Overview

Five Spring Boot 3.4.x microservices communicating via REST, gRPC, and Kafka:

```
Client → API Gateway (4004) → Auth Service (4005)
                             → Patient Service (4000) → Billing Service (gRPC 9001)
                                                      → Kafka → Analytics Service (4002)
```

**API Gateway** (`api-gateway/`): Spring Cloud Gateway. Routes `/auth/**` to auth-service and `/api/patients/**` to patient-service. Applies JWT validation filter on patient routes. Also proxies OpenAPI docs from each service.

**Auth Service** (`auth-service/`): Issues and validates JWTs (JJWT 0.12.6). Has its own PostgreSQL DB. The JWT secret is injected via `JWT_SECRET` env var (base64-encoded 256-bit key).

**Patient Service** (`patient-service/`): Main CRUD service. Acts as gRPC *client* to billing-service and Kafka *producer* (topic: `patient`, serializes `PatientEvent` protobuf). Has its own PostgreSQL DB.

**Billing Service** (`billing-service/`): Pure gRPC server (port 9001). Exposes `BillingService.CreateBillingAccount` RPC. Also has a REST port (4001) for health/docs.

**Analytics Service** (`analytics-service/`): Kafka *consumer* (group: `analytics-service`, topic: `patient`). Deserializes `PatientEvent` protobuf events.

## Protocol Buffers

Proto files live in `src/main/proto/` of each relevant service. Two contracts:

- `billing_service.proto` — shared between patient-service (client) and billing-service (server)
- `patient_event.proto` — shared between patient-service (producer) and analytics-service (consumer)

Compiled by `protobuf-maven-plugin` v0.6.1 during `mvn generate-sources`. Protoc 3.25.5, gRPC Java 1.68.1.

## Key Environment Variables

| Service | Variable | Purpose |
|---------|----------|---------|
| auth-service | `JWT_SECRET` | Base64 JWT signing key |
| auth-service | `SPRING_DATASOURCE_URL/USERNAME/PASSWORD` | PostgreSQL connection |
| patient-service | `BILLING_SERVICE_ADDRESS`, `BILLING_SERVICE_GRPC_PORT` | gRPC target |
| patient-service | `SPRING_KAFKA_BOOTSTRAP_SERVERS` | Kafka (default: localhost:9092) |
| patient-service | `SPRING_DATASOURCE_URL/USERNAME/PASSWORD` | PostgreSQL connection |
| analytics-service | `SPRING_KAFKA_BOOTSTRAP_SERVERS` | Kafka |
| api-gateway | `AUTH_SERVICE_URL` | Auth service base URL |

## API Documentation

Swagger UI available at `http://localhost:<port>/swagger-ui.html` per service. The gateway aggregates docs at:
- `/api-docs/patients` → patient-service OpenAPI spec
- `/api-docs/auth` → auth-service OpenAPI spec

HTTP request examples are in `api-requests/` and gRPC examples in `grpc-requests/`.

## Integration Tests

Tests in `integration-tests/` use REST-Assured against live services. They require the full stack running. Bearer tokens for authenticated requests must be obtained from auth-service first — see existing tests for the pattern (`AuthIntegrationTest` → `PatientIntegrationTest`).
