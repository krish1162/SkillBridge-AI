# SkillBridge AI

SkillBridge AI helps students and job seekers turn resumes and experience into actionable career signals.

## Run & Operate

- `pnpm dev` — run both backend API server and frontend web app in parallel
- `pnpm dev:api` (or `pnpm --filter @workspace/api-server run dev`) — run the API server (port 3001)
- `pnpm dev:web` (or `pnpm --filter @workspace/skillbridge-ai run dev`) — run the web app (port 3002)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- Required secret: `SESSION_SECRET`
- Production secret: `MONGODB_URI`
- Development option: `AUTH_STORAGE=local` when MongoDB is not available
- Optional env: `MONGODB_DB_NAME` (defaults to `skillbridge_ai`)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5
- DB: MongoDB Node.js driver
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/skillbridge-ai` — React/Vite web app
- `artifacts/api-server` — API routes, JWT auth, MongoDB connection
- `lib/api-spec/openapi.yaml` — source of truth for API contracts
- `lib/api-client-react` — generated React Query client
- `lib/api-zod` — generated server validators

## Architecture decisions

- Module 1 uses the requested JWT + bcrypt authentication rather than a third-party identity provider.
- MongoDB is accessed through a lazy shared connection so the API can start and health-check even before a database request is made.
- OpenAPI is the source of truth; React hooks and Zod validators are generated from it.
- The frontend stores the first-module JWT in `localStorage` and configures the shared fetch client to attach it.

## Product

Module 1 provides registration, login, session validation, and a dashboard foundation for resume, ATS, job-match, and roadmap signals.

## User preferences

- User requested a non-Docker setup.

## Gotchas

- Set `MONGODB_URI` and `SESSION_SECRET` before testing registration or login.
- For a database-free development preview, set `AUTH_STORAGE=local`; users and analyzed resume records are persisted in the ignored local data directory.
- MongoDB Atlas must allow the API runtime's public egress IP in Network Access.
- Run API codegen after changing `lib/api-spec/openapi.yaml`.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
