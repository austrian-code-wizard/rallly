# Rallly Local Development Setup & Testing

## Overview
Rallly is a Next.js 16 meeting scheduling app using a monorepo with Turbo, Prisma, PostgreSQL, Redis, MinIO, and Mailpit.

## Prerequisites
- Node.js 24+ (use nvm)
- pnpm 10.28.0+
- Docker & Docker Compose

## Setup Steps

1. **Install dependencies**
   ```bash
   pnpm install
   pnpm approve-builds  # approve all packages when prompted
   ```

2. **Configure environment**
   ```bash
   cp .env.development .env
   cp .env apps/web/.env  # IMPORTANT: Next.js needs .env in its own directory
   ```
   > **Gotcha**: Even though `turbo.json` lists `.env` as a globalDependency, Next.js in `apps/web/` won't load the root `.env`. You must copy it to `apps/web/.env` or the app will crash with "Invalid environment variables" errors.

3. **Generate Prisma client**
   ```bash
   pnpm db:generate
   ```

4. **Start Docker services** (PostgreSQL, Redis, MinIO, Mailpit)
   ```bash
   pnpm docker:up  # uses docker-compose.dev.yml
   ```
   Services:
   - PostgreSQL: `localhost:5450` (user: postgres, pass: postgres, db: rallly)
   - Mailpit SMTP: `localhost:1025`
   - Mailpit UI: `http://localhost:8025`
   - Redis: `localhost:6379`
   - MinIO: `localhost:9000`

5. **Reset and seed database**
   ```bash
   pnpm db:reset  # runs migrations and resets
   pnpm db:seed   # seeds test data
   ```

6. **Start dev server**
   ```bash
   pnpm dev
   ```
   App available at `http://localhost:3000`

## Testing the App

### Login Flow (Email OTP)
1. Navigate to `http://localhost:3000` (redirects to `/login`)
2. Enter a seeded user email (e.g., `dev@rallly.co`)
3. Click "Continue with email" — wait for redirect to `/login/verify`
4. Open Mailpit at `http://localhost:8025` to find the 6-digit OTP code
5. Enter the OTP on the verify page
6. You'll be redirected to the dashboard

### Seeded Test Users
- `dev@rallly.co` (Dev User) — primary test account
- `sarah@rallly.co` (Sarah Chen)
- `michael@rallly.co` (Michael Torres)
- `emily@rallly.co` (Emily Nakamura)
- `james@rallly.co` (James Okonkwo)

### Seeded Polls
The seed data creates 6 open polls with participants, votes, and comments:
- Birthday party venue (8 participants)
- Photography class (3 participants)
- Book club meeting (6 participants)
- Weekend hike (5 participants)
- Dentist appointment (1 participant)
- Coffee chat with Alex (2 participants)

### Key Pages to Verify
- `/` — Dashboard (Home) with poll counts
- `/polls` — List of polls with Open/Closed/Scheduled tabs
- `/poll/{id}` — Poll detail with participants, votes, comments
- `/new` — Create new poll
- `/events` — Events list
- `/settings/preferences` — User preferences

## Known Issues
- A "Timezone Change Detected" dialog may appear on first login if the server timezone differs from the seeded user's timezone. Dismiss it.
- Updating timezone preference may show an "internal server error" toast — this is a minor issue that doesn't affect core functionality.
- The SMTP certificate warning (`Certificate validation is now enabled by default`) appears in dev server logs — this is expected for local Mailpit.

## Devin Secrets Needed
No external secrets are required for local development. All credentials are in `.env.development`.
