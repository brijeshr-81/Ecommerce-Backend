# Ecommerce-Backend

This repository contains a .NET-based microservice architecture for a simple ecommerce website.

---

## 1 — Recommended stack (practical, .NET-focused)

- Platform: .NET 8 (or latest LTS available to you)
- API Gateway: YARP (Microsoft’s reverse-proxy) or Ocelot (community). Recommend YARP for long-term support.
- Auth / OAuth2: OpenIddict (self-hosted OAuth2/OpenID Connect in ASP.NET Core) OR a hosted solution (Azure AD B2C) if you want managed identity. This guide assumes OpenIddict for an all-in-.NET, developer-friendly flow.
- Message broker: RabbitMQ (simple, reliable, good .NET support) or Apache Kafka (if very high throughput). Recommend RabbitMQ for an ecommerce prototype.
- Messaging library: MassTransit (works well with RabbitMQ and handles message contracts, retries, sagas).
- Primary DB: PostgreSQL (reliable, ACID, open-source, great EF Core support).
- Fast cache / ephemeral store: Redis (for Cart sessions / ephemeral cart).
- Data access: EF Core (Npgsql) for Postgres; Redis client: StackExchange.Redis.
- CI/CD: GitHub Actions or Azure DevOps.
- Containerization: Docker and Docker Compose for local development.

---

## 2 — High-level architecture & flow

1. Client → API Gateway (YARP). The gateway enforces authentication (JWT) and routes requests to downstream microservices.
2. Products Service (Read model) serves `GET /products` (backed by Postgres).
3. Cart Service stores carts in Redis (fast) and emits events (e.g. `CartUpdated`) to RabbitMQ via MassTransit.
4. Order Service subscribes to cart events and/or accepts `POST /orders` to create orders in Postgres. It publishes `OrderCreated` events.
5. Payment Service handles payment requests (or subscribes to `OrderCreated`), integrates with external payment gateways (Stripe/PayPal), and publishes `PaymentSucceeded` / `PaymentFailed` events to update order state.
6. Services communicate through RabbitMQ (commands/events). Use MassTransit state machines or sagas to orchestrate the order lifecycle and handle retries/compensation.
7. OpenIddict issues JWTs to clients. Individual services validate tokens and enforce authorization scopes/roles.

This README provides a recommended starting point and architecture overview for building the microservice-based ecommerce backend in .NET. Adjust stack choices (e.g., Kafka instead of RabbitMQ, or Azure AD B2C instead of OpenIddict) to match your team's operational preferences and scale requirements.

# Ecommerce Microservices (.NET)

This repository contains a simple ecommerce system built using .NET microservices.

## Architecture
- Products Service
- Cart Service
- Orders Service
- Payments Service
- API Gateway (YARP)
- Auth Server (OAuth 2.0 / OpenID Connect)

## Infrastructure
- PostgreSQL
- Redis
- RabbitMQ
- Docker & Docker Compose

## Current Status
- Phase 1: Local infrastructure setup complete

## How to run infrastructure
```bash
cd infra
docker compose -f docker-compose.infra.yml up -d