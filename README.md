# Zec Bounties - Local Development Setup

Bounty Platform with Native ZEC Payments

The project consists of:

- **Frontend** – User interface built with Next.js.
- **Backend** – REST API for authentication, bounty management, submissions, and Zcash payments.

---

# Prerequisites

Before getting started, ensure you have the following installed:

- Node.js 22 LTS (the version CI uses). On Node.js 20, `npm install` in the backend has no prebuilt `better-sqlite3` binary and compiles it from source, which needs `make` and a C++ compiler.
- npm and/or Yarn
- PostgreSQL
- Redis
- Docker (optional, to run PostgreSQL and Redis; see below)
- Zebrad, Zaino and Zingo-cli (only for Zcash wallet and payment features; not needed to run the app locally)

For Zcash node setup, see the **ZecHub Developer Resources**:

https://zechub.wiki/developers

---

# Project Setup

## Backend

### 1. Navigate to the backend directory

```bash
cd zec-bounties-backend
```

### 2. Install dependencies

```bash
npm install
```

### 3. Start PostgreSQL and Redis

With Docker:

```bash
docker run -d --name zb-postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=zec_bounties -p 5432:5432 postgres:16-alpine
docker run -d --name zb-redis -p 6379:6379 redis:7
```

### 4. Create your `.env`

The repository does not ship an `.env.example`. Create `zec-bounties-backend/.env` with:

```bash
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/zec_bounties
JWT_SECRET=replace-with-a-long-random-string
PORT=9000
VAPID_PUBLIC_KEY=
VAPID_PRIVATE_KEY=
VAPID_EMAIL=mailto:you@example.com
```

Then generate the VAPID keys:

```bash
npx web-push generate-vapid-keys
```

Paste the generated keys into `VAPID_PUBLIC_KEY` and `VAPID_PRIVATE_KEY`, and set `JWT_SECRET`. See [Environment Variables](#environment-variables).

### 5. Create the database tables

```bash
npx prisma db push
```

### 6. Start the backend

```bash
npm run dev
```

The backend runs at:

```
http://localhost:9000
```

---

## Frontend

### 1. Navigate to the frontend directory

In a second terminal, from the repository root (the backend keeps running in the first):

```bash
cd zec-bounties-frontend
```

### 2. Install dependencies

```bash
yarn install
```

### 3. Start the development server

```bash
yarn dev
```

The frontend runs at:

```
http://localhost:3000
```

---

# Environment Variables

The backend reads its settings from `zec-bounties-backend/.env`.

For local development you only need:

- `DATABASE_URL`: PostgreSQL connection string (the example matches the Docker command above)
- `JWT_SECRET`: any long random string
- `PORT=9000`: matches the frontend's development backend URL in `zec-bounties-frontend/lib/configENV.ts`
- `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_EMAIL`: the backend won't start without them; generate the keys with `npx web-push generate-vapid-keys`

Redis is read from `REDIS_URL`, which defaults to `redis://localhost:6379` (the Docker command above), so it can be left out.

Everything else (GitHub and Discord login, email, Pinata uploads, Zcash wallets and payments) can stay unset until you work on that feature.

---

# Development

Start the backend:

```bash
cd zec-bounties-backend
npm run dev
```

Start the frontend:

```bash
cd zec-bounties-frontend
yarn dev
```

Once both services are running, open:

```
Frontend: http://localhost:3000
Backend:  http://localhost:9000
```

## NOTES & LIMITATIONS

- Database: PostgreSQL (see the Docker command in the backend setup). Redis is also required.
- To reset the database: `docker rm -f zb-postgres`, start it again with the same `docker run` command, and re-run `npx prisma db push`.
- Zcash integration (payments, shielded tx, etc.) requires Zebrad + Zaino running and correctly configured in .env.
  Without them, bounty creation/submission UI may work but actual ZEC transfers will fail.
- GitHub login is used for authentication. Set up a GitHub OAuth App at https://github.com/settings/developers
  and use the Client ID/Secret. For quick testing you can temporarily bypass auth in code if needed.
- Both parts have their own lockfiles (yarn.lock / package-lock.json). Do not mix package managers.
- Production deployment uses Vercel for frontend (see https://bounties.zechub.wiki/).
- For issues or updates, check the subfolder README.md files or open issues on the GitHub repo.
- This setup guide is derived from repo structure and sub-READMEs (as of July 2026). Always verify commands work in your environment.
- Verified on 11 October 2026 at commit `62ae6f6` on Debian GNU/Linux 13 (trixie) with Node.js 22.23.3, npm 10.9.9 and Yarn 1.22.22: backend on port 9000, frontend on port 3000, frontend loading data from the backend. PostgreSQL 17 and Redis 8 ran as local services instead of the Docker containers. GitHub login and Zcash payments were not tested.

## TROUBLESHOOTING

- Prisma errors: Ensure you ran `npx prisma generate` after any schema changes.
- Port conflicts: the backend port is `PORT` in `zec-bounties-backend/.env` (9000 by default in `.env.example`); the frontend expects it at the URL in `zec-bounties-frontend/lib/configENV.ts`. Keep the two in step.
- Yarn issues: Delete node_modules + yarn.lock and re-run `yarn install`.
- Missing env vars: Double-check .env file location (must be in backend root) and restart backend.
- Zcash RPC connection refused: Confirm Zebrad is running, RPC is enabled on the URL/port in .env, and credentials match.

For the most up-to-date info, visit the GitHub repo: https://github.com/ZecHub/zec-bounties
