# Microservices Architecture Project

A Spring Boot microservices implementation demonstrating service-to-service communication using gRPC and REST APIs.

## Architecture Overview

This project consists of two microservices that communicate using both REST and gRPC protocols:

- **Patient Service** - Primary service exposing REST API endpoints
- **Billing Service** - Secondary service exposing gRPC endpoints

```
┌─────────────────────┐         gRPC          ┌─────────────────────┐
│  Patient Service    │ ───────────────────> │  Billing Service    │
│  (REST API)         │      Port 9001        │  (gRPC Server)      │
│  Port 4000          │                       │  Port 4001          │
└─────────────────────┘                       └─────────────────────┘
         │
         │ REST API
         ▼
    Client/HTTP
```

## Technology Stack

### Backend Framework
- **Spring Boot 3.4.x** - Main application framework
- **Java 21** - Programming language
- **Maven** - Dependency management and build tool

### Communication Protocols
- **REST API** - HTTP-based API for client communication
- **gRPC** - High-performance RPC framework for inter-service communication
- **Protocol Buffers (Proto3)** - Interface definition and serialization

### Data Persistence
- **Spring Data JPA** - Data access abstraction
- **PostgreSQL** - Production database (configurable)
- **H2 Database** - In-memory database for testing (optional)

### Validation & Documentation
- **Jakarta Validation** - Request validation with custom validation groups
- **SpringDoc OpenAPI 3** (Swagger) - API documentation and testing UI

### Infrastructure
- **Docker** - Containerization with multi-stage builds
- **Maven Wrapper** - Ensures consistent Maven version across environments

## Project Structure

```
.
├── patient-service/          # Primary REST API service
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/pm/patientservice/
│   │   │   │   ├── controller/      # REST controllers
│   │   │   │   ├── service/         # Business logic
│   │   │   │   ├── repository/      # Data access layer
│   │   │   │   ├── model/           # JPA entities
│   │   │   │   ├── dto/             # Data transfer objects
│   │   │   │   ├── mapper/          # DTO-Entity converters
│   │   │   │   ├── grpc/            # gRPC client
│   │   │   │   └── exception/       # Custom exceptions & handlers
│   │   │   ├── proto/               # Protocol Buffer definitions
│   │   │   └── resources/
│   │   │       ├── application.properties
│   │   │       └── data.sql         # Sample data
│   │   └── test/
│   ├── Dockerfile
│   └── pom.xml
│
├── billing-service/          # gRPC service
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/pm/billingservice/
│   │   │   │   └── grpc/            # gRPC service implementation
│   │   │   ├── proto/               # Protocol Buffer definitions
│   │   │   └── resources/
│   │   │       └── application.properties
│   │   └── test/
│   ├── Dockerfile
│   └── pom.xml
│
├── api-requests/             # HTTP request examples
│   └── patient-service/
│
└── grpc-requests/            # gRPC request examples
    └── billing-service/
```

## Key Features

### Patient Service

**REST API Endpoints:**
- `GET /patients` - Retrieve all entities
- `POST /patients` - Create new entity
- `PUT /patients/{id}` - Update existing entity
- `DELETE /patients/{id}` - Delete entity

**Technical Highlights:**
- UUID-based primary keys
- Email uniqueness validation
- Custom validation groups for create vs update operations
- Global exception handling with custom exceptions
- Automatic OpenAPI/Swagger documentation
- gRPC client for inter-service communication

### Billing Service

**gRPC Service:**
- `CreateBillingAccount` - RPC method for account creation

**Technical Highlights:**
- gRPC server implementation using `grpc-spring-boot-starter`
- Protocol Buffer message definitions
- Blocking stub for synchronous communication

## gRPC Implementation

### Protocol Buffer Definition (`billing_service.proto`)

```protobuf
syntax = "proto3";

option java_multiple_files = true;
option java_package = "billing";

service BillingService {
  rpc CreateBillingAccount (BillingRequest) returns (BillingResponse);
}

message BillingRequest {
  string patientId = 1;
  string name = 2;
  string email = 3;
}

message BillingResponse {
  string accountId = 1;
  string status = 2;
}
```

### Dependencies

**gRPC Core:**
```xml
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-netty-shaded</artifactId>
    <version>1.69.0</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-protobuf</artifactId>
    <version>1.69.0</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-stub</artifactId>
    <version>1.69.0</version>
</dependency>
```

**Spring Boot Integration:**
```xml
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-spring-boot-starter</artifactId>
    <version>3.1.0.RELEASE</version>
</dependency>
```

### Maven Plugin Configuration

```xml
<plugin>
    <groupId>org.xolstice.maven.plugins</groupId>
    <artifactId>protobuf-maven-plugin</artifactId>
    <version>0.6.1</version>
    <configuration>
        <protocArtifact>com.google.protobuf:protoc:3.25.5:exe:${os.detected.classifier}</protocArtifact>
        <pluginId>grpc-java</pluginId>
        <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.68.1:exe:${os.detected.classifier}</pluginArtifact>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
                <goal>compile-custom</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## Configuration

### Patient Service (Port 4000)

```properties
spring.application.name=patient-service
server.port=4000
logging.level.root=info

# gRPC client configuration
billing.service.address=localhost
billing.service.grcp.port=9001
```

### Billing Service (Ports 4001 & 9001)

```properties
spring.application.name=billing-service
server.port=4001
grpc.server.port=9001
```

## Docker Support

Both services include multi-stage Dockerfiles optimized for production:

**Build Stage:**
- Uses Maven to download dependencies offline
- Compiles and packages the application

**Runtime Stage:**
- Uses lightweight JRE image (eclipse-temurin:21-jre)
- Copies only the built JAR file
- Exposes necessary ports

### Build & Run

```bash
# Build patient-service
cd patient-service
docker build -t patient-service .

# Build billing-service
cd billing-service
docker build -t billing-service .

# Run services
docker run -p 4000:4000 patient-service
docker run -p 4001:4001 -p 9001:9001 billing-service
```

## Database Configuration

The patient service supports both in-memory and persistent databases:

**PostgreSQL (Production):**
- Configure connection in `application.properties`
- Docker volumes are gitignored (`db_volumes/`)

**H2 Database (Testing):**
- Uncomment H2 configuration in `application.properties`
- Access console at `/h2-console`
- Automatic schema creation with `data.sql`

## Validation Strategy

Custom validation groups enable different validation rules for create vs update:

```java
// DTO with conditional validation
@NotBlank(groups = CreatePatientValidationGroup.class, 
          message = "Registered date is required")
private String registeredDate;

// Controller usage
@PostMapping
public ResponseEntity<?> create(
    @Validated({Default.class, CreatePatientValidationGroup.class}) 
    @RequestBody PatientRequestDTO dto) { ... }

@PutMapping("/{id}")
public ResponseEntity<?> update(
    @Validated({Default.class}) 
    @RequestBody PatientRequestDTO dto) { ... }
```

## API Documentation

OpenAPI/Swagger documentation is automatically generated and available at:
- **Swagger UI:** `http://localhost:4000/swagger-ui.html`
- **OpenAPI JSON:** `http://localhost:4000/v3/api-docs`

## Exception Handling

Global exception handler provides consistent error responses:

- **MethodArgumentNotValidException** - Returns field-level validation errors
- **EmailAlreadyExistsException** - Custom business logic exception
- **PatientNotFoundException** - Entity not found exception

All exceptions return structured JSON error responses with appropriate HTTP status codes.

## Development Tools

### HTTP Requests
Sample `.http` files are provided in `api-requests/` and `grpc-requests/` directories for testing with IntelliJ HTTP Client or similar tools.

### Maven Wrapper
The project includes Maven wrapper scripts (`mvnw`, `mvnw.cmd`) to ensure consistent build behavior across different environments.

## Building from Source

```bash
# Build patient-service
cd patient-service
./mvnw clean package

# Build billing-service
cd billing-service
./mvnw clean package

# Run patient-service
java -jar patient-service/target/patient-service-0.0.1-SNAPSHOT.jar

# Run billing-service
java -jar billing-service/target/billing-service-0.0.1-SNAPSHOT.jar
```

## Key Design Patterns

- **Repository Pattern** - Data access abstraction via Spring Data JPA
- **DTO Pattern** - Separation of internal models from API contracts
- **Mapper Pattern** - Manual DTO-Entity conversion
- **Service Layer** - Business logic encapsulation
- **Global Exception Handling** - Centralized error handling with `@ControllerAdvice`
- **Validation Groups** - Context-specific validation rules
- **RPC Pattern** - Synchronous service-to-service communication via gRPC

## Notes

- Both services use the same Protocol Buffer definition (duplicated in both projects)
- The patient service acts as both a REST server and a gRPC client
- The billing service is purely a gRPC server
- UUIDs are used for primary keys instead of auto-incrementing integers
- The project includes sample data initialization via `data.sql`
- All date fields use ISO 8601 format (YYYY-MM-DD) as strings in DTOs

## License

This is a demonstration project showcasing microservices architecture patterns.