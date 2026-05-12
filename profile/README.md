# Notes in the Cloud

Notes in the Cloud is a microservice-based platform for notes, todos, reminders, notifications, and note sharing.

For setup and running instructions, use the infrastructure repository:

```text
https://github.com/notes-in-the-cloud/notes-cloud-infrastructure
```

That repository contains the Kubernetes manifests, database migrations setup, NodePort configuration, and the scripts for starting the whole project.

---

## Main Local URLs

When the project is started through the infrastructure setup:

```text
Frontend: http://localhost:8080
Gateway:  http://localhost:8090
```

The browser should communicate with the system through the API Gateway.

```text
Frontend -> API Gateway -> Internal Services
```

Internal services should not be called directly by the browser.

---

## Services

| Service | Purpose |
|---|---|
| `notes-cloud-frontend` | User interface for notes, todos, reminders, notifications, and sharing |
| `notes-cloud-api-gateway` | Public entry point, routing, JWT validation, WebSocket connections |
| `notes-cloud-auth-service` | Registration, login, JWT, refresh tokens, OAuth, user identity |
| `notes-cloud-notes-service` | Creates, updates, deletes, and fetches notes |
| `notes-cloud-todo-service` | Manages todo tasks and todo lists |
| `notes-cloud-reminder-service` | Manages reminders and creates notifications |
| `notes-cloud-sharing-service` | Creates and opens public note share links |
| `migrator` | Runs PostgreSQL database migrations |
| `notes-cloud-infrastructure` | Kubernetes manifests and deployment scripts |

---

## Architecture

```text
Frontend
   |
   v
API Gateway
   |
   +--> Auth Service
   +--> Notes Service
   +--> Todo Service
   +--> Reminder Service
   +--> Sharing Service
   +--> PostgreSQL
```

Inside Kubernetes, services communicate by service name:

```text
http://auth-service:8081
http://notes-service:8082
http://sharing-service:8083
http://reminder-service:8084
http://todo-service:8085
http://api-gateway:8090
```

---

## API Gateway

The gateway is exposed locally on:

```text
http://localhost:8090
```

The frontend should use the gateway as the API entry point:

```text
http://localhost:8090/api/v1
```

Protected requests use:

```http
Authorization: Bearer <access_token>
```

---

## Frontend

The frontend is exposed locally on:

```text
http://localhost:8080
```

Public shared notes are opened through the frontend:

```text
http://localhost:8080/shared/{token}
```

The frontend config should point API calls and WebSocket connections to the gateway:

```text
API_BASE_URL=http://localhost:8090/api/v1
GATEWAY_BASE_URL=http://localhost:8090
WS_REMINDERS_URL=ws://localhost:8090/ws
```

---

## WebSocket Notifications

Real-time reminder notifications go through the gateway.

```text
Frontend
   |
   | WebSocket
   v
API Gateway
   ^
   | internal HTTP push
Reminder Service
```

The frontend connects to:

```text
ws://localhost:8090/ws?token=<access_token>
```

The reminder service sends internal notification pushes to the gateway.

---

## Database

PostgreSQL is created and initialized through the infrastructure and migrator setup.

The migrator creates and updates the database schema through SQL migrations.

Typical database connection inside Kubernetes:

```text
Host: postgres
Port: 5432
Database: notes_cloud
User: notes_cloud_user
```

The services use environment variables such as:

```text
DB_HOST=postgres
DB_PORT=5432
POSTGRES_DB=notes_cloud
DB_USER=notes_cloud_user
DB_PASSWORD=notes_cloud_password
```

---

## Health and Readiness

Services expose health endpoints for Kubernetes probes.

Common custom endpoints:

```text
/healthz
/readyz
```

`/healthz` checks if the application is alive.

`/readyz` checks if the service is ready, usually including a database check.

Some services also expose Spring Boot Actuator endpoints:

```text
/actuator/health
/actuator/health/liveness
/actuator/health/readiness
```

---

## Main Features

- User registration and login
- JWT authentication and refresh tokens
- OAuth login support
- Notes CRUD
- Todo lists and todo tasks
- Reminders
- In-app notifications
- Real-time WebSocket notifications
- Public note sharing through secure tokens
- PostgreSQL persistence
- Kubernetes deployment

---

## Running the Project

Use the infrastructure repository:

```text
https://github.com/notes-in-the-cloud/notes-cloud-infrastructure
```

General flow:

```bash
git clone https://github.com/notes-in-the-cloud/notes-cloud-infrastructure.git
cd notes-cloud-infrastructure
./setup.sh
```

After the setup finishes, open:

```text
http://localhost:8080
```

The gateway should be available at:

```text
http://localhost:8090
```

---

## Notes

- The frontend should call only the API Gateway.
- Internal services communicate through Kubernetes service names.
- Do not use `localhost` for service-to-service communication inside Kubernetes.
- `localhost:8080` is for the frontend from the host machine.
- `localhost:8090` is for the gateway from the host machine.
- CORS should be handled by the API Gateway.
- Public share links should point to the frontend, not directly to the sharing service.
