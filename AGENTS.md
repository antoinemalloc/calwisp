# AGENTS.md

> Instructions for agents working on calwisp. Revise this file whenever meaningful decisions are made, so future sessions stay consistent.

Calwisp is a greenfield collaborative calendar web app, built to be resumable to native Android/iOS.

## Product requirements (confirmed)

- Platform: web app first (SPA/PWA), resumable to native Android/iOS near-term.
- Collaboration: live realtime multi-user editing with presence; concurrent edits must not corrupt data.
- Offline: offline-first, local cache and sync on reconnect (also matters for the future mobile app).
- Backend: self-managed full stack (own auth, own DB, no BaaS).

## Architecture decision record (locked)

- Pattern: schema-first, Zod as single source of truth, RPC with shared types (not ad-hoc HTTP).
- Realtime/concurrency: Yjs CRDT + self-hosted y-websocket + y-indexeddb. Not ElectricSQL, not server-authoritative optimistic UI.
- Data model: Calendar / Event / Attendee (role + status) / Presence (runtime, Redis). Recurring events = series-level RRULE with exception/override rows. Store timestamps UTC with per-event timezone override; Luxon for DST-safe conversion.
- Native path: single Zod schema -> OpenAPI -> Swift/Kotlin codegen, same tRPC endpoints, Yjs sync core, SQLite on native. Storage backend abstracted behind the sync layer (IndexedDB on web, SQLite on native) so the swap is a thin change.

### Frontend
- React + Vite + TypeScript
- Tailwind CSS
- shadcn/ui
- React Router
- Offline cache: y-indexeddb (IndexedDB) on web; SQLite on native, abstracted

### Backend
- HTTP/WS: Express (+ ws / socket.io)
- Language: TypeScript
- Validation: Zod v4
- ORM: Drizzle ORM (drizzle-kit for migrations)
- DB driver: pg
- DB: PostgreSQL 16 + Redis (presence, job queue)
- Auth: better-auth (JWT access + rotating refresh; HttpOnly cookie on web, Secure Storage on native)
- Realtime/sync: self-hosted Yjs + y-websocket
- Background jobs: BullMQ

## Libraries
- CRDT: yjs
- Sync provider: y-websocket
- Local persistence: y-indexeddb (swappable for SQLite on native)
- Many packages are at new major versions (Zod 4, tRPC 11). Verify current versions before relying on or pinning them; do not trust stale training data.

## Build order (phased)
1. npm-workspaces monorepo + `@calwisp/core` (Zod + Drizzle schema), tRPC skeleton, pinned versions, CI codegen
2. Auth via better-auth
3. Realtime core: Yjs + y-websocket + y-indexeddb, RRULE series + overrides data model
4. Business logic + sync + BullMQ, Drizzle persistence
5. Presence + scaling + Yjs log compaction
6. PWA hardening and native spike (OpenAPI/codegen, SQLite storage swap)

## Project conventions
- Before coding, state assumptions; ask when requirements are ambiguous. Touch only what the task requires.
- Verify library versions and APIs before relying on them (training data is stale).
- Use `npx` for global npm CLIs.
- Self-update this file whenever meaningful decisions change.
