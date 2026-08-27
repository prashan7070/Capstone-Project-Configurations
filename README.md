# ⚙️ Capstone Project Configurations

Centralized Configuration Repository consumed by the Spring Cloud Config Server for the entire Capstone microservices ecosystem.

---

## 📋 Student & Submission Details

| Field | Details |
|---|---|
| **Student Name** | Prashan Anupama |
| **Student Number** | 241711044 |
| **Slack Handle** | prashan anupama |
| **GCP Project ID** |	prashan-gcp-project |
| **Submission Type** | Alternative Option (Capstone Project) |

---

## 📖 Project Description

This repository serves as the single source of truth for configuration across the entire Capstone microservices platform. It is polled directly by the `config-server` (from the Backend Microservices Platform repo), which exposes these `.yaml` files to every downstream microservice at startup and refresh time — decoupling configuration from application code entirely, in line with the Twelve-Factor App methodology.

Each microservice fetches its own configuration file (matched by `spring.application.name`) plus the shared `application.yaml`, enabling centralized, environment-agnostic, and version-controlled configuration management across the distributed system.

---

## 📑 Key Configuration Files

| File | Consumed By | Purpose |
|---|---|---|
| `application.yaml` | All Services | Global shared configuration, including Eureka client zones and default settings inherited by every microservice. |
| `api-gateway.yaml` | `api-gateway` | Route predicates and filters, global CORS policy, and `spring.codec.max-in-memory-size` adjustments for large request handling. |
| `student-service.yaml` | `student-service` | PostgreSQL connection details, Tomcat `max-swallow-size: -1` (to support large multipart uploads), and the target GCS bucket name. |
| `program-service.yaml` | `program-service` | PostgreSQL connection string and JPA/Hibernate settings. |
| `enrollment-service.yaml` | `enrollment-service` | MongoDB connection URI and database settings. |

---

## 🏛️ How This Repository Fits the Architecture

```text
  ┌───────────────────────────────┐
  │  Capstone Project             │
  │  Configurations (this repo)   │
  │                               │
  │  application.yaml             │
  │  api-gateway.yaml             │
  │  student-service.yaml         │
  │  program-service.yaml         │
  │  enrollment-service.yaml      │
  └────────────────┬──────────────┘
                   │  Git-backed polling
                   ▼
  ┌───────────────────────────────┐
  │  config-server (Port 9000)    │
  │  Spring Cloud Config Server   │
  └────────────────┬──────────────┘
                   │  HTTP fetch on startup / refresh
        ┌──────────┼───────────┬─────────────┬──────────────┐
        ▼          ▼           ▼             ▼              ▼
   api-gateway student-svc program-svc enrollment-svc service-registry
```

---

## 📁 Repository Structure

```text
capstone-project-configurations/
├── application.yaml             # Global shared config (Eureka zones, defaults)
├── api-gateway.yaml             # Gateway routes, CORS, in-memory buffer size
├── student-service.yaml         # PostgreSQL, GCS bucket name, Tomcat settings
├── program-service.yaml         # PostgreSQL connection
├── enrollment-service.yaml      # MongoDB connection
└── README.md
```

---

## 🛠️ Technology Stack

| Category | Technology |
|---|---|
| **Configuration Format** | YAML |
| **Configuration Server** | Spring Cloud Config Server |
| **Backing Store** | Git (this repository) |
| **Service Discovery Integration** | Netflix Eureka |
| **Cloud Provider** | Google Cloud Platform (GCP) |
| **Consumed By** | Spring Boot 3.3.2 / Spring Cloud 2023.0.3 microservices |

---

## 🚀 Local Setup / Getting Started

This repository does not run as a standalone application — it is a passive Git-backed data source read by the `config-server`. To use it locally:

---

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/<your-username>/capstone-project-configurations.git
```

---

### 2️⃣ Point Your Local config-server to This Repository
In the `config-server`'s `application.yaml` (or `bootstrap.yaml`), set the Git URI to your local clone path or the remote GitHub URL:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/<your-username>/capstone-project-configurations.git
          default-label: main
          clone-on-start: true
```

---

### 3️⃣ Start the Config Server
```bash
cd config-server
mvn spring-boot:run
```

---

### 4️⃣ Verify Configuration Is Served Correctly
```bash
# Global config
curl http://localhost:9000/application/default

# Service-specific config
curl http://localhost:9000/student-service/default
curl http://localhost:9000/program-service/default
curl http://localhost:9000/enrollment-service/default
curl http://localhost:9000/api-gateway/default
```

Each endpoint should return the merged YAML properties for that service, combining `application.yaml` with the service-specific file.

---

## ✏️ Making Configuration Changes

1. Edit the relevant `.yaml` file in this repository.
2. Commit and push to the `main` branch.
3. Trigger a config refresh on the target microservice via its `/actuator/refresh` endpoint (requires Spring Boot Actuator + `@RefreshScope` beans), or restart the instance to pick up changes.

```bash
curl -X POST http://localhost:<service-port>/actuator/refresh
```

---

## 📄 License
This project was developed as part of the Enterprise Cloud Architecture university module (Capstone Project — Alternative Option).
