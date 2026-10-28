# SkillBridge AI

SkillBridge AI is an AI career and resume intelligence platform for students and job seekers. The first module includes a responsive web app, REST API boundaries implemented in the existing Express service, MongoDB-backed authentication, JWT sessions, and a career dashboard foundation.

This project runs locally without Docker.

## What is included

- Register and login with email/password
- Password hashing with bcrypt
- JWT access tokens
- Authenticated current-user endpoint
- MongoDB connection with a reusable database helper
- Dashboard summary endpoint for resume count, ATS score, matched jobs, roadmap progress, and profile completion
- Authenticated PDF/DOCX resume upload with private object storage, text extraction, and an initial ATS score
- Responsive SkillBridge AI login, registration, and dashboard screens
- Shared OpenAPI contract and generated React Query hooks

## Project structure

```text
artifacts/
  api-server/
    src/
      lib/auth.ts          # Password hashing, JWTs, public user mapping
      lib/mongo.ts         # MongoDB connection
      middleware/auth.ts   # Bearer-token middleware
      routes/auth.ts       # Register, login, current user
      routes/dashboard.ts  # Dashboard summary
      routes/resumes.ts    # Resume upload, extraction, and ATS scoring
  skillbridge-ai/
    src/
      pages/login.tsx
      pages/register.tsx
      pages/dashboard.tsx
      App.tsx
lib/
  api-spec/openapi.yaml    # Source of truth for API contracts
  api-client-react/        # Generated React Query client
  api-zod/                 # Generated request/response validators
```

## Step-by-step local setup

### 1. Install prerequisites

- Node.js 20 or newer
- pnpm 10 or newer
- A MongoDB database:
  - local MongoDB running on your machine, or
  - a MongoDB Atlas cluster

### 2. Install dependencies

```bash
pnpm install
```

### 3. Configure environment variables

Set these in the Replit Secrets/environment settings, or export them in your shell:

```bash
export MONGODB_URI="mongodb://127.0.0.1:27017/skillbridge_ai"
export MONGODB_DB_NAME="skillbridge_ai"
export SESSION_SECRET="use-a-long-random-secret"
```

For MongoDB Atlas, use the connection string from Atlas instead of the local URI. Do not commit the real connection string or session secret.

If MongoDB is not available yet, run the development auth store instead:

```bash
export AUTH_STORAGE="local"
export SESSION_SECRET="use-a-long-random-secret"
```

The local mode stores bcrypt-hashed users in `artifacts/api-server/data/users.json` and local resume analysis records/files in the same ignored data directory. It is useful for development and preview only and does not replace MongoDB for production.

### 4. Generate API clients after contract changes

```bash
pnpm --filter @workspace/api-spec run codegen
```

The OpenAPI file is `lib/api-spec/openapi.yaml`. The generated files should not be edited by hand.

### 5. Run the services

#### Option A: Run everything with a single command (Recommended)

In your terminal at the project root:

```bash
pnpm dev
```

This starts both:
- **API Server** on `http://localhost:3001`
- **Frontend Web App** on `http://localhost:3002`

#### Option B: Run services in separate terminals

**Terminal 1 (Backend API):**
```bash
pnpm dev:api
```
*(or `pnpm --filter @workspace/api-server run dev`)*

**Terminal 2 (Frontend Web):**
```bash
pnpm dev:web
```
*(or `pnpm --filter @workspace/skillbridge-ai run dev`)*

> [!NOTE]
> Environment variables (`AUTH_STORAGE=local`, `SESSION_SECRET`, etc.) are pre-configured in `.env` and automatically loaded.

### 6. Open the app

Open your browser to:
**[http://localhost:3002](http://localhost:3002)**

Health check:
```bash
curl http://localhost:3001/api/healthz
# or via frontend proxy:
curl http://localhost:3002/api/healthz
```

## API endpoints in Module 1

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/healthz` | No | Service health |
| POST | `/api/auth/register` | No | Create a user |
| POST | `/api/auth/login` | No | Get a JWT |
| GET | `/api/auth/me` | Bearer JWT | Get the signed-in user |
| GET | `/api/dashboard/summary` | Bearer JWT | Get dashboard metrics |
| POST | `/api/resumes` | Bearer JWT | Upload a PDF or DOCX resume and receive its initial ATS score |

The frontend stores the JWT in `localStorage` under `skillbridge_token` for this first module and automatically attaches it to API requests.

## Verification commands

```bash
pnpm run typecheck:libs
pnpm --filter @workspace/api-server run typecheck
pnpm --filter @workspace/skillbridge-ai run typecheck
pnpm --filter @workspace/api-server run build
PORT=24753 BASE_PATH=/ pnpm --filter @workspace/skillbridge-ai run build
```

## Common errors

### `Registration service is unavailable`

For development, set `AUTH_STORAGE=local` and restart the API. For MongoDB-backed use, check the connection string, make sure the local MongoDB process or Atlas network access is available, and restart the API.

### `SESSION_SECRET is not configured`

Add a long random value as the `SESSION_SECRET` secret. The API intentionally fails clearly instead of creating insecure tokens.

### MongoDB Atlas connection timeout

In MongoDB Atlas, open **Security → Network Access → Add IP Address** and allow the public egress IP of the environment running the API. The current development egress IP observed during setup is:

```text
35.200.169.21/32
```

This address can change between environments or sessions, so use the current egress address shown by your runtime when needed. Then confirm the database user has access to the selected database and copy the connection string exactly. For a short-lived local test only, Atlas also supports allowing `0.0.0.0/0`, but a specific IP is safer.

### API returns 401 on the dashboard

The browser has no valid `skillbridge_token`, or the token has expired. Sign in again.

## Next modules

The next build can add resume upload and parsing, followed by skill extraction, ATS scoring, job matching, career roadmaps, interview preparation, and AI-powered improvement.