# Vivah — AI Wedding Planner

A full-stack Indian wedding planning app with 7 modules: Dashboard, Budget, Guests, Vendors, Tasks, Invitations, and Outfits.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080, proxied at `/api`)
- `pnpm --filter @workspace/wedding-planner run dev` — run the frontend (port 25016, proxied at `/`)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run typecheck:libs` — build composite libs only
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET`

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React 18 + Vite, TailwindCSS, shadcn/ui, wouter (routing), TanStack Query
- API: Express 5, pino logger
- DB: PostgreSQL + Drizzle ORM (`lib/db`)
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (OpenAPI spec → React Query hooks + Zod schemas)
- Build: esbuild (CJS bundle for API server)

## Where things live

```
artifacts/
  api-server/src/      — Express routes, app.ts, index.ts, lib/{logger,serialize}.ts
  wedding-planner/src/ — React app (pages/, components/, hooks/, lib/)
lib/
  api-spec/            — openapi.yaml (source of truth for API contract)
  api-zod/             — generated Zod validators (from codegen)
  api-client-react/    — generated React Query hooks + custom-fetch.ts
  db/                  — Drizzle schema (schema/*.ts) + migrations
```

- DB schema: `lib/db/src/schema/` (7 files: wedding, budget, guests, vendors, tasks, invitations, outfits)
- API contract: `lib/api-spec/openapi.yaml`

## Architecture decisions

- Contract-first: OpenAPI spec → codegen → Zod validators + React Query hooks; never write these by hand
- `lib/api-server/src/lib/serialize.ts` handles Date → ISO string conversion for JSON responses
- `GET /api/wedding` intentionally returns 404 when no wedding is configured (not an error)
- `/guests/stats` route is registered before `/guests/:id` to avoid Express matching "stats" as an id
- All routes mounted under `/api` prefix; the shared reverse proxy routes `/api` → API server (port 8080), `/` → frontend (port 25016)

## Product

- **Dashboard**: Wedding countdown, budget/guest/task/vendor stats + progress bars; Set Up Wedding dialog
- **Budget**: Category-based budget tracker with allocated vs. spent, over-budget warnings
- **Guests**: RSVP management with meal preferences (veg/nonveg/vegan/jain), side (bride/groom/both), table assignments, search & filter
- **Vendors**: Track vendors by category (catering, photography, etc.) and status (enquiry → paid)
- **Tasks**: Month-by-month checklist grouped by timeline (12 months before → final week)
- **Invitations**: Template-based invitation generator in English, Hindi, Marathi, or mixed language; 5 styles (traditional, modern, floral, minimal, royal); save and copy
- **Outfits**: Color palette + outfit suggestions for each wedding event (mehendi, haldi, sangeet, wedding, reception) based on season, venue, and style

## Gotchas

- Run `pnpm --filter @workspace/api-spec run codegen` after changing `openapi.yaml`, then update route imports if schema names changed
- Do not run `pnpm dev` at the workspace root — use `restart_workflow` or the workflow panel
- `lib/api-zod/src/index.ts` must only export from `./generated/api` (not `./generated/types`) to avoid duplicate export conflicts

## Pointers

- See `.local/skills/pnpm-workspace` for workspace structure and codegen details
- See `.local/skills/pnpm-workspace/references/server.md` for Express route patterns
