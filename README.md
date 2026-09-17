# Students API

A simple **CRUD REST API built with Go** for managing student records. The project uses Go's standard `net/http` package for HTTP routing and **SQLite** for persistent local storage.

## Features

- Create a student
- Get all students
- Get a student by ID
- Update a student
- Delete a student
- JSON request/response handling
- Request validation using `go-playground/validator`
- Configuration loaded from YAML using `cleanenv`
- SQLite database using `go-sqlite3`
- Graceful server shutdown
- Structured logging with Go's `log/slog`

## Tech Stack

- **Language:** Go 1.26.6
- **HTTP:** `net/http`
- **Database:** SQLite
- **SQLite Driver:** `github.com/mattn/go-sqlite3`
- **Validation:** `github.com/go-playground/validator/v10`
- **Configuration:** YAML + `github.com/ilyakaznacheev/cleanenv`

## Project Structure

```text
Students-api/
├── cmd/
│   └── students-api/
│       └── main.go
├── config/
│   └── local.yaml
├── internal/
│   ├── config/
│   │   └── config.go
│   ├── http/
│   │   └── handlers/
│   │       └── student/
│   │           └── student.go
│   ├── storage/
│   │   ├── sqlite/
│   │   │   └── sqlite.go
│   │   └── storage.go
│   ├── types/
│   │   └── types.go
│   └── utils/
│       └── response/
│           └── response.go
├── go.mod
├── go.sum
└── README.md
```

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/students` | Create a student |
| `GET` | `/api/students` | Get all students |
| `GET` | `/api/students/{id}` | Get a student by ID |
| `PUT` | `/api/students/{id}` | Update a student |
| `DELETE` | `/api/students/{id}` | Delete a student |

## Student Model

```json
{
  "id": 1,
  "name": "Rahul",
  "email": "rahul@gmail.com",
  "age": 22
}
```

For creating or updating a student, send:

```json
{
  "name": "Rahul",
  "email": "rahul@gmail.com",
  "age": 22
}
```

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/WhooShubh/students-api.git
cd students-api
```

### 2. Install dependencies

```bash
go mod tidy
```

### 3. Create the configuration file

Create:

```text
config/local.yaml
```

with:

```yaml
env: "dev"
storage_path: "storage/storage.db"
http_server:
  address: "localhost:8082"
```

The application reads the configuration path from the `CONFIG_PATH` environment variable.

### 4. Create the storage directory

Create a `storage` directory in the project root:

```text
Students-api/
└── storage/
```

The SQLite database file will be created automatically at:

```text
storage/storage.db
```

### 5. Windows users: enable CGO

This project uses `go-sqlite3`, which requires CGO.

Make sure GCC is installed and available in your PATH. For example, with MSYS2 UCRT64:

```powershell
$env:CGO_ENABLED="1"
$env:Path += ";C:\msys64\ucrt64\bin"
```

Then verify:

```powershell
gcc --version
```

### 6. Set the configuration path

PowerShell:

```powershell
$env:CONFIG_PATH="config/local.yaml"
```

### 7. Run the API

```bash
go run ./cmd/students-api
```

The server runs on:

```text
http://localhost:8082
```

## Testing the APIs

You can test the endpoints using **Postman**, `curl`, or any REST client.

### Create Student

**POST**

```text
http://localhost:8082/api/students
```

Body → `raw` → `JSON`:

```json
{
  "name": "Rahul",
  "email": "rahul@gmail.com",
  "age": 22
}
```

Example response:

```json
{
  "id": 1
}
```

### Get All Students

**GET**

```text
http://localhost:8082/api/students
```

Example response:

```json
[
  {
    "id": 1,
    "name": "Rahul",
    "email": "rahul@gmail.com",
    "age": 22
  }
]
```

### Get Student by ID

**GET**

```text
http://localhost:8082/api/students/1
```

### Update Student

**PUT**

```text
http://localhost:8082/api/students/1
```

Body:

```json
{
  "name": "Rahul Updated",
  "email": "rahul.updated@gmail.com",
  "age": 23
}
```

Example response:

```json
{
  "message": "student updated successfully"
}
```

### Delete Student

**DELETE**

```text
http://localhost:8082/api/students/1
```

Example response:

```json
{
  "message": "student deleted successfully"
}
```

## Validation

The student model validates the required fields:

```go
type Student struct {
    Id    int64  `json:"id"`
    Name  string `json:"name" validate:"required"`
    Email string `json:"email" validate:"required"`
    Age   int    `json:"age" validate:"required"`
}
```

Invalid or missing required fields result in a `400 Bad Request` response.

## Architecture

The request flow is organized into separate layers:

```text
HTTP Request
     ↓
HTTP Handler
     ↓
Storage Interface
     ↓
SQLite Implementation
     ↓
SQLite Database
```

The `Storage` interface keeps the HTTP handlers independent of the database implementation. The SQLite package implements the storage operations.

## Database

The application automatically creates the `students` table when the SQLite storage is initialized:

```sql
CREATE TABLE IF NOT EXISTS students (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    email TEXT,
    age INTEGER
);
```

## Graceful Shutdown

The application listens for operating-system signals such as `SIGINT` and `SIGTERM` and performs a graceful HTTP server shutdown with a 5-second timeout.

## Author

**Shubhansh Srivastava**

GitHub: https://github.com/WhooShubh
