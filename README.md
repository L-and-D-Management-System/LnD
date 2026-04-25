# Multiple Training Management System
## I&D (Innovation & Development) Department

---

## Architecture Diagramm

```
┌─────────────────────────────────────────────────┐
│                  React Frontend                  │
│              (Vite + Tailwind CSS)               │
│                Port: 5173                        │
└──────────────────────┬──────────────────────────┘
                       │ HTTP (Axios)
                       ▼
┌─────────────────────────────────────────────────┐
│                  API Gateway                     │
│         JWT Validation + Proxy Routing           │
│                Port: 3000                        │
└──┬──────────┬──────────┬──────────┬─────────────┘
   │          │          │          │
   ▼          ▼          ▼          ▼
┌──────┐  ┌──────┐  ┌────────┐  ┌──────────┐
│ Auth │  │ User │  │Training│  │Attendance│
│ :3001│  │ :3002│  │  :3003 │  │  :3004   │
└──┬───┘  └──┬───┘  └──┬─────┘  └──┬───────┘
   │         │          │            │
   ▼         ▼          ▼            ▼
┌──────┐  ┌──────┐  ┌────────┐  ┌──────────┐
│auth  │  │user  │  │programs│  │attendance│
│_db   │  │_db   │  │_db     │  │_db       │
└──────┘  └──────┘  └────────┘  └──────────┘
            (All MongoDB on localhost:27017)
```

---

## Prerequisites

- Node.js >= 18.x
- MongoDB running locally on port 27017
- npm

---

## Project Structure

```
training-management-system/
├── api-gateway/          # Port 3000 – JWT gateway + proxy
├── auth-service/         # Port 3001 – Register/Login/JWT
├── user-service/         # Port 3002 – Employee profiles CRUD
├── training-service/     # Port 3003 – Training programs CRUD
├── attendance-service/   # Port 3004 – Enrollment + Attendance + Progress
└── frontend/             # Port 5173 – React + Tailwind CSS UI
```

---

## Setup & Run Instructions

### Step 1 — Start MongoDB
Ensure MongoDB is running locally:
```bash
# If installed as a service it may already be running.
# Otherwise start it manually:
mongod
```

### Step 2 — Start API Gateway
```bash
cd api-gateway
npm start
```

### Step 3 — Start Auth Service
```bash
cd auth-service
npm start
```

### Step 4 — Start User Service
```bash
cd user-service
npm start
```

### Step 5 — Start Training Service
```bash
cd training-service
npm start
```

### Step 6 — Start Attendance Service
```bash
cd attendance-service
npm start
```

### Step 7 — Start Frontend
```bash
cd frontend
npm run dev
```

Open: **http://localhost:5173**

---

## Environment Variables (auto-configured)

Each service has a `.env` file pre-configured for local development. No changes needed to run locally.

| Service | Port | DB Name |
|---|---|---|
| API Gateway | 3000 | — |
| Auth Service | 3001 | training_auth_db |
| User Service | 3002 | training_user_db |
| Training Service | 3003 | training_programs_db |
| Attendance Service | 3004 | training_attendance_db |

---

## API Reference (Postman Format)

### Base URL: `http://localhost:3000/api`

### 1. Auth Endpoints (No token required)

**Register:**
```http
POST /api/auth/register
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123",
  "role": "Employee",
  "department": "R&D"
}
```

**Login:**
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "password123"
}
```
Response: `{ "token": "<JWT>", "user": { "id": "...", "role": "Employee" } }`

---

### 2. User Endpoints (Token required)

```http
GET    /api/users                     # Get all users (Admin/Trainer)
GET    /api/users/me                  # Get current user profile
GET    /api/users/:id                 # Get user by ID
PUT    /api/users/:id                 # Update user (Admin only)
DELETE /api/users/:id                 # Delete user (Admin only)
```

---

### 3. Training Endpoints (Token required)

```http
GET    /api/trainings                 # Get all trainings
GET    /api/trainings/stats           # Get stats (Admin)
GET    /api/trainings/:id             # Get training by ID
GET    /api/trainings/trainer/:id     # Get trainings by trainer
POST   /api/trainings                 # Create training (Admin/Trainer)
PUT    /api/trainings/:id             # Update training (Admin/Trainer)
DELETE /api/trainings/:id             # Delete training (Admin)
```

**Create Training Body:**
```json
{
  "title": "Advanced React Development",
  "description": "Deep dive into React patterns",
  "trainerId": "<trainer-user-id>",
  "trainerName": "Jane Smith",
  "startDate": "2026-05-01",
  "endDate": "2026-05-15",
  "capacity": 25,
  "status": "Upcoming",
  "category": "Technology"
}
```

---

### 4. Enrollment Endpoints (Token required)

```http
POST   /api/attendance/enrollments              # Enroll employee
GET    /api/attendance/enrollments              # Get all enrollments
GET    /api/attendance/enrollments/my           # Get my enrollments
GET    /api/attendance/enrollments/training/:id # Get by training
PATCH  /api/attendance/enrollments/:id/progress # Update progress
DELETE /api/attendance/enrollments/:id          # Unenroll
GET    /api/attendance/enrollments/stats        # Enrollment stats
```

**Enroll Body:**
```json
{
  "userId": "<user-id>",
  "userName": "John Doe",
  "trainingId": "<training-id>",
  "trainingTitle": "Advanced React Development"
}
```

---

### 5. Attendance Endpoints (Token required)

```http
POST /api/attendance/attendance/mark                  # Mark attendance
GET  /api/attendance/attendance/training/:id          # Get by training
GET  /api/attendance/attendance/my                    # My attendance
GET  /api/attendance/attendance/report/:trainingId    # Full report
```

**Mark Attendance Body:**
```json
{
  "userId": "<user-id>",
  "trainingId": "<training-id>",
  "date": "2026-05-01",
  "present": true
}
```

---

## Roles & Permissions

| Feature | Admin | Trainer | Employee |
|---|---|---|---|
| View Dashboard | ✅ Full stats | ✅ Own trainings | ✅ Own progress |
| Create Training | ✅ | ✅ | ❌ |
| Edit Training | ✅ | ✅ | ❌ |
| Delete Training | ✅ | ❌ | ❌ |
| View All Users | ✅ | ✅ | ❌ |
| Edit/Delete Users | ✅ | ❌ | ❌ |
| Enroll Employees | ✅ | ✅ | ❌ |
| Mark Attendance | ✅ | ✅ | ❌ |
| Update Progress | ✅ | ✅ | ❌ |
| View Own Progress | ✅ | ✅ | ✅ |
| View Reports | ✅ | ✅ | ❌ |

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + Vite + Tailwind CSS |
| Charts | Recharts |
| HTTP Client | Axios |
| Routing | React Router v6 |
| API Gateway | Express + http-proxy-middleware |
| Backend Services | Node.js + Express |
| Database | MongoDB + Mongoose |
| Authentication | JWT (jsonwebtoken) |
| Password Hashing | bcryptjs |
| Validation | Joi |
