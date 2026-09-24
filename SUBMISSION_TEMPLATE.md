# NodeWave Delivery OS Assessment

## Live links

- Frontend: https://nodewave-delivery-frontend.vercel.app
- Backend API: https://nodewave-delivery-api-production-7af8.up.railway.app
- Backend repository: https://github.com/RaihanIlhamHabibi/nodewave-delivery-backend
- Frontend repository: https://github.com/RaihanIlhamHabibi/nodewave-delivery-frontend
- Recording script: [RECORDING_SCRIPT.md](RECORDING_SCRIPT.md)
- Screenshots: [Auth](evidence-auth.png), [PM board](evidence-pm-board.png), [Blocked task](evidence-blocked-task.png), [Client view](evidence-client-view.png)

## Seeded accounts

All accounts use password `password`.

| Role | Email |
|---|---|
| Product Manager | pm@nodewave.test |
| UI/UX | ui@nodewave.test |
| Frontend | frontend@nodewave.test |
| Client Guest | client@atlas.test |

## Architecture

The Hono API is the policy enforcement point. JWT identifies the user, then project membership, role, department, task assignee, current status, and dependency status are checked before every mutation. Client responses are projected to client-visible tasks and do not include assignees or internal audit history.

The API includes register/login/logout, project task creation, search/filter/pagination (`page`, `pageSize`, `search`, `status`, `department`), dependency creation, status transitions, attachment metadata, internal comments, audit history, and soft-delete archive.

Task mutations use an integer `version` in the request and an atomic `updateMany` predicate. A stale version returns `409 Conflict`; every successful mutation writes an immutable AuditLog row in the same transaction.

Tasks are soft-delete ready through `deletedAt`. Dependencies are explicit rows in `TaskDependency`; a task cannot enter `IN_PROGRESS` until every dependency is `DONE`. The UI mirrors blocked state for discoverability, while the API remains authoritative.

## Demo flow

1. Sign in as PM and show the full board plus completion metrics.
2. Sign in as Client Guest and show that only client-visible tasks and aggregate-safe data are returned.
3. Sign in as Frontend and show the blocked task with its dependency lock.
4. Use the API contract to demonstrate stale-version rejection with `409`.

## Verification

- Backend: `bun run typecheck`, `bun test`
- Frontend: `npm run typecheck`, `npm run build`
- Database: Prisma migrations and seed applied through Docker PostgreSQL.
- Live smoke test: all four seeded accounts authenticate; project overview returns 4 tasks and 50% completion.
- Optional standup endpoint: `GET /projects/:projectId/standup` for PM/internal users.

## Run locally

```powershell
docker compose up -d
Set-Location backend
Copy-Item .env.example .env
bun install
bunx prisma migrate dev --name init
bun run db:seed
bun run dev

Set-Location ..\frontend
Copy-Item .env.local.example .env.local
npm install
npm run dev
```
