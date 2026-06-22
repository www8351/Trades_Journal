# Miror Copy Trades

Real-time **trade-copy analytics & mirroring dashboard** for proprietary-firm and
multi-broker accounts. Imports execution data, normalizes it across brokers,
computes risk/performance metrics, and renders them through an animated,
data-dense React interface.

Built to demonstrate production-grade **full-stack SaaS engineering**:
low-latency data flows, strict type safety end-to-end, secure auth, and a
clean separation of code from configuration.

> **Note on this repository.** This is a sanitized public mirror of a private
> production codebase. No secrets exist in source or in Git history — all
> credentials are injected at runtime via environment variables / platform
> secret stores (see [`.env.example`](./.env.example)). Wire it to your own
> Supabase project to run it.

---

## Stack

| Layer        | Technology |
|--------------|------------|
| Framework    | Next.js 16 (App Router, Server Actions, Route Handlers) |
| Language     | TypeScript 5 (strict) — typed end-to-end against the DB schema |
| UI           | React 19, Tailwind CSS 4, `class-variance-authority`, shadcn-style primitives |
| Animation    | Framer Motion 12 |
| Data viz     | Recharts 3 |
| Tables       | TanStack React Table 8 (sorting, filtering, virtualization-ready) |
| Backend      | Supabase (Postgres + Row-Level Security + Auth) |
| Validation   | Zod 4 (parse-don't-validate at every boundary) |
| Precision    | `decimal.js` — no float drift on monetary math |
| Import       | PapaParse — streaming CSV ingest of broker statements |
| Testing      | Vitest 4 |

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│  Browser (React 19 Client Components)                              │
│  charts · animated trade tables · import wizard · settings        │
└───────────────▲───────────────────────────────┬──────────────────┘
                │ typed props / server actions   │ user events
┌───────────────┴───────────────────────────────▼──────────────────┐
│  Next.js 16 Server Layer                                          │
│  • Server Actions  (src/actions)   — mutations, auth flows        │
│  • Route Handlers  (src/app/auth)  — OAuth callback               │
│  • Proxy/session refresh           — cookie-based auth on every req│
└───────────────▲───────────────────────────────┬──────────────────┘
                │ RLS-scoped queries             │ parse/normalize
┌───────────────┴────────────────┐  ┌────────────▼──────────────────┐
│  Supabase (Postgres + Auth)     │  │  Domain libs (src/lib)         │
│  • Row-Level Security policies   │  │  • metrics/  — perf & risk calc│
│  • anon key safe for browser     │  │  • import/   — CSV → trades    │
│  • service-role key server-only  │  │  • trades/   — normalization   │
└──────────────────────────────────┘  │  • data/     — query layer     │
                                       └────────────────────────────────┘
```

### Directory map

```
src/
├── actions/         Server Actions — auth + mutations (no secrets in source)
├── app/
│   ├── (dashboard)/ Authenticated app shell + routes
│   ├── auth/        OAuth callback Route Handler
│   └── login/       Public auth entry
├── components/
│   ├── charts/      Recharts wrappers (equity curve, drawdown, distribution)
│   ├── trades/      Animated, sortable trade tables (TanStack + Framer Motion)
│   ├── import/      CSV import wizard (PapaParse)
│   ├── dashboard/   KPI cards, layout
│   ├── accounts/    Multi-account / multi-broker management
│   ├── settings/    User & risk-rule configuration
│   └── ui/          Reusable primitives (cva + tailwind-merge)
├── hooks/           Typed React hooks
├── lib/
│   ├── supabase/    Browser + server clients (env-driven, RLS-aware)
│   ├── metrics/     Performance & risk calculations (decimal.js)
│   ├── import/      Broker-statement parsing & mapping
│   ├── trades/      Cross-broker trade normalization
│   └── data/        Query layer
└── types/           Generated DB types + domain models
```

### Engineering principles applied

- **Code/config separation** — zero hardcoded credentials. Every secret is read
  from `process.env`; templates live in `.env.example`; real values come from
  the platform secret store (Vercel env / GitHub Actions secrets / Vault).
- **Type safety to the database** — Supabase clients are generic over a
  generated `Database` type, so query results are typed at compile time.
- **Security by policy, not by obscurity** — access controlled by Postgres
  Row-Level Security; the browser only ever holds the RLS-scoped anon key.
- **Monetary correctness** — all P&L / risk math uses `decimal.js`; floats are
  never used for money.
- **Validation at boundaries** — external input (CSV rows, form data, OAuth
  payloads) is parsed through Zod schemas before entering the domain.

---

## Local development

```bash
# 1. Install
npm install

# 2. Configure — copy the template and fill in your own Supabase keys
cp .env.example .env.local

# 3. Run
npm run dev          # http://localhost:3000

# 4. Quality gates
npm run lint
npm test             # Vitest
npm run build        # production build
```

### Prerequisites
- Node.js 20+
- A Supabase project (free tier is enough) — create one, then copy its
  `URL` and `anon` key from **Settings → API** into `.env.local`.

---

## Security & secret handling

| Concern                | Approach |
|------------------------|----------|
| Secrets in source      | None. Source reads `process.env.*` only. |
| Secrets in Git history | None. Public mirror initialized clean — original private history is not exposed. |
| `.env.local`           | Gitignored. Never committed. |
| Browser-exposed key    | Supabase anon key only — scoped by Row-Level Security. |
| Admin key              | `SUPABASE_SERVICE_ROLE_KEY` is server-only; never `NEXT_PUBLIC_`-prefixed. |
| Production injection    | Vercel env vars / GitHub Actions secrets / Vault — not files. |

---

## License

MIT — see [`LICENSE`](./LICENSE).
