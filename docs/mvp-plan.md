# Tracework — MVP Implementation Plan

Follows the implementation order in `tracework-mvp.md`. Tick each box when done.
Folder names match `project-structure.md`.

## Phase 0 — Project setup

### Workspaces and tooling

- [x] Create `shared/`, `client/`, `server/` folders, each with its own `package.json`
- [ ] Add root `tsconfig.base.json`; extend it in each workspace
- [ ] Set up Oxlint (and optionally Oxfmt) at the root
- [ ] Set up Vitest in `client/`, `server/`, `shared/`
- [ ] Add root scripts: `dev`, `build`, `lint`, `test`
- [ ] Fill in `.env.example` with all required variable names (no values)

### Shared package

- [ ] Set up `shared/src/index.ts` as the package entry
- [ ] Make `shared` importable from `client` and `server`

### Client skeleton

- [x] Scaffold React + TypeScript with Vite in `client/`
- [ ] Install and configure Tailwind CSS
- [ ] Initialize shadcn/ui (`components/ui/`)
- [ ] Add React Router with an empty page per route
- [ ] Check the app runs in the browser

### Server skeleton

- [ ] Set up Express + TypeScript in `server/` (`app.ts`, `index.ts`)
- [ ] Add a `GET /health` route
- [ ] Add a central error-handling middleware
- [ ] Add a dev script with auto-reload
- [ ] Check the client can call `/health` (Vite proxy or CORS)

### Supabase

- [ ] Create the staging Supabase project
- [ ] Decide what local dev uses: the staging project or local Supabase with the CLI; note it in `tracework-mvp.md`
- [ ] Create `supabase/migrations/` and `supabase/seed.sql`
- [ ] Add the Supabase client in `server/src/db/` and `client/src/lib/`

## Phase 1 — Auth, routes, profile, applications table

### Database

- [ ] Migration: `user_setting` (theme, time zone, No Answer threshold, default 14)
- [ ] Migration: `user_pipeline_status` (stable ID, label, order, kind: active / closed)
- [ ] Migration: `application` (company, position, applied date, status ID, URL, source, missing requirements, notes)
- [ ] Add row-level security policies for each table (owner only)
- [ ] Create default settings and default pipeline statuses on sign-up (trigger or server logic)

### Shared types

- [ ] Define default status IDs and closing statuses (Rejected, Withdrawn)
- [ ] Define `Application` DTO and create/update input types
- [ ] Define API request/response contracts for applications

### Auth (client)

- [ ] `/signup` page with email and password
- [ ] `/login` page
- [ ] `/reset-password` page (request link and set new password)
- [ ] Auth context/provider holding the Supabase session
- [ ] Auth guard that redirects signed-out users to `/login`
- [ ] Sign-out action

### Password reset email (Resend)

- [ ] Verify `tracework.surf` (or a sending subdomain of it) in Resend; without it, Resend only sends to your own address
- [ ] Create a Resend API key for staging
- [ ] Configure custom SMTP with Resend in the staging Supabase project
- [ ] Set Site URL and Redirect URLs in staging Supabase to include `/reset-password` (local dev and staging frontend)
- [ ] Adjust the reset email template and sender name
- [ ] Check the full reset flow with a test account

### Auth (server)

- [ ] Middleware that verifies the Supabase access token from the `Authorization` header
- [ ] Attach the user ID to the request
- [ ] Ownership check helper for application routes
- [ ] API client in `client/src/lib/` that sends the token on every request

### Applications API

- [ ] `GET /applications` with search (company/position) and status filter
- [ ] `POST /applications` with validation of required fields
- [ ] `GET /applications/:id`
- [ ] `PATCH /applications/:id`
- [ ] `DELETE /applications/:id`
- [ ] Tests for validation and ownership

### Applications UI

- [ ] App layout with navigation
- [ ] Toolbar: search, status filter, Add application button
- [ ] Table with company, position, applied date, status, latest reply
- [ ] Add application form (dialog); applied date defaults to today in the user's time zone
- [ ] Edit form with all fields
- [ ] Details side panel on wide screens
- [ ] `/applications/:id` page for direct links and narrow screens
- [ ] Loading, empty, and error states

### Profile

- [ ] `/profile` page with name and email
- [ ] `/profile/settings` page shell (sections filled in Phase 2)

## Phase 2 — Statuses, Kanban, history, No Answer, themes

### Pipeline status settings

- [ ] API: list, create, rename, reorder active statuses
- [ ] Keep status IDs stable when labels change
- [ ] Keep Rejected and Withdrawn as closing statuses even after renaming
- [ ] Settings UI: add, rename, reorder stages
- [ ] Tests: renaming/reordering does not break existing applications

### Application history

- [ ] Migration: `application_event` (type, previous/new status, source, timestamp, payload)
- [ ] Write an event on create and on every status change
- [ ] Allow a manual reply/note event (needed for clearing No Answer)
- [ ] API: `GET /applications/:id/events`
- [ ] Timeline in the details panel, in chronological order

### Kanban board

- [ ] Table / Board switch in the toolbar (shared filters and data)
- [ ] Board columns = active statuses in settings order
- [ ] Application cards
- [ ] Drag and drop between columns with dnd-kit
- [ ] Rejected and Withdrawn drop zones, visible only while dragging, sticky on scroll
- [ ] Highlight and expand the zone on hover without moving its drop area
- [ ] Optimistic update; revert the card and show an error if saving fails
- [ ] Closed applications leave the board; reachable through table filters
- [ ] Card menu with the same move actions (keyboard access)
- [ ] Reduced-motion mode uses a static highlight

### No Answer

- [ ] Function that decides if an application has No Answer (status Applied, no substantive reply, days ≥ threshold, user's time zone)
- [ ] Unit tests: time zones, threshold edges, reply clears the badge, acknowledgment does not
- [ ] Threshold setting in settings (minimum 1)
- [ ] Recalculate when the threshold changes
- [ ] No Answer badge in table and board
- [ ] No Answer filter shared by both views

### Themes

- [ ] Add Silent Waters Light and Dark tokens from `tracework-design.md` to `client/src/styles/`
- [ ] Status and reply badge colors for both themes
- [ ] Theme switcher in settings; save to `user_setting`
- [ ] Apply theme to dialogs, dropdowns, overlays
- [ ] Check contrast and readability in both themes

## Phase 3 — Excel import

### Parsing

- [ ] Choose an `.xlsx` parsing library
- [ ] Upload endpoint or client-side parsing (decide and note the reason)
- [ ] Sheet picker when the workbook has more than one sheet

### Column mapping

- [ ] Suggest mappings for Company, Position, Applied date, Status
- [ ] Optional field mapping (Source, URL, Missing Requirements, Notes)
- [ ] Resolve ambiguous date formats
- [ ] Map unknown status values to the account's stages
- [ ] Map legacy "No Answer" / "Ignore" values to Applied
- [ ] Tests for date parsing and status mapping

### Preview and duplicates

- [ ] Detect duplicates: same company + position, ignoring case and extra whitespace
- [ ] Show the count of possible duplicates at the top
- [ ] For each flagged row show the matching records
- [ ] Add as new / Skip per row, with Apply to all similar rows
- [ ] Tests for duplicate detection and decisions

### Confirm

- [ ] Import selected rows in one request
- [ ] Show added, skipped, and invalid counts
- [ ] Evaluate No Answer for imported rows right away

## Phase 4 — Gmail monitoring

### Connection

- [ ] Set up a Google Cloud project and OAuth client (read-only Gmail scope)
- [ ] Add OAuth redirect URIs for local dev, staging, and production
- [ ] Migration: `gmail_connection` (tokens stored securely)
- [ ] OAuth connect and disconnect flow on the server
- [ ] Connect Gmail banner in Applications (only while disconnected)
- [ ] Connect/disconnect in settings

### Sync

- [ ] Migration: `gmail_message` (unique per Gmail account + message ID)
- [ ] Periodic sync job in `server/src/jobs/`
- [ ] Idempotent sync; safe to retry
- [ ] Link messages from known threads to their application
- [ ] Match new threads by company, position, and sender
- [ ] Tests for retries and duplicate messages

### AI classification

- [ ] Choose an AI provider and model
- [ ] Classify replies: acknowledgment, screening, test task, tech interview, offer, rejection, other
- [ ] Clear link + clear class → change status automatically and record the source message and previous status
- [ ] Ambiguous match or class → send to review queue
- [ ] Conflict with a later manual change or a closed application → send to review queue
- [ ] Tests for uncertain classifications and conflicts

### Review and history

- [ ] Review emails entry in Applications when items need attention
- [ ] Review panel: confirm, change, or dismiss suggestions
- [ ] Show Gmail messages in the timeline (sender, subject, time, snippet, link)
- [ ] Let the user undo or correct an automatic status change
- [ ] Substantive reply clears No Answer

## Phase 5 — Release

- [ ] Full test run and lint pass

### Versioning and changelog

- [ ] Root `CHANGELOG.md` in Keep a Changelog style: `## Unreleased` on top, then `## x.y.z — YYYY-MM-DD` per release, user-facing lines only
- [ ] Inject app version (root `package.json`) and commit SHA into the client with Vite `define`
- [ ] Return the version from the server (`GET /health` or `GET /version`)
- [ ] Parse function: changelog text → list of `{ version, date, body }`, skipping `Unreleased`
- [ ] Tests for the parse function
- [ ] Version label in the app navigation or user menu (a button)
- [ ] "What's new" dialog: latest version expanded, older versions in an accordion, rendered with `react-markdown`
- [ ] Release flow: rename `Unreleased`, run `npm version` at the root, deploy

### Staging

- [ ] Apply migrations to the staging Supabase project
- [ ] Set staging environment variables
- [ ] Deploy the frontend; serve the app shell on direct visits to routes
- [ ] Deploy the backend and the sync job
- [ ] Manual check of every route in both themes and on a narrow screen
- [ ] Check sign-up, login, password reset, and Gmail connect with test accounts

### Production

- [ ] Create the production Supabase project and apply migrations
- [ ] Create a Resend API key for production and configure custom SMTP in production Supabase
- [ ] Set Site URL and Redirect URLs in production Supabase to the production frontend
- [ ] Set production environment variables
- [ ] Deploy the frontend, backend, and sync job
- [ ] Smoke test: sign up, password reset, add an application
- [ ] Review Google restricted-scope and Limited Use requirements before a public launch
