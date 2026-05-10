# Notes Cloud Todo Service

Todo Service is a Spring Boot microservice for managing todo lists and todo tasks in the Notes Cloud platform.

The service supports:

- Creating, updating, deleting and reading todo lists
- Creating, updating, deleting and reading todo tasks
- Standalone tasks that do not belong to a list
- Tasks attached to a todo list
- Marking tasks as done
- Returning todo lists together with their active tasks
- Returning standalone active tasks
- PostgreSQL persistence
- Kubernetes deployment support
- Health and readiness checks through Spring Boot Actuator

## Tech Stack

- Java 21
- Spring Boot
- Spring Web
- Spring Data JPA
- Spring Validation
- PostgreSQL
- Docker
- Kubernetes
- Maven

## Project Structure

```text
todo-service/
├── src/
│   ├── main/
│   │   ├── java/com/notescloud/todo_service/
│   │   │   ├── controller/
│   │   │   ├── domain/
│   │   │   ├── dto/
│   │   │   ├── exception/
│   │   │   ├── repository/
│   │   │   ├── service/
│   │   │   └── TodoServiceApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
├── pom.xml
└── Dockerfile
```

## Environment Variables

The service expects the following environment variables when running in Kubernetes:

```text
SERVER_PORT=8085
DB_HOST=postgres
DB_PORT=5432
POSTGRES_DB=notes_cloud
POSTGRES_USER=notes_cloud_user
POSTGRES_PASSWORD=notes_cloud_password
SPRING_JPA_HIBERNATE_DDL_AUTO=validate
```

For local development through port-forwarding:

```text
DB_HOST=localhost
DB_PORT=15432
POSTGRES_DB=notes_cloud
POSTGRES_USER=notes_cloud_user
POSTGRES_PASSWORD=notes_cloud_password
```

## Running Locally

From the service folder:

```bash
./mvnw spring-boot:run
```

Or with Maven installed:

```bash
mvn spring-boot:run
```

The service runs on:

```text
http://localhost:8085
```

## Local Database Port Forward

If PostgreSQL is running inside the Kubernetes cluster, expose it locally with:

```bash
kubectl port-forward svc/postgres 15432:5432 -n notes-cloud
```

Then the service can connect locally to:

```text
localhost:15432
```

You need to keep the port-forward terminal running while accessing the database locally.

## Health Checks

### General health

```http
GET /actuator/health
```

Example:

```bash
curl http://localhost:8085/actuator/health
```

### Liveness

```http
GET /actuator/health/liveness
```

Example:

```bash
curl http://localhost:8085/actuator/health/liveness
```

### Readiness

```http
GET /actuator/health/readiness
```

Example:

```bash
curl http://localhost:8085/actuator/health/readiness
```

## API Endpoints

Base URL:

```text
http://localhost:8085
```

## Todo List Endpoints

### Create todo list

```http
POST /api/todo-lists
```

Request body:

```json
{
  "userId": "8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f",
  "title": "Web Project Tasks"
}
```

Example response:

```json
{
  "id": "b74f4f78-2eb5-4ef4-9c35-7e08c21a9d33",
  "userId": "8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f",
  "title": "Web Project Tasks",
  "createdAt": "2026-05-04T11:35:00",
  "updatedAt": "2026-05-04T11:35:00"
}
```

### Get all todo lists for a user

```http
GET /api/todo-lists?userId={userId}
```

Example:

```bash
curl "http://localhost:8085/api/todo-lists?userId=8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f"
```

### Get todo list by id

```http
GET /api/todo-lists/{id}
```

Example:

```bash
curl http://localhost:8085/api/todo-lists/b74f4f78-2eb5-4ef4-9c35-7e08c21a9d33
```

### Get todo lists with their tasks

```http
GET /api/todo-lists/with-tasks?userId={userId}
```

Example:

```bash
curl "http://localhost:8085/api/todo-lists/with-tasks?userId=8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f"
```

Example response:

```json
[
  {
    "id": "b74f4f78-2eb5-4ef4-9c35-7e08c21a9d33",
    "title": "Web Project Tasks",
    "createdAt": "2026-05-04T11:35:00",
    "updatedAt": "2026-05-04T11:35:00",
    "tasks": [
      {
        "id": "38d3719a-8b97-4460-9ee9-e96c6112d5a5",
        "listId": "b74f4f78-2eb5-4ef4-9c35-7e08c21a9d33",
        "userId": "8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f",
        "title": "Test todo-service endpoints",
        "done": false,
        "priority": "HIGH",
        "dueDate": "2026-05-08T18:00:00",
        "createdAt": "2026-05-04T11:40:00",
        "updatedAt": "2026-05-04T11:40:00"
      }
    ]
  }
]
```

### Update todo list

```http
PUT /api/todo-lists/{id}
```

Request body:

```json
{
  "title": "Updated Web Project Tasks"
}
```

Example:

```bash
curl -X PUT http://localhost:8085/api/todo-lists/b74f4f78-2eb5-4ef4-9c35-7e08c21a9d33 \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated Web Project Tasks"}'
```

### Delete todo list

```http
DELETE /api/todo-lists/{id}
```

Example:

```bash
curl -X DELETE http://localhost:8085/api/todo-lists/b74f4f78-2eb5-4ef4-9c35-7e08c21a9d33
```

Expected response:

```text
204 No Content
```

## Todo Task Endpoints

### Create standalone todo task

Standalone tasks do not have a `listId`.

```http
POST /api/todo-tasks
```

Request body:

```json
{
  "userId": "8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f",
  "title": "Finish todo-service CRUD",
  "priority": "HIGH",
  "dueDate": "2026-05-08T18:00:00"
}
```

Example response:

```json
{
  "id": "38d3719a-8b97-4460-9ee9-e96c6112d5a5",
  "listId": null,
  "userId": "8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f",
  "title": "Finish todo-service CRUD",
  "done": false,
  "priority": "HIGH",
  "dueDate": "2026-05-08T18:00:00",
  "createdAt": "2026-05-04T12:00:00",
  "updatedAt": "2026-05-04T12:00:00"
}
```

### Create todo task attached to a list

```http
POST /api/todo-tasks
```

Request body:

```json
{
  "listId": "b74f4f78-2eb5-4ef4-9c35-7e08c21a9d33",
  "userId": "8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f",
  "title": "Prepare final project demo",
  "priority": "MEDIUM",
  "dueDate": "2026-05-10T12:00:00"
}
```

### Get todo task by id

```http
GET /api/todo-tasks/{id}
```

Example:

```bash
curl http://localhost:8085/api/todo-tasks/38d3719a-8b97-4460-9ee9-e96c6112d5a5
```

### Get all todo tasks for a user

Recommended endpoint:

```http
GET /api/todo-tasks?userId={userId}
```

Alternative endpoint if the controller uses `/all`:

```http
GET /api/todo-tasks/all?userId={userId}
```

Example:

```bash
curl "http://localhost:8085/api/todo-tasks?userId=8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f"
```

### Get standalone todo tasks

Returns tasks where `listId` is `null`.

```http
GET /api/todo-tasks/standalone?userId={userId}
```

Example:

```bash
curl "http://localhost:8085/api/todo-tasks/standalone?userId=8f3a0f2d-5d3a-4e9a-bfa1-6a1b2c3d4e5f"
```

### Get tasks by list id

```http
GET /api/todo-tasks/list/{listId}
```

Example:

```bash
curl http://localhost:8085/api/todo-tasks/list/b74f4f78-2eb5-4ef4-9c35-7e08c21a9d33
```

### Update todo task

```http
PUT /api/todo-tasks/{id}
```

Request body:

```json
{
  "title": "Updated task name",
  "priority": "LOW",
  "dueDate": "2026-05-12T15:30:00"
}
```

Example:

```bash
curl -X PUT http://localhost:8085/api/todo-tasks/38d3719a-8b97-4460-9ee9-e96c6112d5a5 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated task name",
    "priority": "LOW",
    "dueDate": "2026-05-12T15:30:00"
  }'
```

### Mark task as done

```http
PUT /api/todo-tasks/{id}/done
```

Example:

```bash
curl -X PUT http://localhost:8085/api/todo-tasks/38d3719a-8b97-4460-9ee9-e96c6112d5a5/done
```

### Reopen task

```http
PUT /api/todo-tasks/{id}/reopen
```

Example:

```bash
curl -X PUT http://localhost:8085/api/todo-tasks/38d3719a-8b97-4460-9ee9-e96c6112d5a5/reopen
```

### Delete todo task

```http
DELETE /api/todo-tasks/{id}
```

Example:

```bash
curl -X DELETE http://localhost:8085/api/todo-tasks/38d3719a-8b97-4460-9ee9-e96c6112d5a5
```

Expected response:

```text
204 No Content
```

## Request DTOs

### CreateTodoListRequest

```json
{
  "userId": "UUID",
  "title": "string"
}
```

### UpdateTodoListRequest

```json
{
  "title": "string"
}
```

### CreateTodoTaskRequest

```json
{
  "listId": "UUID or null",
  "userId": "UUID",
  "title": "string",
  "priority": "LOW | MEDIUM | HIGH",
  "dueDate": "yyyy-MM-ddTHH:mm:ss"
}
```

For standalone tasks, omit `listId`.

### UpdateTodoTaskRequest

```json
{
  "title": "string",
  "priority": "LOW | MEDIUM | HIGH",
  "dueDate": "yyyy-MM-ddTHH:mm:ss"
}
```

## Error Response Format

When an error occurs, the service returns a response like:

```json
{
  "message": "Todo task not found with id: 38d3719a-8b97-4460-9ee9-e96c6112d5a5",
  "status": 404,
  "timestamp": "2026-05-04T14:20:00"
}
```

Validation errors return status `400`.

Example:

```json
{
  "message": "title: Title is required",
  "status": 400,
  "timestamp": "2026-05-04T14:20:00"
}
```

## Docker

Build the image locally:

```bash
docker build -t hristo12319/notes-cloud-todo-service:v1 .
```

Build and push a multi-platform image:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t hristo12319/notes-cloud-todo-service:v1 \
  --push .
```

Check the image platforms:

```bash
docker buildx imagetools inspect hristo12319/notes-cloud-todo-service:v1
```

Expected platforms:

```text
linux/amd64
linux/arm64
```

## Kubernetes

The service is deployed in the `notes-cloud` namespace.

Apply the deployment:

```bash
kubectl apply -f todo-service-deployment.yaml
```

Restart the deployment:

```bash
kubectl rollout restart deployment todo-service -n notes-cloud
```

Check rollout status:

```bash
kubectl rollout status deployment todo-service -n notes-cloud
```

Check pods:

```bash
kubectl get pods -n notes-cloud
```

Check logs:

```bash
kubectl logs -n notes-cloud deployment/todo-service
```

Port-forward the service:

```bash
kubectl port-forward -n notes-cloud svc/todo-service 8085:8085
```

Then access it locally:

```text
http://localhost:8085
```

## Database Connection in Kubernetes

Inside the Kubernetes cluster, the service connects to PostgreSQL using:

```text
postgres:5432
```

Example JDBC URL:

```text
jdbc:postgresql://postgres:5432/notes_cloud
```

The Deployment should provide these environment variables:

```text
DB_HOST=postgres
DB_PORT=5432
POSTGRES_DB=notes_cloud
POSTGRES_USER=notes_cloud_user
POSTGRES_PASSWORD=notes_cloud_password
```

## Database Connection Locally

For local development with the database running in Kubernetes:

```bash
kubectl port-forward svc/postgres 15432:5432 -n notes-cloud
```

Then connect to:

```text
Host: localhost
Port: 15432
Database: notes_cloud
User: notes_cloud_user
Password: notes_cloud_password
```

## Notes

- A todo task can be standalone or attached to a todo list.
- A standalone task has `listId = null`.
- A completed task is not deleted automatically.
- Marking a task as done sets `done = true`.
- Deleting a task removes it from the database.
- Deleting a todo list should not delete standalone tasks.
- If a todo list is deleted, attached tasks should either be detached or handled through the database foreign key rule.
