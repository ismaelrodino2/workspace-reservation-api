# Workspace Reservation System — Backend & IoT

> **Backend + IoT technical assessment built in 2 days against a written specification, including the optional containerization requirement.**

Backend for a coworking workspace reservation system built as part of a **Darien Technology technical assessment**.

The system combines a REST API with relational persistence, reservation business rules, MQTT-based IoT communication, real-time WebSocket updates, automated testing, and separate Docker environments.

**Frontend:** [Practical-Test-Darien-Technology-Frontend](https://github.com/ismaelrodino2/workspace-reservation-web)

---

## Tech Stack

**Backend**

* Node.js
* TypeScript
* Express.js
* Prisma
* PostgreSQL

**Real-time & IoT**

* MQTT
* WebSockets

**Testing & Infrastructure**

* Jest
* Docker
* Docker Compose

---

## What I Built

The backend exposes a REST API for managing coworking locations, spaces, and reservations.

Beyond basic CRUD operations, it implements domain rules around reservation availability and usage limits, while also acting as the bridge between IoT devices and the frontend's real-time monitoring dashboard.

### API

All routes are protected using the `x-api-key` header.

```text
GET                    /locations
GET/POST/PUT/DELETE    /spaces
GET/POST/PUT/DELETE    /reservations
```

Reservations support pagination:

```text
/reservations?page=1&pageSize=10
```

---

## Business Rules

Reservation creation is validated at the backend instead of relying on the client.

The API prevents:

* Overlapping reservations for the same space
* More than **3 reservations per client per week**
* Reservations scheduled in the past

This keeps the domain rules consistent regardless of the client consuming the API.

---

## IoT & Real-Time Architecture

The backend subscribes to MQTT topics using wildcard subscriptions:

```text
sites/+/offices/+/telemetry
sites/+/offices/+/reported
sites/+/offices/+/desired
```

Telemetry received through MQTT is broadcast to connected frontend clients through WebSockets.

```text
IoT Device / Simulator
        │
        │ MQTT
        ▼
┌─────────────────────┐
│   MQTT Subscriber   │
│  wildcard topics    │
└──────────┬──────────┘
           │
           │ telemetry events
           ▼
┌─────────────────────┐
│   Node / Express    │
└──────────┬──────────┘
           │
           │ WebSocket
           ▼
┌─────────────────────┐
│  Real-time Admin    │
│     Dashboard       │
└─────────────────────┘
```

The WebSocket endpoint is available at:

```text
/telemetry?x-api-key=YOUR_KEY
```

This architecture allows telemetry updates to reach the dashboard without continuous HTTP polling.

---

## Docker & Multi-Environment Setup

Containerization was an **optional requirement worth extra points** in the original assessment.

The project includes dedicated configurations for production, development, and testing:

```text
Dockerfile
Dockerfile.dev

docker-compose.yml
docker-compose.dev.yml
docker-compose.test.yml
```

This provides isolated and reproducible environments instead of depending on locally installed infrastructure.

### Development

```bash
docker compose -f docker-compose.dev.yml up --build
```

### Production

```bash
docker compose up --build
```

### Testing

```bash
docker compose -f docker-compose.test.yml up --build
```

See [`docker.md`](./docker.md) for the complete Docker setup.

---

## Running Locally

### Prerequisites

* Node.js
* pnpm
* PostgreSQL

### 1. Install dependencies

```bash
pnpm install
```

### 2. Configure environment variables

Copy the example environment file:

```bash
cp example.env .env
```

Configure at least:

```env
DATABASE_URL=your_postgresql_connection
API_KEY=your_api_key
```

### 3. Run database migrations

```bash
pnpm prisma:migrate
```

### 4. Start the API

```bash
pnpm dev
```

---

## IoT Simulator

Telemetry can be generated using the included simulator.

Examples:

```bash
# SITE_A
node index.js --site-id SITE_A --office-id OFFICE_1
node index.js --site-id SITE_A --office-id OFFICE_2

# SITE_B
node index.js --site-id SITE_B --office-id OFFICE_3
node index.js --site-id SITE_B --office-id OFFICE_4
```

The backend consumes the MQTT messages and forwards telemetry updates to connected WebSocket clients.

---

## Testing

Automated tests are implemented with **Jest**.

The project also provides a dedicated Docker Compose environment for testing, isolated from development and production.

See [`TEST.md`](./TEST.md) for the full testing setup.

---

## System Architecture

```text
                    ┌────────────────────┐
                    │  Next.js Frontend  │
                    │                    │
                    │ Spaces             │
                    │ Reservations       │
                    │ IoT Dashboard      │
                    └─────────┬──────────┘
                              │
                    REST API + WebSocket
                              │
                              ▼
                    ┌────────────────────┐
                    │ Node.js / Express  │
                    │                    │
                    │ Business Rules     │
                    │ MQTT Subscriber    │
                    │ WebSocket Server   │
                    └─────────┬──────────┘
                              │
                         Prisma ORM
                              │
                              ▼
                       ┌────────────┐
                       │ PostgreSQL │
                       └────────────┘

IoT Simulator ── MQTT ──► Backend ── WebSocket ──► Dashboard
```

---

## Technical Assessment Context

This project was developed against a **written external specification with a maximum duration of 2 days**.

The assessment covered API design, relational persistence, business rules, IoT communication, real-time updates, testing, error handling, and documentation.

Containerization with Docker and Docker Compose was listed as an **optional extra-points requirement** and was implemented with separate development, testing, and production configurations.

The frontend was developed as a separate application:

👉 [View the Frontend Repository](https://github.com/ismaelrodino2/workspace-reservation-web)
