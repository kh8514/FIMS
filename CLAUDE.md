# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
# Build all modules
./gradlew build

# Build a specific module
./gradlew :user-service:build

# Run tests
./gradlew test
./gradlew :user-service:test

# Run a service (from project root)
./gradlew :discovery:bootRun
./gradlew :gateway:bootRun
./gradlew :user-service:bootRun
```

Start services in order: **discovery → gateway → user-service**

## Architecture

Spring Boot 3.4.3 / Java 21 multi-module Gradle project with Spring Cloud microservices.

**Modules:**
- `common` — shared library (not a runnable service); jar-only (bootJar disabled). Contains standardized response wrapper (`Api<T>`, `Result`) and exception hierarchy (`ApiException`, `BusinessException`, `GlobalExceptionHandler`, `ErrorCode` enum).
- `discovery` — Eureka Server on port 8761. Does not register itself with Eureka.
- `gateway` — Spring Cloud Gateway (WebFlux/reactive) on port 8000. Routes `/api/users/**` → `lb://user-service` with `StripPrefix=1`. Contains `CustomFilter` for pre/post logging.
- `user-service` — REST service on port 8081. Depends on `:common`. Registers with Eureka using `prefer-ip-address: true`.

**Request flow:** Client → Gateway (8000) → Eureka load-balanced discovery → User Service (8081)

**Response conventions:** All responses should use `Api<T>` wrapper with a `Result` (resultCode, resultMessage, resultDescription). Error messages use Korean strings (e.g., `성공`, `에러발생`).

**Service discovery URLs:**
- Eureka dashboard: `http://localhost:8761`
- Gateway routes: `http://localhost:8000/api/users/{path}`
- Direct service: `http://localhost:8081/users/{path}`