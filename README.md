# ai_recruitment_platform_SERVER
# 🤖 AI Recruitment & Talent Screening Platform

> **Automate hiring with Generative AI** — Resume parsing, candidate ranking, ATS scoring, and interview question generation powered by Spring Boot microservices.

---

## 📐 Architecture Overview

```
Next.js Frontend (TypeScript · Tailwind)
        │
        │  HTTPS / REST
        ▼
┌─────────────────────────────────────────────────────┐
│              API Gateway  :8080                      │
│   Spring Cloud Gateway · JWT Auth · Rate Limiting   │
└────────┬────────┬────────┬────────┬────────┬────────┘
         │        │        │        │        │
    ┌────▼──┐ ┌───▼──┐ ┌───▼────┐ ┌─▼──┐ ┌──▼───────┐ ┌──────────────┐
    │ User  │ │ Job  │ │Resume  │ │ AI │ │Interview │ │Notification  │
    │ :8081 │ │:8082 │ │ :8083  │ │:8084│ │  :8085   │ │    :8086     │
    └───┬───┘ └──┬───┘ └───┬────┘ └─┬──┘ └──┬───────┘ └──────┬───────┘
        │        │         │        │       │                  │
        └────────┴─────────┴────────┴───────┴──────────────────┘
                                    │
                    ┌───────────────▼────────────────┐
                    │       Apache Kafka :9092         │
                    │  resume.uploaded · ai.scored     │
                    │  interview.scheduled · notif.*   │
                    └──┬──────┬──────┬──────┬─────────┘
                       │      │      │      │
                  ┌────▼─┐ ┌──▼─┐ ┌──▼─┐ ┌─▼────────┐  ┌────────┐
                  │user  │ │job │ │res.│ │ ai / int │  │ Redis  │
                  │  db  │ │ db │ │ db │ │   db     │  │ Cache  │
                  └──────┘ └────┘ └────┘ └──────────┘  └────────┘
                                    │
                          ┌─────────▼──────────┐
                          │  OpenAI / Gemini    │
                          │  S3 / MinIO Storage │
                          └────────────────────┘
```

---

## 📁 Project Structure

```
ai-recruitment-platform/
│
├── 📦 pom.xml                        ← Parent POM (manages all versions)
│
├── 🔍 service-registry/              ← Eureka Server (:8761)
│   ├── src/main/java/
│   │   └── ServiceRegistryApplication.java
│   ├── src/main/resources/
│   │   └── application.yml
│   └── pom.xml
│
├── 🚪 api-gateway/                   ← Spring Cloud Gateway (:8080)
│   ├── src/main/java/
│   │   ├── ApiGatewayApplication.java
│   │   ├── filter/
│   │   │   └── JwtAuthenticationFilter.java
│   │   └── config/
│   │       └── GatewayConfig.java
│   ├── src/main/resources/
│   │   └── application.yml
│   └── pom.xml
│
├── ⚙️ config-server/                 ← Spring Cloud Config (:8888)  [optional]
│   ├── src/main/resources/
│   │   └── application.yml
│   └── pom.xml
│
├── 👤 user-service/                  ← Auth · RBAC (:8081)
│   ├── src/main/java/com/recruitment/user/
│   │   ├── UserServiceApplication.java
│   │   ├── controller/
│   │   │   ├── AuthController.java
│   │   │   └── UserController.java
│   │   ├── service/
│   │   │   ├── UserService.java
│   │   │   └── JwtService.java
│   │   ├── repository/
│   │   │   └── UserRepository.java
│   │   ├── entity/
│   │   │   └── User.java
│   │   ├── dto/
│   │   │   ├── LoginRequest.java
│   │   │   ├── RegisterRequest.java
│   │   │   └── AuthResponse.java
│   │   └── security/
│   │       ├── SecurityConfig.java
│   │       └── JwtAuthFilter.java
│   ├── src/main/resources/
│   │   ├── application.yml
│   │   └── db/migration/
│   │       └── V1__init_users.sql
│   └── pom.xml
│
├── 💼 job-service/                   ← Job CRUD · Search (:8082)
│   ├── src/main/java/com/recruitment/job/
│   │   ├── JobServiceApplication.java
│   │   ├── controller/JobController.java
│   │   ├── service/JobService.java
│   │   ├── repository/JobRepository.java
│   │   ├── entity/Job.java
│   │   └── dto/JobRequest.java
│   ├── src/main/resources/
│   │   ├── application.yml
│   │   └── db/migration/V1__init_jobs.sql
│   └── pom.xml
│
├── 📄 resume-service/                ← Parse · Store · S3 (:8083)
│   ├── src/main/java/com/recruitment/resume/
│   │   ├── ResumeServiceApplication.java
│   │   ├── controller/ResumeController.java
│   │   ├── service/
│   │   │   ├── ResumeService.java
│   │   │   └── S3StorageService.java
│   │   ├── kafka/ResumeEventProducer.java
│   │   ├── repository/ResumeRepository.java
│   │   ├── entity/Resume.java
│   │   └── dto/ResumeUploadResponse.java
│   ├── src/main/resources/
│   │   ├── application.yml
│   │   └── db/migration/V1__init_resumes.sql
│   └── pom.xml
│
├── 🤖 ai-service/                    ← Score · Rank · GenAI (:8084)
│   ├── src/main/java/com/recruitment/ai/
│   │   ├── AiServiceApplication.java
│   │   ├── controller/AiController.java
│   │   ├── service/
│   │   │   ├── AtsScoreService.java
│   │   │   ├── CandidateRankingService.java
│   │   │   └── InterviewQuestionService.java
│   │   ├── kafka/
│   │   │   ├── ResumeUploadedConsumer.java
│   │   │   └── AiScoredProducer.java
│   │   ├── client/OpenAiClient.java
│   │   ├── repository/AiScoreRepository.java
│   │   └── entity/AiScore.java
│   ├── src/main/resources/
│   │   ├── application.yml
│   │   └── db/migration/V1__init_ai_scores.sql
│   └── pom.xml
│
├── 🗓️ interview-service/             ← Schedule · Q&A (:8085)
│   ├── src/main/java/com/recruitment/interview/
│   │   ├── InterviewServiceApplication.java
│   │   ├── controller/InterviewController.java
│   │   ├── service/InterviewService.java
│   │   ├── kafka/InterviewEventProducer.java
│   │   ├── repository/InterviewRepository.java
│   │   └── entity/Interview.java
│   ├── src/main/resources/
│   │   ├── application.yml
│   │   └── db/migration/V1__init_interviews.sql
│   └── pom.xml
│
├── 🔔 notification-service/          ← Email · SMS (:8086)
│   ├── src/main/java/com/recruitment/notification/
│   │   ├── NotificationServiceApplication.java
│   │   ├── kafka/NotificationConsumer.java
│   │   ├── service/
│   │   │   ├── EmailService.java
│   │   │   └── SmsService.java
│   │   └── dto/NotificationEvent.java
│   └── pom.xml
│
├── 🧩 common-lib/                    ← Shared DTOs · Utils · Constants
│   ├── src/main/java/com/recruitment/common/
│   │   ├── dto/
│   │   │   ├── ApiResponse.java
│   │   │   └── ErrorResponse.java
│   │   ├── exception/
│   │   │   ├── GlobalExceptionHandler.java
│   │   │   └── ResourceNotFoundException.java
│   │   ├── constants/
│   │   │   └── KafkaTopics.java
│   │   └── util/
│   │       └── JwtUtil.java
│   └── pom.xml
│
├── 🐳 docker-compose.yml             ← Local dev environment
│
└── ☸️ k8s/                           ← Kubernetes manifests
    ├── namespace.yml
    ├── api-gateway-deployment.yml
    ├── user-service-deployment.yml
    ├── kafka-deployment.yml
    └── postgres-statefulset.yml
```

---

## 📦 Complete Dependency Reference

### Parent POM — `pom.xml`

| Artifact | Version | Purpose |
|---|---|---|
| `spring-boot-starter-parent` | `3.3.2` | Spring Boot BOM + plugin mgmt |
| `spring-cloud-dependencies` | `2023.0.2` | Spring Cloud BOM |
| Java | `21` | LTS runtime |

---

### 🚪 API Gateway — `api-gateway/pom.xml`

| Dependency | Group ID | Version | Purpose |
|---|---|---|---|
| `spring-cloud-starter-gateway` | `org.springframework.cloud` | managed | Reactive routing |
| `spring-cloud-starter-netflix-eureka-client` | `org.springframework.cloud` | managed | Service discovery |
| `spring-boot-starter-security` | `org.springframework.boot` | managed | Security config |
| `jjwt-api` | `io.jsonwebtoken` | `0.12.5` | JWT validation |
| `jjwt-impl` | `io.jsonwebtoken` | `0.12.5` | JWT impl (runtime) |
| `jjwt-jackson` | `io.jsonwebtoken` | `0.12.5` | JWT JSON (runtime) |
| `spring-boot-starter-actuator` | `org.springframework.boot` | managed | Health endpoints |
| `lombok` | `org.projectlombok` | managed | Boilerplate reduction |

---

### 🔍 Service Registry — `service-registry/pom.xml`

| Dependency | Group ID | Purpose |
|---|---|---|
| `spring-cloud-starter-netflix-eureka-server` | `org.springframework.cloud` | Eureka server |
| `spring-boot-starter-actuator` | `org.springframework.boot` | Health check |

---

### 👤 User Service — `user-service/pom.xml`

| Dependency | Group ID | Version | Purpose |
|---|---|---|---|
| `spring-boot-starter-web` | `org.springframework.boot` | managed | REST endpoints |
| `spring-boot-starter-data-jpa` | `org.springframework.boot` | managed | ORM / Hibernate |
| `spring-boot-starter-security` | `org.springframework.boot` | managed | Auth + RBAC |
| `spring-boot-starter-validation` | `org.springframework.boot` | managed | Bean validation |
| `spring-boot-starter-actuator` | `org.springframework.boot` | managed | Health + metrics |
| `spring-cloud-starter-netflix-eureka-client` | `org.springframework.cloud` | managed | Service registry |
| `spring-kafka` | `org.springframework.kafka` | managed | Kafka producer |
| `postgresql` | `org.postgresql` | managed | JDBC driver |
| `flyway-core` | `org.flywaydb` | managed | DB migrations |
| `jjwt-api` | `io.jsonwebtoken` | `0.12.5` | JWT creation |
| `jjwt-impl` | `io.jsonwebtoken` | `0.12.5` | JWT impl |
| `jjwt-jackson` | `io.jsonwebtoken` | `0.12.5` | JWT JSON |
| `lombok` | `org.projectlombok` | managed | Code gen |
| `spring-boot-starter-test` | `org.springframework.boot` | managed | Unit tests |
| `spring-security-test` | `org.springframework.security` | managed | Security tests |

---

### 💼 Job Service — `job-service/pom.xml`

| Dependency | Group ID | Purpose |
|---|---|---|
| `spring-boot-starter-web` | `org.springframework.boot` | REST endpoints |
| `spring-boot-starter-data-jpa` | `org.springframework.boot` | ORM |
| `spring-boot-starter-validation` | `org.springframework.boot` | Validation |
| `spring-boot-starter-actuator` | `org.springframework.boot` | Health |
| `spring-cloud-starter-netflix-eureka-client` | `org.springframework.cloud` | Discovery |
| `spring-kafka` | `org.springframework.kafka` | Events |
| `postgresql` | `org.postgresql` | DB driver |
| `flyway-core` | `org.flywaydb` | Migrations |
| `lombok` | `org.projectlombok` | Code gen |
| `spring-boot-starter-test` | `org.springframework.boot` | Tests |

---

### 📄 Resume Service — `resume-service/pom.xml`

| Dependency | Group ID | Version | Purpose |
|---|---|---|---|
| `spring-boot-starter-web` | `org.springframework.boot` | managed | REST + multipart upload |
| `spring-boot-starter-data-jpa` | `org.springframework.boot` | managed | ORM |
| `spring-boot-starter-data-redis` | `org.springframework.boot` | managed | Redis cache |
| `spring-boot-starter-validation` | `org.springframework.boot` | managed | Validation |
| `spring-boot-starter-actuator` | `org.springframework.boot` | managed | Health |
| `spring-cloud-starter-netflix-eureka-client` | `org.springframework.cloud` | managed | Discovery |
| `spring-kafka` | `org.springframework.kafka` | managed | Publish `resume.uploaded` |
| `postgresql` | `org.postgresql` | managed | DB driver |
| `flyway-core` | `org.flywaydb` | managed | Migrations |
| `s3` | `software.amazon.awssdk` | `2.26.7` | S3 file storage |
| `apache-tika-core` | `org.apache.tika` | `2.9.2` | PDF/DOCX text extract |
| `lombok` | `org.projectlombok` | managed | Code gen |
| `spring-boot-starter-test` | `org.springframework.boot` | managed | Tests |

---

### 🤖 AI Service — `ai-service/pom.xml`

| Dependency | Group ID | Version | Purpose |
|---|---|---|---|
| `spring-boot-starter-web` | `org.springframework.boot` | managed | REST endpoints |
| `spring-boot-starter-data-jpa` | `org.springframework.boot` | managed | ORM |
| `spring-boot-starter-actuator` | `org.springframework.boot` | managed | Health |
| `spring-cloud-starter-netflix-eureka-client` | `org.springframework.cloud` | managed | Discovery |
| `spring-kafka` | `org.springframework.kafka` | managed | Consume + produce events |
| `postgresql` | `org.postgresql` | managed | DB driver |
| `flyway-core` | `org.flywaydb` | managed | Migrations |
| `spring-ai-openai-spring-boot-starter` | `org.springframework.ai` | `1.0.0-M1` | OpenAI / Gemini calls |
| `spring-boot-starter-webflux` | `org.springframework.boot` | managed | Non-blocking HTTP client |
| `lombok` | `org.projectlombok` | managed | Code gen |
| `spring-boot-starter-test` | `org.springframework.boot` | managed | Tests |

> **Note:** For Spring AI, add the milestone repository to your `pom.xml`:
> ```xml
> <repositories>
>   <repository>
>     <id>spring-milestones</id>
>     <url>https://repo.spring.io/milestone</url>
>   </repository>
> </repositories>
> ```

---

### 🗓️ Interview Service — `interview-service/pom.xml`

| Dependency | Group ID | Purpose |
|---|---|---|
| `spring-boot-starter-web` | `org.springframework.boot` | REST |
| `spring-boot-starter-data-jpa` | `org.springframework.boot` | ORM |
| `spring-boot-starter-validation` | `org.springframework.boot` | Validation |
| `spring-boot-starter-actuator` | `org.springframework.boot` | Health |
| `spring-cloud-starter-netflix-eureka-client` | `org.springframework.cloud` | Discovery |
| `spring-kafka` | `org.springframework.kafka` | Publish `interview.scheduled` |
| `postgresql` | `org.postgresql` | DB driver |
| `flyway-core` | `org.flywaydb` | Migrations |
| `lombok` | `org.projectlombok` | Code gen |
| `spring-boot-starter-test` | `org.springframework.boot` | Tests |

---

### 🔔 Notification Service — `notification-service/pom.xml`

| Dependency | Group ID | Version | Purpose |
|---|---|---|---|
| `spring-boot-starter-web` | `org.springframework.boot` | managed | REST |
| `spring-boot-starter-mail` | `org.springframework.boot` | managed | SMTP email via JavaMail |
| `spring-boot-starter-actuator` | `org.springframework.boot` | managed | Health |
| `spring-cloud-starter-netflix-eureka-client` | `org.springframework.cloud` | managed | Discovery |
| `spring-kafka` | `org.springframework.kafka` | managed | Consume `notification.send` |
| `twilio` | `com.twilio.sdk` | `10.4.1` | SMS delivery |
| `thymeleaf` | `org.springframework.boot` | managed | Email templates |
| `lombok` | `org.projectlombok` | managed | Code gen |
| `spring-boot-starter-test` | `org.springframework.boot` | managed | Tests |

---

### 🧩 Common Library — `common-lib/pom.xml`

| Dependency | Group ID | Purpose |
|---|---|---|
| `spring-boot-starter` | `org.springframework.boot` | Base |
| `spring-boot-starter-web` | `org.springframework.boot` | For shared exception handlers |
| `jjwt-api` | `io.jsonwebtoken` | Shared JWT utilities |
| `lombok` | `org.projectlombok` | Code gen |
| `jakarta.validation-api` | `jakarta.validation` | Validation annotations |

---

## 🐳 Local Development with Docker Compose

```bash
# Start all infrastructure (Postgres × 5, Kafka, Redis)
docker compose up -d

# Verify containers
docker compose ps
```

**Services started by Docker Compose:**

| Container | Image | Port | Purpose |
|---|---|---|---|
| `postgres-user` | `postgres:16` | 5432 | user_db |
| `postgres-job` | `postgres:16` | 5433 | job_db |
| `postgres-resume` | `postgres:16` | 5434 | resume_db |
| `postgres-ai` | `postgres:16` | 5435 | ai_db |
| `postgres-interview` | `postgres:16` | 5436 | interview_db |
| `redis` | `redis:7` | 6379 | Cache + Sessions |
| `zookeeper` | `confluentinc/cp-zookeeper:7.6.0` | 2181 | Kafka dependency |
| `kafka` | `confluentinc/cp-kafka:7.6.0` | 9092 | Event bus |

---

## 🚀 Service Startup Order

```
1. docker compose up -d          → Infrastructure
2. service-registry  :8761       → Eureka (wait ~15s)
3. config-server     :8888       → Config (optional)
4. api-gateway       :8080       → Gateway
5. user-service      :8081       → Auth
6. job-service       :8082
7. resume-service    :8083
8. ai-service        :8084
9. interview-service :8085
10. notification-service :8086
```

Verify all registered at: http://localhost:8761

---

## ⚙️ Environment Variables

Create a `.env` file (never commit to git):

```env
# PostgreSQL
POSTGRES_PASSWORD=your_secret_password

# JWT
JWT_SECRET=your-256-bit-secret-key-here
JWT_EXPIRY_MS=86400000

# OpenAI
OPENAI_API_KEY=sk-...

# AWS S3
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_S3_BUCKET=recruitment-resumes
AWS_REGION=ap-south-1

# Twilio (SMS)
TWILIO_ACCOUNT_SID=AC...
TWILIO_AUTH_TOKEN=...
TWILIO_FROM_NUMBER=+1...

# Email (SMTP)
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your@gmail.com
MAIL_PASSWORD=your_app_password
```

---

## 📡 Kafka Topics

| Topic | Producer | Consumer | Payload |
|---|---|---|---|
| `resume.uploaded` | resume-service | ai-service | `{ resumeId, jobId, s3Key }` |
| `ai.scored` | ai-service | interview-service | `{ resumeId, candidateId, atsScore }` |
| `interview.scheduled` | interview-service | notification-service | `{ interviewId, candidateEmail, dateTime }` |
| `notification.send` | any service | notification-service | `{ to, channel, templateId, payload }` |

---

## 🔐 Security Flow

```
Client → API Gateway
  ├── POST /api/auth/login    → user-service (no JWT required)
  ├── POST /api/auth/register → user-service (no JWT required)
  └── ALL other routes        → JwtAuthenticationFilter validates token
                                 → Forward with X-User-Id header to downstream
```

**Roles:** `ADMIN` · `RECRUITER` · `CANDIDATE`

---

## 🤖 AI Resume Workflow (Event-Driven)

```
1. Candidate uploads PDF/DOCX  →  resume-service
2. resume-service stores file  →  S3 / MinIO
3. Publishes event             →  Kafka: resume.uploaded
4. ai-service consumes event
5. Calls Apache Tika           →  extract raw text
6. Calls OpenAI / Gemini       →  extract skills, experience
7. Calculates ATS score (0–100)
8. Ranks candidate against job
9. Generates interview questions
10. Publishes event            →  Kafka: ai.scored
11. interview-service creates slot
12. Publishes event            →  Kafka: interview.scheduled
13. notification-service sends email/SMS to candidate
```

---

## 🔧 Tech Stack Summary

| Layer | Technology | Version |
|---|---|---|
| **Language** | Java | 21 (LTS) |
| **Framework** | Spring Boot | 3.3.2 |
| **Architecture** | Microservices | — |
| **Gateway** | Spring Cloud Gateway | 2023.0.2 |
| **Service Discovery** | Netflix Eureka | 2023.0.2 |
| **Database** | PostgreSQL | 16 |
| **ORM** | Spring Data JPA / Hibernate | 6.x |
| **Migrations** | Flyway | 10.x |
| **Messaging** | Apache Kafka | 7.6.0 (Confluent) |
| **Cache** | Redis | 7 |
| **Security** | Spring Security + JWT (jjwt) | 0.12.5 |
| **AI / LLM** | Spring AI (OpenAI / Gemini) | 1.0.0-M1 |
| **File Parse** | Apache Tika | 2.9.2 |
| **File Storage** | AWS S3 SDK | 2.26.7 |
| **SMS** | Twilio SDK | 10.4.1 |
| **Email** | Spring Mail + Thymeleaf | managed |
| **Observability** | Zipkin + Micrometer | managed |
| **Containers** | Docker | latest |
| **Orchestration** | Kubernetes / AWS EKS | — |
| **Frontend** | Next.js + TypeScript + Tailwind | — |
| **Build** | Maven | 3.9+ |

---

## 📊 API Endpoints Reference

### User Service `:8081`
| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/api/auth/register` | Register new user | ❌ |
| `POST` | `/api/auth/login` | Login & get JWT | ❌ |
| `GET` | `/api/users/me` | Get current user | ✅ |
| `PUT` | `/api/users/{id}` | Update user | ✅ ADMIN |

### Job Service `:8082`
| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/api/jobs` | List all jobs | ✅ |
| `POST` | `/api/jobs` | Create job | ✅ RECRUITER |
| `GET` | `/api/jobs/{id}` | Get job by ID | ✅ |
| `DELETE` | `/api/jobs/{id}` | Delete job | ✅ ADMIN |

### Resume Service `:8083`
| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/api/resumes/upload` | Upload resume (multipart) | ✅ CANDIDATE |
| `GET` | `/api/resumes/{id}` | Get resume metadata | ✅ |
| `GET` | `/api/resumes/job/{jobId}` | All resumes for a job | ✅ RECRUITER |

### AI Service `:8084`
| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/api/ai/score/{resumeId}` | Get ATS score | ✅ |
| `GET` | `/api/ai/rank/{jobId}` | Ranked candidates list | ✅ RECRUITER |
| `POST` | `/api/ai/questions/{resumeId}` | Generate interview questions | ✅ RECRUITER |

### Interview Service `:8085`
| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/api/interviews` | Schedule interview | ✅ RECRUITER |
| `GET` | `/api/interviews/{id}` | Get interview details | ✅ |
| `PUT` | `/api/interviews/{id}/status` | Update status | ✅ RECRUITER |

---

## 🧪 Running Tests

```bash
# Unit tests for a specific service
cd user-service && mvn test

# All services
mvn test --projects user-service,job-service,resume-service

# Integration tests (requires Docker)
mvn verify -P integration-tests
```

---

## ☸️ Kubernetes Deployment (AWS EKS)

```bash
# Create namespace
kubectl apply -f k8s/namespace.yml

# Deploy infrastructure
kubectl apply -f k8s/postgres-statefulset.yml
kubectl apply -f k8s/kafka-deployment.yml

# Deploy services
kubectl apply -f k8s/service-registry-deployment.yml
kubectl apply -f k8s/api-gateway-deployment.yml
kubectl apply -f k8s/user-service-deployment.yml
# ... repeat for each service

# Check rollout
kubectl rollout status deployment/user-service -n recruitment
```

---

## 📈 Observability

| Tool | URL | Purpose |
|---|---|---|
| Eureka Dashboard | http://localhost:8761 | Service registry |
| Zipkin | http://localhost:9411 | Distributed tracing |
| Actuator (per svc) | http://localhost:{port}/actuator/health | Health check |
| Kafka UI | http://localhost:8090 | Topic & message browser |

---

## 🗺️ Future Enhancements

- [ ] Video interview analysis (Whisper API transcription)
- [ ] Resume fraud detection (similarity scoring)
- [ ] LinkedIn OAuth integration
- [ ] Multilingual resume support (i18n)
- [ ] Real-time notifications via WebSocket
- [ ] A/B testing for ranking algorithms
- [ ] GraphQL API layer

---

## 🤝 Contributing

1. Fork the repo
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Commit: `git commit -m "feat: add your feature"`
4. Push: `git push origin feat/your-feature`
5. Open a Pull Request

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

*Built with ☕ Java · 🍃 Spring Boot · 🤖 GenAI*