# Notes in the Cloud

Notes in the Cloud is a microservice-based platform for notes, todos, reminders, notifications, and note sharing.

The project is built around separate backend services, a frontend application, and an API Gateway that acts as the public entry point to the system.

---

## Services

| Service | Purpose |
|---|---|
| `notes-cloud-frontend` | User interface for notes, todos, reminders, notifications, and sharing |
| `notes-cloud-api-gateway` | Public entry point, routes requests, validates JWT, handles WebSocket connections |
| `notes-cloud-auth-service` | Registration, login, JWT tokens, refresh tokens, user identity |
| `notes-cloud-notes-service` | Creates, updates, deletes, and fetches notes |
| `notes-cloud-todo-service` | Manages todo tasks and todo lists |
| `notes-cloud-reminder-service` | Manages reminders and creates notifications |
| `notes-cloud-sharing-service` | Creates and opens public note share links |
| `migrator` | Runs database migrations |
| `notes-cloud-infrastructure` | Kubernetes manifests and deployment configuration |

---

## System Flow

The frontend should communicate only with the API Gateway.

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
```

The internal services should not be called directly by the browser.

---

## Authentication Flow

```text
1. User logs in or registers from the frontend.
2. Frontend sends the request to the API Gateway.
3. API Gateway forwards auth requests to Auth Service.
4. Auth Service returns an access token and refresh token.
5. Frontend sends the access token in future requests.
6. API Gateway validates the JWT.
7. API Gateway forwards the request to the correct internal service.
```

Protected requests use:

```http
Authorization: Bearer <access_token>
```

After validation, the gateway can forward user context internally, for example:

```http
X-User-Id: <authenticated-user-id>
```

---

## WebSocket Notification Flow

Reminder notifications should be delivered in real time through the gateway.

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

Flow:

```text
1. Frontend opens a WebSocket connection to the API Gateway.
2. API Gateway validates the JWT and stores the connection by userId.
3. Reminder Service creates a notification when a reminder is due.
4. Reminder Service sends the notification to the API Gateway through an internal endpoint.
5. API Gateway sends the notification to the correct frontend WebSocket connection.
```

Example WebSocket endpoint:

```text
/ws/reminders?token=<access_token>
```

Example internal notification push endpoint:

```text
POST /internal/notifications/push
```

---

## Main API Endpoints

All requests go through the API Gateway.

Depending on deployment, the gateway may expose the auth proxy routes under:

```text
/authService/api/v1
```

So the frontend can call:

```text
http://localhost:8090/authService/api/v1
```

---

## Auth Endpoints

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/register` | Public | Register user |
| `POST` | `/login` | Public | Login user |
| `POST` | `/logout` | Public | Logout user |
| `POST` | `/refresh` | Public | Refresh access token |
| `GET` | `/me` | Protected | Get current user |
| `POST` | `/email/verify` | Public | Verify email |
| `POST` | `/email/resend-verification` | Public | Resend verification email |

OAuth endpoints:

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/auth/google/start` | Start Google login |
| `GET` | `/auth/google/callback` | Google callback |
| `GET` | `/auth/gitlab/start` | Start GitLab login |
| `GET` | `/auth/gitlab/callback` | GitLab callback |

---

## Notes Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/notes` | Get all notes |
| `POST` | `/notes` | Create note |
| `GET` | `/notes/{note_id}` | Get note |
| `PUT` | `/notes/{note_id}` | Update note |
| `DELETE` | `/notes/{note_id}` | Delete note |

---

## Sharing Endpoints

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/notes/{note_id}/share-links` | Protected | Create share link |
| `GET` | `/share-links/{token}` | Public | Open shared note |

---

## Todo Endpoints

### Todo Tasks

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/todos` | Get standalone tasks |
| `POST` | `/todos` | Create task |
| `GET` | `/todos/{todo_id}` | Get task |
| `PUT` | `/todos/{todo_id}` | Update task / mark as done |
| `DELETE` | `/todos/{todo_id}` | Delete task |

### Todo Lists

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/todo-lists` | Get todo lists with tasks |
| `POST` | `/todo-lists` | Create todo list |
| `GET` | `/todo-lists/{list_id}` | Get todo list |
| `PUT` | `/todo-lists/{list_id}` | Update todo list |
| `DELETE` | `/todo-lists/{list_id}` | Delete todo list |

Todo behavior:

```text
The Todo page shows active tasks only.
When a task is marked as done, it is hidden from the active task list.
When a todo list is deleted, its tasks can become standalone tasks.
```

---

## Reminder Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/reminders` | Get reminders |
| `POST` | `/reminders` | Create reminder |
| `PUT` | `/reminders` | Update reminder |
| `GET` | `/reminders/{reminder_id}` | Get reminder |
| `DELETE` | `/reminders/{reminder_id}` | Delete reminder |

Query support:

```text
GET /reminders?status=PENDING
GET /reminders?status=COMPLETED
```

---

## Notification Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/notifications` | Get notifications |
| `DELETE` | `/notifications` | Delete all notifications |
| `GET` | `/notifications/unread-count` | Get unread count |
| `POST` | `/notifications/read-all` | Mark all as read |
| `POST` | `/notifications/{notification_id}/read` | Mark one notification as read |

Query support:

```text
GET /notifications?read=true
GET /notifications?read=false
```

---

## Kubernetes Flow

Inside Kubernetes, services communicate using service names:

```text
http://auth-service:8080
http://notes-service:8082
http://todo-service:8085
http://reminder-service:8084
http://sharing-service:8083
http://api-gateway:8090
```

Do not use `localhost` for service-to-service communication inside the cluster.

---

## Local Testing

Port-forward the API Gateway:

```bash
kubectl port-forward -n notes-cloud svc/api-gateway 8090:8090
```

Frontend should call:

```text
http://localhost:8090
```

Example frontend config:

```env
VITE_API_BASE_URL=http://localhost:8090/authService/api/v1
VITE_WS_GATEWAY_URL=ws://localhost:8090
```

---

## CORS

CORS should be handled by the API Gateway.

Final flow:

```text
Browser -> API Gateway -> Internal Services
```

Internal services do not need CORS because the browser does not call them directly.

---

## Summary

Notes in the Cloud uses a microservice architecture where:

```text
API Gateway = public entry point, routing, JWT validation, WebSocket management
Auth Service = users, login, tokens
Notes Service = note logic
Todo Service = todo logic
Reminder Service = reminders and notifications
Sharing Service = public note sharing
Frontend = user interface
```

The main goal is to keep each service focused on one responsibility while the gateway controls access to the system.
