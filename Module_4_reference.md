# Module 4 Reference Guide
### Claude Code — VibeMail Project Backend: Prompts, Commands, and Verification

Use this guide alongside the Module 4 videos.

---

## Table of Contents

1. [Project: VibeMail Backend - Architecture and Planning](#project-vibemail-backend---architecture-and-planning)
2. [Setting Up Google Cloud](#setting-up-google-cloud)
3. [Repository Creation & CLAUDE.md](#repository-creation--claudemd)
4. [Worktree Architecture: Parallel Sessions](#worktree-architecture-parallel-sessions)
5. [Database Planning](#database-planning)
6. [Building the Core Engine](#building-the-core-engine)
7. [Writing Out Our Server Based Logic](#writing-out-our-server-based-logic)
8. [Combining the Server and Schema Sessions](#combining-the-server-and-schema-sessions)
9. [Deploying and Configuring Production Infrastructure](#deploying-and-configuring-production-infrastructure)
10. [Verifying What We Built](#verifying-what-we-built)
11. [Running Final Tests](#running-final-tests)

---

## Core Concepts

**VibeMail is a data-liberation and synchronization layer.** Gmail is the teaching vehicle; the reusable pattern is Extract → Structure → Embed.

**The contract is the synchronization point.** It defines endpoint shapes, errors, the message model, acceptance criteria, and the sequencing rule shared by both sessions.

**Logic ships before schema merges.** The server session establishes what storage actually needs. The schema session plans in parallel but does not merge until the Session 1 verification gate passes.

**OAuth tokens are production data.** Encrypt them, persist refreshed values, never hardcode credentials, and use the official OAuth2 client for refresh behaviour.

**Push, do not poll.** Gmail changes arrive through Pub/Sub. The webhook acknowledges first, then processes the delta from the stored history ID.

**Every unit ends in proof.** TypeScript, tests, diff scope, and live production behaviour are separate gates.

---

## Project: VibeMail Backend - Architecture and Planning

### CONTRACT.md prompt

Enter Plan Mode with `Shift+Tab`, then dictate:

> "I need to write a CONTRACT.md for the VibeMail Engine. Before producing anything, ask me clarifying questions one at a time until you are 95% confident you have everything you need. The contract should cover: the acceptance criteria that define when the project is complete, the API endpoint contracts for the four core endpoints — OAuth callback, message list, message send, and mark as read — with request shape, response shape, and typed error cases for each. Also include the data model for a stored message: every field, its type, and its source in the Gmail API response. And include the sequencing rule for the two-session build: server logic ships and tests pass before schema is reviewed or merged."

### Acceptance criteria to preserve

- Gmail OAuth completes and encrypted tokens persist in Supabase.
- First authentication syncs the 50 most recent messages.
- New messages arrive through Pub/Sub without polling.
- An authenticated user can send through Gmail.
- Read state synchronizes in both directions.
- Authentication failures return typed recoverable errors.
- Tests pass with no skipped tests.
- The backend is live on Vercel.
- Git history shows clean sequential units.

### BUILD_SEQUENCE.md prompt

Exit Plan Mode, then dictate:

> "Create a file called BUILD_SEQUENCE.md. I'm going to dictate the atomic build sequence for the VibeMail Engine. For each unit, I'll describe what it is and what verified looks like — one sentence per unit describing what I'd check to confirm it's correct before moving to the next."

Dictate these units in order:

1. Provider abstraction interface — TypeScript compiles clean and the interface defines all required methods.
2. Gmail OAuth layer with token persistence listener — OAuth completes, tokens are stored, refresh works without mismatch.
3. Sync and read layer — initial sync fetches 50 messages and objects match the contract model.
4. Pub/Sub webhook receiver — a notification fetches the correct delta through history ID.
5. Send layer — a message sends successfully through Gmail for an authenticated user.
6. Vercel API function entry points — all four endpoints respond correctly in local preview.
7. Integration tests — the full suite passes with no skipped tests against live Supabase.

---

## Setting Up Google Cloud

### Console setup

1. Create a Google Cloud project named `vibemail`.
2. Enable **Gmail API** and **Cloud Pub/Sub API**.
3. Configure the OAuth consent screen as External and keep the app in Testing mode.
4. Add your Gmail address as a test user.
5. Add the scope `https://www.googleapis.com/auth/gmail.modify`.
6. Create a Web application OAuth client.
7. Add JavaScript origin `http://localhost:3000`.
8. Add redirect URI `http://localhost:3000/api/v1/auth/google/callback`.
9. Download the credentials JSON, but never commit it.
10. Create topic `vibemail-gmail-notifications`.
11. Record `projects/your-project-id/topics/vibemail-gmail-notifications`.
12. Grant `gmail-api-push@system.gserviceaccount.com` the Pub/Sub Publisher role on the topic.

### Values needed for the repository

- `GOOGLE_CLIENT_ID`
- `GOOGLE_CLIENT_SECRET`
- `GOOGLE_REDIRECT_URI`
- `GOOGLE_PUBSUB_TOPIC`

---

## Repository Creation & CLAUDE.md

### Repository and project skeleton

```bash
gh repo create vibemail-engine --private
git clone https://github.com/your-username/vibemail-engine
cd vibemail-engine
```

```bash
mkdir -p src/providers/gmail src/providers/imap src/sync src/webhook src/send src/db src/types src/middleware src/cron
mkdir -p api/v1/auth/google api/v1/messages api/webhook api/cron
mkdir -p tests/unit tests/integration docs .claude/commands
```

```bash
npm install typescript ts-node @types/node googleapis @supabase/supabase-js jsonwebtoken @types/jsonwebtoken
npm install --save-dev jest ts-jest @types/jest supertest @types/supertest vercel
```

### tsconfig.json

Create manually:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*", "api/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Environment template

`.env.example` is committed with empty values; `.env` is local and ignored.

```dotenv
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=http://localhost:3000/api/v1/auth/google/callback
GOOGLE_PUBSUB_TOPIC=projects/your-project-id/topics/vibemail-gmail-notifications
GOOGLE_PUBSUB_VERIFICATION_TOKEN=
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=
JWT_SECRET=
ENCRYPTION_KEY=
FRONTEND_URL=http://localhost:3001
```

```bash
cp .env.example .env
git add .
git commit -m "init: project skeleton and dependencies"
git push -u origin main
```

Run `/init`, then refine `CLAUDE.md`:

> "Revise the CLAUDE.md. Add the following sections: What this project is — a data liberation sync engine that extracts Gmail data via OAuth2 into Supabase, exposes a REST API at /api/v1, deployed to Vercel serverless functions. Stack — Node 20, TypeScript strict mode, googleapis, Supabase JS client, Jest with supertest, Vercel. The two-session architecture — server logic session on main branch owns src/ and api/, schema session on schema branch owns migrations/ and types/. Sequencing rule — schema session cannot be merged until server logic tests pass. Never-do rules: never use any as a TypeScript type, never poll the Gmail API for new messages — use Pub/Sub push webhooks, never store OAuth tokens in plaintext, never make Gmail API calls without using the googleapis OAuth2 client which handles token refresh automatically, never write to src/db/ from the schema session, never merge the schema session before npm test exits 0, never hardcode credentials. Coding conventions: all errors use the CONTRACT.md error envelope shape, cursor-based pagination on all list endpoints, /api/v1 base path on all client endpoints, JWT Bearer token authentication on all endpoints except the OAuth callback, the Pub/Sub webhook endpoint lives at /webhook/gmail not under /api/v1."

```bash
git add CLAUDE.md tsconfig.json .env.example .gitignore
git commit -m "setup: CLAUDE.md, tsconfig, env structure"
git push
```

Context verification prompt:

> "What project is this, what are the two sessions responsible for, and what is the sequencing rule?"

Configure the deploy-verification API routine and store its endpoint URL and token securely for deployment.

---

## Worktree Architecture: Parallel Sessions

### Copy the local environment into app-created worktrees

```bash
echo ".env" > .worktreeinclude
git add .worktreeinclude
git commit -m "setup: copy .env into new worktrees automatically"
git push
```

### Create the schema branch

```bash
git checkout -b schema
git push -u origin schema
git checkout main
```

Create two desktop sessions:

- `VibeMail — Server Logic` on `main`
- `VibeMail — Schema` on `schema`

### Isolation test

Server session:

```bash
echo "server logic test" > isolation-test.txt
```

Schema session:

```bash
ls isolation-test.txt
# Expected: No such file or directory
```

Clean up in the server session:

```bash
rm isolation-test.txt
```

### Install and verify both worktrees

Run in each session:

```bash
npm install
npx tsc --noEmit
```

The initial `TS18003: No inputs were found` is expected while TypeScript source folders are empty.

Schema session:

```bash
ls .env
```

If it is missing:

```bash
cp ../../.env .env
```

Ask in each session:

> "What project is this, what is your session responsible for, and what is the sequencing rule?"

---

## Database Planning

### Supabase setup

Create `vibemail-engine` at Supabase, then add these local `.env` values:

- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY` — not the anon key

```bash
npm install -g supabase
supabase --version
```

Add package scripts manually:

```json
{
  "scripts": {
    "db:push": "supabase db push",
    "db:types": "supabase gen types typescript --linked > types/database.ts",
    "build": "npx tsc",
    "test": "jest --runInBand",
    "dev": "ts-node src/index.ts"
  }
}
```

```bash
git add package.json
git commit -m "setup: Supabase CLI scripts"
git push origin main
```

Schema session:

```bash
supabase login
supabase link --project-ref your-project-ref
supabase status
```

### Schema Plan Mode prompt

> "Ask me clarifying questions one at a time until you're 95% confident you understand what the Supabase schema needs to contain, then produce the plan. Read CONTRACT.md and BUILD_SEQUENCE.md first — they define every field, every relationship, and every constraint the schema must satisfy. The schema must cover: users table with OAuth state and watch metadata, messages table matching CONTRACT.md §3 exactly, and any support structures required. RLS policies must enforce that users can only access their own data. All migrations must be idempotent."

```bash
git add docs/schema-plan.md
git commit -m "plan: Supabase schema design"
git push origin schema
```

Do not build or apply migrations yet. The schema session remains behind the server-logic gate.

---

## Building the Core Engine

### Create `/verify-unit`

```bash
touch .claude/commands/verify-unit.md
```

> "Create the .claude/commands/verify-unit.md file. The command should run the following verification checks and report only — it must not fix anything. Check 1: run npx tsc --noEmit and report whether it exits 0 or list the type errors. Check 2: run npm test and report the pass count, fail count, and any failure messages. Check 3: run git diff --name-only and list which files have been modified since the last commit — flag any files outside the expected scope for the current build unit. Report UNIT VERIFIED if all three pass with no unexpected file modifications, or UNIT FAILED with specific failures if any check fails."

```bash
git add .claude/
git commit -m "setup: verify-unit slash command"
git push
```

### Confirm current library contracts

> "Before we start building, use Context7 to pull the current documentation for the googleapis npm package — specifically the OAuth2 client, the Gmail API users.messages and users.history endpoints, and users.watch. Let me know if anything in the current documentation differs from what CONTRACT.md assumes."

### Unit 1 — Provider abstraction

> "Ask clarifying questions until 95% confident, then plan the provider abstraction interface. It lives in src/types/provider.ts and defines the TypeScript interface that all email provider implementations must satisfy. The interface must include all methods required by CONTRACT.md: list messages, send a message, mark a message read or unread, initiate the OAuth flow, exchange an authorization code for tokens, and refresh an access token. No Gmail-specific types leak into this interface — it must be provider-agnostic."

### Unit 2 — Gmail OAuth

> "Build the Gmail OAuth layer in src/providers/gmail/auth.ts. It must: use the googleapis OAuth2 client to initiate the authorization flow and generate the consent URL, handle the OAuth callback by exchanging the authorization code for tokens, encrypt the access token and refresh token using AES-256-GCM before persisting them to the users table in Supabase using the service role key — upsert on conflict google_id, implement the token persistence listener — when the googleapis client auto-refreshes an access token, decrypt the new token, re-encrypt it, and write it back to Supabase immediately via updateUserTokens, implement token refresh that fetches the stored encrypted refresh token from Supabase, decrypts it, and uses it to obtain a new access token, and after successful token storage call users.watch with the Pub/Sub topic from GOOGLE_PUBSUB_TOPIC and store the watch_expiry timestamp, watch_resource_id, and initial history_id to the users table."

### Unit 3 — Sync and read

> "Build the initial sync in src/sync/index.ts. It must: call messages.list with labelIds INBOX and pageToken for pagination, call messages.get for each message ID with format FULL, normalise the Gmail API response to the Message shape — extracting from, to, subject, and date from the headers array by name (case-insensitive), note that from and to must be stored as from_address and to_address to match the database schema, base64url-decoding body_plain and body_html from the parts array by mimeType, for single-part messages with no parts array falling back to message.payload.body.data, and deriving is_read from !labelIds.includes('UNREAD') and is_starred from labelIds.includes('STARRED'), upsert each normalised message to Supabase using gmail_id as the conflict target, stop after 50 messages for the initial sync, and store the historyId from the last messages.list response to users.history_id."

### Unit 4 — Pub/Sub webhook

> "Build the Pub/Sub webhook receiver in src/webhook/gmail.ts. It must: receive a Pub/Sub push payload as an HTTP POST, return HTTP 200 immediately before doing any further processing — this acknowledges the notification to Pub/Sub before work begins, decode the base64-encoded data field from the Pub/Sub message to extract emailAddress and historyId, fetch the user's current history_id from Supabase for the user matching emailAddress, call history.list with the stored history_id as the startHistoryId parameter to get only the delta, normalise any new or modified messages from the history response and upsert them to Supabase, and update users.history_id to the historyId from the current notification."

### Unit 5 — Send layer

> "Build the send layer in src/send/index.ts. It must: accept to, subject, body, and optional threadId, construct an RFC 2822 formatted message encoded as base64url, call messages.send via the Gmail API on behalf of the authenticated user, normalise the sent message response — mapping the response fields to the database shape including from_address and to_address, and upsert it to Supabase."

After every unit:

```text
/verify-unit
```

```bash
git add .
git commit -m "feat: [unit description]"
git push
```

---

## Writing Out Our Server Based Logic

### Unit 6 — Vercel API functions

> "Wire all four route handlers from CONTRACT.md §4 as Vercel Serverless Functions at: api/v1/auth/google/callback.ts for GET /api/v1/auth/google/callback, api/v1/messages/index.ts for GET /api/v1/messages, api/v1/messages/send.ts for POST /api/v1/messages, api/v1/messages/[id]/read.ts for PATCH /api/v1/messages/:id. Also create api/webhook/gmail.ts for the Pub/Sub webhook receiver — this is not under /api/v1 because it is a Google-to-server callback, not a client-facing endpoint. Apply JWT middleware to the three authenticated routes. Every error response must use the error envelope from CONTRACT.md: { error: { code, message, details? } }."

### Postman collection

> "Read CONTRACT.md §4 and generate a Postman collection JSON file at docs/vibemail-api.postman_collection.json. The collection should include one folder per endpoint. For each endpoint include: a happy path request with the correct method, URL using {{baseUrl}} as a variable, required headers including Authorization: Bearer {{jwt}} for authenticated endpoints, and a sample request body where applicable. Also include one request per named error code from CONTRACT.md §4 — for example a request with no Authorization header to test UNAUTHORIZED, a request with an invalid body to test MISSING_FIELDS, and so on. Set the collection variable baseUrl to http://localhost:3000 as the default value."

### Unit 7 — Watch renewal cron

> "Add a crons array to the existing vercel.json with a single entry: path set to /api/cron/renew-watch and schedule set to 0 6 * * *. Then build the watch renewal cron job in src/cron/renewWatch.ts — it must query Supabase for all users whose watch_expiry is within 24 hours of the current time, call users.watch for each user with the Pub/Sub topic from GOOGLE_PUBSUB_TOPIC, and update watch_expiry and watch_resource_id with the new values returned by the watch call. Wire it as a Vercel Serverless Function at api/cron/renew-watch.ts."

Create manually:

```json
{
  "crons": [
    {
      "path": "/api/cron/renew-watch",
      "schedule": "0 6 * * *"
    }
  ]
}
```

### Unit 8 — Integration tests

Use a strong model to specify the suite:

> "Write the full Jest integration test suite covering every endpoint contract and every named error code in CONTRACT.md §4. Tests run against a live Supabase instance — do not mock Supabase. Gmail API calls may be mocked. Test categories: message normalisation unit tests covering base64url decoding, header extraction by name, from_address and to_address mapping, and boolean derivation from labelIds, OAuth unit tests covering token encryption, persistence, and the token persistence listener firing correctly, webhook receiver tests covering the acknowledge-first pattern and delta fetch using history_id, and endpoint integration tests covering every HTTP status code and every error.code value in CONTRACT.md §4."

Use a smaller model for the mechanical implementation pass, then run `/verify-unit`.

```bash
git add .
git commit -m "test: full integration suite passing, Session 1 gate cleared"
git push origin main
```

---

## Combining the Server and Schema Sessions

### Open and verify the schema PR

```bash
git push origin schema
gh pr create --title "feat: Supabase schema for VibeMail Engine" --body "Session 1 gate cleared — npm test exits 0 on main."
```

### Apply schema and generate types

```bash
supabase link --project-ref your-project-ref
npm run db:push
npm run db:types
```

If the migration history says up-to-date while tables are missing, inspect and repair the specific migration state before retrying.

```bash
git add migrations/ types/
git commit -m "feat: schema migration applied and types verified"
git push origin schema
```

After the PR passes and is merged, pull `main` in the server session:

```bash
git pull origin main
```

### Database layer prompt

> "Build the database layer in src/db/index.ts using the Supabase JS client with the service role key for all write operations. Use the TypeScript types from types/database.ts as the row shapes. Implement: upsertMessage that upserts to the messages table using gmail_id as the conflict target — map from and to to from_address and to_address, getUser that fetches a user row by userId, updateUserTokens that writes new encrypted access token, refresh token, and expiry — called by the token persistence listener, updateHistoryId that writes the new historyId to users.history_id after each sync, and updateWatchExpiry that writes the new expiry timestamp and watch_resource_id after a watch renewal."

### Sync wiring prompt

> "Update src/sync/index.ts to call the database layer functions instead of writing to Supabase directly. Normalised message objects flow through upsertMessage — map from and to to from_address and to_address. The historyId flows through updateHistoryId."

```bash
npm test
git add src/db/ src/sync/ src/providers/
git commit -m "feat: database layer, sync wiring, full integration tests passing"
git push origin main
```

---

## Deploying and Configuring Production Infrastructure

### Deploy

```bash
vercel --prod
```

Generate secrets locally:

```bash
openssl rand -hex 32  # JWT_SECRET
openssl rand -hex 32  # ENCRYPTION_KEY
openssl rand -hex 32  # GOOGLE_PUBSUB_VERIFICATION_TOKEN
```

Import the local environment into Vercel, then verify:

- `GOOGLE_REDIRECT_URI=https://your-domain/api/v1/auth/google/callback`
- `FRONTEND_URL=https://your-domain` temporarily
- `SUPABASE_SERVICE_ROLE_KEY` is the service-role key
- Every secret is configured for Production

Redeploy after configuration:

```bash
vercel --prod
```

### Update Google Cloud production settings

- Add the production OAuth redirect URI.
- Create a Pub/Sub push subscription for `vibemail-gmail-notifications`.
- Set endpoint to `https://your-domain/webhook/gmail`.
- Set acknowledgement deadline to 60 seconds.

### Verify the cron endpoint

```bash
curl -X GET https://your-domain/api/cron/renew-watch \
  -H "Authorization: Bearer $CRON_SECRET"
```

### OAuth initiation endpoint fix, only if the production check returns 404

> "Create api/v1/auth/google/index.ts — a GET handler for /api/v1/auth/google that calls initiateOAuth from src/providers/gmail/auth.ts, gets back the url and state, and redirects the user to the url with a 302. Apply the same error handling pattern as the callback — METHOD_NOT_ALLOWED for non-GET requests, and a 500 CONFIG_ERROR if initiateOAuth throws a configuration error."

If live OAuth reveals a profile-fetch failure, use the current corrective prompt:

> "In the OAuth callback handler in src/providers/gmail/auth.ts, fix the profile fetch failure. Add `https://www.googleapis.com/auth/userinfo.email` and `https://www.googleapis.com/auth/userinfo.profile` to the scopes array in `initiateOAuth`. Remove any separate API call to fetch the user profile. After the token exchange, call `oauth2Client.getTokenInfo(tokens.access_token)` to get the user's Google ID, email, and name from the token response — extract `sub`, `email`, and `name` from the result. Use `sub` as the Google ID for the Supabase upsert. Store `email` and `name` in the users table — add those columns if they don't exist in the schema."

Redeploy and rerun the failed production check after any fix.

---

## Verifying What We Built

Update Postman collection variables:

- `baseUrl` — the exact production origin
- `jwt` — blank until OAuth succeeds

### Live acceptance checks

**OAuth**

Complete the browser OAuth flow, confirm the production callback succeeds, and copy the JWT from the redirect into the Postman `jwt` variable.

**Message list and pagination**

Run `GET /api/v1/messages`. Pass the returned `nextCursor` back as `cursor` and confirm the next page is correct.

**Send**

Run `POST /api/v1/messages` and confirm the real email arrives in Gmail.

**Read state**

Run `PATCH /api/v1/messages/:id` with both `{ "read": true }` and `{ "read": false }`. Confirm the state changes in Gmail as well as VibeMail.

**Typed errors**

Run every named error request in the collection. Confirm status codes, exact `error.code` values, and this envelope:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {}
  }
}
```

Do not treat a successful deployment command as proof. These checks exercise the live routes, credentials, OAuth configuration, Gmail delivery, and data synchronization.

---

## Running Final Tests

### Test suite

```bash
npm test
```

Require no failed or skipped tests.

### TypeScript

```bash
npx tsc --noEmit
```

### Credential scan

```bash
grep -r "sk-\|SUPABASE\|google\|secret" src/ api/ --include="*.ts" -i
```

Review every match. Environment variable names are expected; literal secrets are not.

### Idempotent migration

Run the full initial migration a second time in Supabase SQL Editor and confirm it completes without error or destructive duplication.

### Final acceptance checklist

- OAuth works against production.
- Initial and incremental sync satisfy the contract.
- Pub/Sub delivery is configured and acknowledged correctly.
- Send and mark-as-read work against a real Gmail account.
- Error cases use the exact contract envelope.
- `npm test` passes with no skipped tests.
- `npx tsc --noEmit` exits 0.
- No credentials are hardcoded.
- Migrations are idempotent.
- The production URL responds on every documented route.

### Final commit

```bash
git add .
git commit -m "deploy: VibeMail Engine live on Vercel, all AC criteria verified"
git push origin main
```
