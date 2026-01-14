# Microservices Architecture Project

A comprehensive Spring Boot microservices implementation demonstrating modern architectural patterns including service-to-service communication (gRPC), event-driven architecture (Kafka), and REST APIs.

## Architecture Overview

This project consists of three microservices that communicate using REST, gRPC, and Kafka:

- **Patient Service** - Primary service exposing REST API endpoints, gRPC client, and Kafka producer
- **Billing Service** - Secondary service exposing gRPC endpoints for billing account creation
- **Analytics Service** - Event-driven service consuming patient events via Kafka

```
┌─────────────────────┐         gRPC          ┌─────────────────────┐
│  Patient Service    │ ───────────────────> │  Billing Service    │
│  (REST API)         │      Port 9001        │  (gRPC Server)      │
│  Port 4000          │                       │  Port 4001          │
└─────────────────────┘                       └─────────────────────┘
         │
         │ Kafka Events
         ▼
┌─────────────────────┐
│ Analytics Service   │
│ (Kafka Consumer)    │
│ Port 4002           │
└─────────────────────┘
         ▲
         │
    ┌────────┐
    │ Kafka  │
    │ Broker │
    └────────┘
```

## Technology Stack

### Backend Framework
- **Spring Boot 3.4.x** - Main application framework
- **Java 21** - Programming language
- **Maven** - Dependency management and build tool

### Communication Protocols
- **REST API** - HTTP-based API for client communication
- **gRPC** - High-performance RPC framework for inter-service communication
- **Apache Kafka** - Distributed event streaming platform for asynchronous messaging
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
- **Docker Compose** - Multi-container orchestration (optional)
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
│   │   │   │   ├── kafka/           # Kafka producer
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
├── analytics-service/        # Kafka consumer service
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/pm/analyticsservice/
│   │   │   │   └── kafka/           # Kafka consumer
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
├── grpc-requests/            # gRPC request examples
│   └── billing-service/
│
└── dockerFileConfig/         # IntelliJ Docker run configurations
```

## Key Features

### Patient Service

**REST API Endpoints:**
- `GET /patients` - Retrieve all patients
- `POST /patients` - Create new patient
- `PUT /patients/{id}` - Update existing patient
- `DELETE /patients/{id}` - Delete patient

**Technical Highlights:**
- UUID-based primary keys
- Email uniqueness validation
- Custom validation groups for create vs update operations
- Global exception handling with custom exceptions
- Automatic OpenAPI/Swagger documentation at `http://localhost:4000/swagger-ui.html`
- gRPC client for inter-service communication with Billing Service
- Kafka producer for publishing patient events
- PostgreSQL integration with sample data initialization

### Billing Service

**gRPC Service:**
- `CreateBillingAccount` - RPC method for account creation

**Technical Highlights:**
- gRPC server implementation using `grpc-spring-boot-starter`
- Protocol Buffer message definitions
- Blocking stub for synchronous communication
- Lightweight service focused on billing account creation

### Analytics Service

**Kafka Consumer:**
- Consumes patient events from the `patient` topic
- Processes events for analytics and reporting purposes

**Technical Highlights:**
- Event-driven architecture using Apache Kafka
- Protocol Buffer deserialization for event processing
- Consumer group: `analytics-service`
- Asynchronous message processing

## Communication Patterns

### 1. REST API (Client → Patient Service)
Standard HTTP REST endpoints for CRUD operations on patient data.

### 2. gRPC (Patient Service → Billing Service)
Synchronous RPC calls for creating billing accounts when new patients are registered.

**Protocol Buffer Definition:**
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

### 3. Kafka Events (Patient Service → Analytics Service)
Asynchronous event publishing for patient creation events.

**Event Schema:**
```protobuf
syntax = "proto3";

package patient.events;

message PatientEvent {
  string patientId = 1;
  string name = 2;
  string email = 3;
  string event_type = 4;
}
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

# Kafka producer configuration
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.ByteArraySerializer

# Database configuration (PostgreSQL)
# See application.properties for full configuration
```

### Billing Service (Ports 4001 & 9001)

```properties
spring.application.name=billing-service
server.port=4001
grpc.server.port=9001
```

### Analytics Service (Port 4002)

```properties
spring.application.name=analytics-service

# Kafka consumer configuration
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.ByteArrayDeserializer
```

## Getting Started

### Prerequisites

- **Java 21** or higher
- **Maven 3.9+** (or use included Maven wrapper)
- **Docker** (optional, for containerized deployment)
- **PostgreSQL** (for patient-service database)
- **Apache Kafka** (for event streaming)

### Running Locally

#### 1. Start Infrastructure Services

**Option A: Using Docker**

Start PostgreSQL:
```bash
docker run -d \
  --name patient-service-db \
  -e POSTGRES_USER=admin_user \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=db \
  -p 5000:5432 \
  postgres:latest
```

Start Kafka (KRaft mode):
```bash
docker run -d \
  --name kafka \
  -p 9092:9092 \
  -p 9094:9094 \
  -e KAFKA_CFG_NODE_ID=0 \
  -e KAFKA_CFG_PROCESS_ROLES=controller,broker \
  -e KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka:9093 \
  -e KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094 \
  -e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094 \
  -e KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT \
  -e KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  bitnamilegacy/kafka:latest
```

**Option B: Local Installation**
- Install and run PostgreSQL locally
- Install and run Apache Kafka locally

#### 2. Build Services

```bash
# Build patient-service
cd patient-service
./mvnw clean package

# Build billing-service
cd ../billing-service
./mvnw clean package

# Build analytics-service
cd ../analytics-service
./mvnw clean package
```

#### 3. Run Services

**Terminal 1 - Billing Service:**
```bash
cd billing-service
java -jar target/billing-service-0.0.1-SNAPSHOT.jar
```

**Terminal 2 - Analytics Service:**
```bash
cd analytics-service
java -jar target/analytics-service-0.0.1-SNAPSHOT.jar
```

**Terminal 3 - Patient Service:**
```bash
cd patient-service
java -jar target/patient-service-0.0.1-SNAPSHOT.jar
```

### Running with Docker

#### 1. Build Docker Images

```bash
# Build patient-service image
cd patient-service
docker build -t patient-service:latest .

# Build billing-service image
cd ../billing-service
docker build -t billing-service:latest .

# Build analytics-service image
cd ../analytics-service
docker build -t analytics-service:latest .
```

#### 2. Create Docker Network

```bash
docker network create internal
```

#### 3. Run Containers

**Database:**
```bash
docker run -d \
  --name patient-service-db \
  --network internal \
  -e POSTGRES_USER=admin_user \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=db \
  -p 5000:5432 \
  postgres:latest
```

**Kafka:**
```bash
docker run -d \
  --name kafka \
  --network internal \
  -p 9092:9092 \
  -p 9094:9094 \
  -e KAFKA_CFG_NODE_ID=0 \
  -e KAFKA_CFG_PROCESS_ROLES=controller,broker \
  -e KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka:9093 \
  -e KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094 \
  -e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094 \
  -e KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT \
  -e KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  bitnamilegacy/kafka:latest
```

**Billing Service:**
```bash
docker run -d \
  --name billing-service \
  --network internal \
  -p 4001:4001 \
  -p 9001:9001 \
  billing-service:latest
```

**Analytics Service:**
```bash
docker run -d \
  --name analytics-service \
  --network internal \
  -p 4002:4002 \
  -e SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092 \
  analytics-service:latest
```

**Patient Service:**
```bash
docker run -d \
  --name patient-service \
  --network internal \
  -p 4000:4000 \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://patient-service-db:5432/db \
  -e SPRING_DATASOURCE_USERNAME=admin_user \
  -e SPRING_DATASOURCE_PASSWORD=password \
  -e SPRING_JPA_HIBERNATE_DDL_AUTO=update \
  -e SPRING_SQL_INIT_MODE=always \
  -e BILLING_SERVICE_ADDRESS=billing-service \
  -e BILLING_SERVICE_GRPC_PORT=9001 \
  -e SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092 \
  patient-service:latest
```

### Using IntelliJ Docker Run Configurations

The project includes pre-configured IntelliJ run configurations in the `dockerFileConfig/` directory:
- `patient-service-db.run.xml` - PostgreSQL database
- `kafka.run.xml` - Kafka broker
- `billing-service.run.xml` - Billing service
- `analytics-service.run.xml` - Analytics service
- `patient-service.run.xml` - Patient service

These can be imported into IntelliJ IDEA for easy container management.

## Testing the Application

### REST API Testing

Use the sample HTTP requests in `api-requests/patient-service/`:

**Create a Patient:**
```http
POST http://localhost:4000/patients
Content-Type: application/json

{
  "name": "John Smith",
  "email": "john.smith@example.com",
  "address": "123 Main St",
  "dateOfBirth": "1990-01-15",
  "registeredDate": "2025-01-15"
}
```

**Get All Patients:**
```http
GET http://localhost:4000/patients
```

**Update a Patient:**
```http
PUT http://localhost:4000/patients/{id}
Content-Type: application/json

{
  "name": "John Smith Updated",
  "email": "john.smith@example.com",
  "address": "456 Oak Ave",
  "dateOfBirth": "1990-01-15"
}
```

**Delete a Patient:**
```http
DELETE http://localhost:4000/patients/{id}
```

### gRPC Testing

Use the sample request in `grpc-requests/billing-service/`:

```
GRPC localhost:9001/BillingService/CreateBillingAccount
Content-Type: application/json

{
  "patientId": "12334",
  "name": "John Doe",
  "email": "john.doe@example.com"
}
```

### Verifying Kafka Events

When a patient is created, check the analytics-service logs:
```bash
docker logs -f analytics-service
```

You should see:
```
Received Patient Event: [ PatientId=..., PatientName=..., PatientEmail=... ]
```

## API Documentation

OpenAPI/Swagger documentation is automatically generated and available at:
- **Swagger UI:** `http://localhost:4000/swagger-ui.html`
- **OpenAPI JSON:** `http://localhost:4000/v3/api-docs`

## Database Configuration

### PostgreSQL (Production)

The patient service uses PostgreSQL by default. Configure in `application.properties`:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5000/db
spring.datasource.username=admin_user
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

### H2 Database (Testing)

To use in-memory H2 database, uncomment the H2 configuration in `patient-service/src/main/resources/application.properties`:
```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
spring.datasource.url=jdbc:h2:mem:testdb
# ... additional H2 configuration
```

Access H2 console at `http://localhost:4000/h2-console`

## Key Dependencies

### gRPC Dependencies
```xml
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-netty-shaded</artifactId>
    <version>1.69.0</version>
</dependency>
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-spring-boot-starter</artifactId>
    <version>3.1.0.RELEASE</version>
</dependency>
```

### Kafka Dependencies
```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>3.3.0</version>
</dependency>
```

### Protocol Buffers
```xml
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>4.29.1</version>
</dependency>
```

## Maven Plugin Configuration

### Protobuf Maven Plugin
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

## Design Patterns & Best Practices

- **Repository Pattern** - Data access abstraction via Spring Data JPA
- **DTO Pattern** - Separation of internal models from API contracts
- **Mapper Pattern** - Manual DTO-Entity conversion
- **Service Layer** - Business logic encapsulation
- **Global Exception Handling** - Centralized error handling with `@ControllerAdvice`
- **Validation Groups** - Context-specific validation rules
- **Synchronous RPC** - gRPC for inter-service communication
- **Event-Driven Architecture** - Kafka for asynchronous messaging
- **Multi-Stage Docker Builds** - Optimized container images

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
```

## Exception Handling

Global exception handler provides consistent error responses:
- **MethodArgumentNotValidException** - Field-level validation errors
- **EmailAlreadyExistsException** - Custom business logic exception
- **PatientNotFoundException** - Entity not found exception

All exceptions return structured JSON responses with appropriate HTTP status codes.

## Monitoring & Logging

All services use SLF4J with Logback for logging:
- Patient Service: Logs REST requests, gRPC calls, and Kafka events
- Billing Service: Logs incoming gRPC requests
- Analytics Service: Logs consumed Kafka events

## Troubleshooting

### Common Issues

**Kafka Connection Refused:**
- Ensure Kafka is running and accessible
- Check `SPRING_KAFKA_BOOTSTRAP_SERVERS` environment variable
- Verify network connectivity between services

**gRPC Connection Failed:**
- Ensure billing-service is running on port 9001
- Check `BILLING_SERVICE_ADDRESS` and `BILLING_SERVICE_GRPC_PORT` configuration
- Verify network connectivity between services

**Database Connection Failed:**
- Ensure PostgreSQL is running
- Verify database credentials
- Check database URL configuration

**Proto Compilation Errors:**
- Run `mvn clean compile` to regenerate proto classes
- Ensure protobuf-maven-plugin is properly configured



## License

This is a demonstration project showcasing microservices architecture patterns with Spring Boot, gRPC, and Apache Kafka.