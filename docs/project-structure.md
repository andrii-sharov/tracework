# Project Structure

```
tracework/
├── package.json              npm workspaces: client, server, shared
├── docs/
├── shared/                   TS types and constants used by both sides
│   └── src/                  status IDs, Application/Event DTOs, API contracts
├── client/                   React + TS + Vite + React Router
│   └── src/
│       ├── app/              router, providers, auth guard
│       ├── pages/            login, signup, reset-password, applications,
│       │                     application-details, profile, settings
│       ├── features/
│       │   ├── applications/ table, board (dnd-kit), details panel, filters
│       │   ├── import/       xlsx upload, column mapping, preview, duplicates
│       │   ├── gmail/        banner, review emails panel
│       │   └── settings/     pipeline statuses, No Answer threshold, theme
│       ├── components/ui/    shadcn/ui components
│       ├── lib/              api client, supabase client, utils
│       └── styles/           Tailwind + theme tokens (Silent Waters light/dark)
├── server/                   Node + Express + TS
│   └── src/
│       ├── app.ts            express app setup
│       ├── index.ts          entry point
│       ├── middleware/       auth (Supabase session), ownership, errors
│       ├── modules/          one folder per domain: routes, service, repo, tests
│       │   ├── applications/ CRUD, status changes, history events
│       │   ├── pipeline/     user_pipeline_status
│       │   ├── settings/     user_setting
│       │   ├── no-answer/    badge calculation
│       │   ├── import/       xlsx parsing, mapping, duplicate detection
│       │   └── gmail/        OAuth, sync, AI classification, review queue
│       ├── jobs/             periodic Gmail sync
│       └── db/               Supabase client, typed queries
└── supabase/
    ├── migrations/           tables + row-level security policies
    └── seed.sql
```
