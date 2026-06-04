# User Service

A production-ready User Management & Authentication Microservice built with Spring Boot, Spring Security, JWT, PostgreSQL, Eureka Discovery Client, OpenAPI, and JPA.

---

## Overview

The User Service is responsible for:

- User Registration
- User Authentication
- JWT Access & Refresh Token Management
- User Profile Management
- Role-Based User Management
- Service Discovery with Eureka
- API Documentation with Swagger/OpenAPI
- Global Exception Handling
- Secure REST APIs using Spring Security

---

## Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                    UserServiceApplication                   │
│     @SpringBootApplication · @EnableDiscoveryClient        │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
                    Registers With Eureka
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       Eureka Server                         │
│                            :8761                            │
└─────────────────────────────────────────────────────────────┘

                              ▲
                              │
                              │
┌─────────────────────────────────────────────────────────────┐
│                      Client Request                         │
│                 Next.js Frontend / API Gateway             │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 JwtAuthenticationFilter                     │
│                  OncePerRequestFilter                       │
└─────────────────────────────┬───────────────────────────────┘
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
         Public Endpoint             Protected Endpoint
                │                           │
                ▼                           ▼
      Controller Executes         JWT Validation Process
                                            │
                                            ▼
                              ┌─────────────────────────┐
                              │       JwtService        │
                              │ Generate / Validate JWT │
                              └─────────────┬───────────┘
                                            │
                                            ▼
                              ┌─────────────────────────┐
                              │ UserDetailsServiceImpl  │
                              │ Load User By Username   │
                              └─────────────┬───────────┘
                                            │
                                            ▼
                              SecurityContextHolder
                                            │
                                            ▼
                                   Authenticated User
```

---

# Project Structure

```text
user-service
│
├── pom.xml
│
└── src
    │
    ├── main
    │   │
    │   ├── java
    │   │   └── com
    │   │       └── server
    │   │           └── userservice
    │   │
    │   │               ├── UserServiceApplication.java
    │   │
    │   │               ├── config
    │   │               │   ├── SecurityConfig.java
    │   │               │   ├── OpenApiConfig.java
    │   │               │   └── ApplicationConfig.java
    │   │
    │   │               ├── controller
    │   │               │   ├── AuthController.java
    │   │               │   └── UserController.java
    │   │
    │   │               ├── dto
    │   │               │   ├── request
    │   │               │   │   ├── RegisterRequest.java
    │   │               │   │   ├── LoginRequest.java
    │   │               │   │   └── UpdateProfileRequest.java
    │   │               │   │
    │   │               │   └── response
    │   │               │       ├── AuthResponse.java
    │   │               │       └── UserResponse.java
    │   │
    │   │               ├── entity
    │   │               │   ├── User.java
    │   │               │   └── Role.java
    │   │
    │   │               ├── exception
    │   │               │   ├── GlobalExceptionHandler.java
    │   │               │   ├── UserAlreadyExistsException.java
    │   │               │   ├── UserNotFoundException.java
    │   │               │   └── InvalidTokenException.java
    │   │
    │   │               ├── repository
    │   │               │   └── UserRepository.java
    │   │
    │   │               ├── security
    │   │               │   └── JwtAuthenticationFilter.java
    │   │
    │   │               └── service
    │   │                   ├── AuthService.java
    │   │                   ├── UserService.java
    │   │                   ├── JwtService.java
    │   │                   └── UserDetailsServiceImpl.java
    │   │
    │   └── resources
    │       ├── application.yml
    │       └── application-dev.yml
    │
    └── test
        └── java
            └── com
                └── server
                    └── userservice
                        ├── controller
                        │   └── AuthControllerTest.java
                        └── service
                            └── AuthServiceTest.java
```

---

# Package Responsibilities

## config

Configuration classes used across the application.

| Class | Responsibility |
|---------|---------------|
| SecurityConfig | Security filter chain, endpoint authorization, CORS |
| OpenApiConfig | Swagger/OpenAPI configuration |
| ApplicationConfig | PasswordEncoder, AuthenticationManager, AuthenticationProvider |

---

## controller

REST API entry points.

| Controller | Endpoints |
|------------|-----------|
| AuthController | `/api/v1/auth/**` |
| UserController | `/api/v1/users/**` |

---

## service

Business logic layer.

| Service | Responsibility |
|----------|---------------|
| AuthService | Registration, Login, Refresh Token |
| UserService | User Profile Operations |
| JwtService | JWT Generation & Validation |
| UserDetailsServiceImpl | Spring Security User Lookup |

---

## security

Authentication and request filtering.

| Component | Responsibility |
|------------|---------------|
| JwtAuthenticationFilter | Extract JWT, Validate Token, Authenticate Request |

---

## repository

Database access layer.

| Repository | Responsibility |
|------------|---------------|
| UserRepository | User CRUD & Custom Queries |

---

## entity

Database models.

### User

```java
@Entity
public class User implements UserDetails
```

Fields:

```text
id
email
password
firstName
lastName
role
enabled
createdAt
updatedAt
```

### Role

```java
public enum Role
```

```text
ADMIN
RECRUITER
CANDIDATE
```

---

## dto

Data transfer objects.

### Requests

```text
RegisterRequest
LoginRequest
UpdateProfileRequest
```

### Responses

```text
AuthResponse
UserResponse
```

---

## exception

Centralized exception handling.

| Exception | Description |
|------------|------------|
| UserAlreadyExistsException | Duplicate email registration |
| UserNotFoundException | User not found |
| InvalidTokenException | Invalid JWT |
| GlobalExceptionHandler | Handles all API exceptions |

---

# Authentication Flow

```text
Client Request
      │
      ▼
JwtAuthenticationFilter
      │
      ▼
Extract Bearer Token
      │
      ▼
JwtService.validateToken()
      │
      ▼
UserDetailsServiceImpl
      │
      ▼
Load User From Database
      │
      ▼
SecurityContextHolder
      │
      ▼
Authenticated Request
      │
      ▼
Controller
      │
      ▼
Service Layer
      │
      ▼
Repository
      │
      ▼
PostgreSQL
```

---

# REST API Endpoints

## Authentication

| Method | Endpoint | Description |
|----------|-----------|------------|
| POST | `/api/v1/auth/register` | Register User |
| POST | `/api/v1/auth/login` | Login User |
| POST | `/api/v1/auth/refresh` | Refresh Access Token |

---

## User Management

| Method | Endpoint | Description |
|----------|-----------|------------|
| GET | `/api/v1/users/me` | Current User Profile |
| GET | `/api/v1/users/{id}` | User By ID |
| PUT | `/api/v1/users/me` | Update Profile |
| DELETE | `/api/v1/users/me` | Delete Account |

---

# Database

## PostgreSQL

```text
Database      : user_db
Port          : 5432
ORM           : Spring Data JPA
Migration     : Flyway
```

Core Tables:

```text
users
refresh_tokens
```

---

# Security Stack

```text
Spring Security
JWT Authentication
BCrypt Password Encoding
Stateless Sessions
Role-Based Authorization
CORS Configuration
Authentication Manager
Authentication Provider
```

---

# API Documentation

Swagger UI

```text
http://localhost:8081/swagger-ui.html
```

OpenAPI Specification

```text
http://localhost:8081/v3/api-docs
```

---

# Testing

Unit and integration tests are organized by layer.

```text
controller/
└── AuthControllerTest.java

service/
└── AuthServiceTest.java
```

Run Tests:

```bash
mvn test
```

---

# Technology Stack

| Category | Technology |
|-----------|------------|
| Language | Java 21 |
| Framework | Spring Boot |
| Security | Spring Security + JWT |
| Database | PostgreSQL |
| ORM | Spring Data JPA |
| Migration | Flyway |
| Service Discovery | Eureka |
| API Documentation | OpenAPI / Swagger |
| Build Tool | Maven |
| Testing | JUnit 5 + Mockito |

---

# Application Startup

```bash
# Clone repository
git clone <repository-url>

# Move into project
cd user-service

# Build project
mvn clean install

# Run application
mvn spring-boot:run
```

Application starts on:

```text
http://localhost:8081
```

---

# Design Principles

- Layered Architecture
- Separation of Concerns
- DTO-Based API Design
- Stateless Authentication
- Centralized Exception Handling
- Service Discovery Ready
- Production-Oriented Structure
- Testable & Maintainable Codebase