# miko

A full-stack movie & TV streaming-tracker web app. Browse trending and top-rated
titles (powered by [TMDB](https://www.themoviedb.org/)), watch movies and episodes
via an embedded player, build a personal watchlist, and get a personalized
"Continue Watching" + "For You" feed driven by your viewing history.

Built with **React 19**, **TanStack Router & Query**, **Hono**, **Firebase**, and
**Tailwind CSS v4**, wrapped in a "luxury black" UI with a premium gold accent.

---

## Features

- **Discovery** — Trending, Popular Movies, and Top Rated TV rows on the home page.
- **Search** — Debounced, infinite-scroll multi-search across movies and TV shows.
- **Detail pages** — Movie and TV show pages with backdrops, metadata, seasons,
  episode selectors, and "More Like This" recommendations.
- **Streaming** — Embedded iframe player for movies and TV episodes.
- **Watchlist** — Add or remove any movie/show to a personal, persistent watchlist.
- **Watch history** — Automatically tracks what you watch; powers
  "Continue Watching".
- **Personalized "For You"** — Derives recommendations from your recent history,
  filtered and shuffled for a dynamic feed.
- **Authentication** — Email/password sign-up & sign-in, plus Google OAuth.
- **Profile** — Update display name / avatar, clear all data, and sign out.

---

## Tech Stack

| Layer           | Technology                                                        |
| --------------- | ----------------------------------------------------------------- |
| UI Framework    | React 19 + React Compiler                                         |
| Routing         | TanStack Router (file-based, auto code-splitting)                 |
| Data Fetching   | TanStack Query                                                    |
| Styling         | Tailwind CSS v4 + tailwind-variants + tailwind-merge              |
| UI Primitives   | @base-ui/react                                                    |
| Icons           | @iconify/react                                                    |
| Build           | Vite 7 + vite-plugin-vercel                                       |
| Server          | Hono + @hono/standard-validator                                   |
| Validation      | arktype                                                           |
| Backend         | firebase-admin (Authentication + Firestore)                       |
| Media Data      | The Movie Database (TMDB) API                                     |
| HTTP Client     | ofetch                                                            |
| Language        | TypeScript 5.9                                                    |
| Lint/Format     | Biome                                                             |
| Git Hooks       | Lefthook                                                          |
| Package Manager | pnpm (+ Bun for dev runtime/server)                              |

---

## Project Structure

```
miko/
├── server/                 # Hono API (serverless)
│   ├── index.ts            # App entry — mounts /api routes + logger
│   ├── routes/             # auth, tmdb, watchlist, history
│   ├── lib/                # firebase, tmdb client, typed-env, errors, test-utils
│   └── middleware/         # session (cookie-based) middleware
├── src/
│   ├── main.tsx            # App bootstrap — QueryClient, Router, Toast providers
│   ├── routes/             # TanStack Router file-based routes
│   ├── features/           # Feature modules (auth, browse, history, ...)
│   │   └── [name]/
│   │       ├── *.hooks.ts  # React Query hooks for the feature
│   │       ├── types.ts    # Domain types
│   │       └── components/ # Feature-scoped components
│   ├── ui/                 # Promoted, shared UI components + design system
│   ├── hooks/              # Cross-cutting hooks (useInfiniteScroll, useIsMobile)
│   ├── utils/              # Helpers (pagination)
│   └── styles/
│       └── theme.css       # Tailwind v4 @theme tokens + custom utilities
├── scripts/
│   └── concurrently.ts     # Runs multiple `bun` tasks in parallel
├── vercel.json             # SPA + API rewrites
├── biome.json              # Lint/format config (4-space, double quotes)
├── lefthook.yml            # pre-push: lint:check + typecheck
└── vite.config.ts          # Vite + TanStack Router + React Compiler + Vercel
```

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) (LTS)
- [pnpm](https://pnpm.io/)
- [Bun](https://bun.sh/) (used as the dev runtime for the API)
- A [TMDB](https://www.themoviedb.org/settings/api) account (for an API token)
- A [Firebase](https://console.firebase.google.com/) project (Auth + Firestore)

### Environment Variables

Create a `.env` file in the project root with all of the following:

| Variable                   | Description                                                            |
| -------------------------- | --------------------------------------------------------------------- |
| `FIREBASE_SERVICE_ACCOUNT` | Firebase service-account JSON, provided as a **single-line JSON string** |
| `TMDB_TOKEN`               | TMDB API read access token (Bearer)                                    |
| `FIREBASE_API_KEY`         | Firebase Web API key                                                   |
| `GOOGLE_CLIENT_ID`         | Google OAuth client ID                                                 |
| `GOOGLE_CLIENT_SECRET`     | Google OAuth client secret                                             |

These are validated at startup via an arktype schema in
`server/lib/typed-env.ts`.

### Installation

```bash
pnpm install
```

### Development

Runs the Vite dev server (port 5000) and the Hono API (port 3000) concurrently:

```bash
pnpm dev
```

The Vite dev server proxies `/api` requests to `http://localhost:3000`.

### Firebase Setup

1. Enable **Email/Password** and **Google** sign-in providers in Firebase Auth.
2. Create a **Firestore** database.
3. Add your domain to the **Authorized domains** list for Auth.
4. For Google OAuth, add the redirect URI
   `https://<your-domain>/api/auth/google/callback` to your Google Cloud OAuth
   client's authorized redirect URIs.

---

## Scripts

| Script        | Description                                                        |
| ------------- | ----------------------------------------------------------------- |
| `pnpm dev`    | Start Vite + API concurrently (Bun)                               |
| `pnpm build`  | Type-check and build for production (runs `prebuild` for the API) |
| `pnpm preview`| Preview the production build                                      |
| `pnpm lint`   | Run Biome check and auto-fix                                      |
| `pnpm lint:check` | Run Biome check (no writes) — used in pre-push hook           |
| `pnpm typecheck` | Run `tsc` with no emit — used in pre-push hook                 |

> Lefthook's `pre-push` hook runs `lint:check` and `typecheck` in parallel.

---

## API Reference

All routes are mounted under `/api`.

### Auth — `/api/auth`

| Method | Endpoint             | Auth    | Description                          |
| ------ | -------------------- | ------- | ------------------------------------ |
| POST   | `/auth/login`        | —       | Exchange a Firebase ID token         |
| POST   | `/auth/register`     | —       | Email/password sign-up               |
| POST   | `/auth/login-email`  | —       | Email/password sign-in               |
| GET    | `/auth/google`       | —       | Start Google OAuth flow              |
| GET    | `/auth/google/callback` | —    | OAuth callback → session             |
| GET    | `/auth/me`           | session | Current user                         |
| PATCH  | `/auth/profile`      | session | Update display name / photo          |
| POST   | `/auth/logout`       | —       | Clear session cookie                 |

### TMDB Proxy — `/api/tmdb`

Public (no session required); forwards to the TMDB v3 API.

| Method | Endpoint                                  |
| ------ | ----------------------------------------- |
| GET    | `/tmdb/trending/:mediaType/:timeWindow`   |
| GET    | `/tmdb/movie/popular`                     |
| GET    | `/tmdb/tv/top_rated`                      |
| GET    | `/tmdb/movie/:id`                         |
| GET    | `/tmdb/tv/:id`                            |
| GET    | `/tmdb/tv/:id/season/:season`             |
| GET    | `/tmdb/search/multi`                      |
| GET    | `/tmdb/movie/:id/recommendations`         |
| GET    | `/tmdb/tv/:id/recommendations`            |
| GET    | `/tmdb/discover/:mediaType`               |

### Watchlist — `/api/watchlist` (session required)

| Method | Endpoint             | Description                |
| ------ | -------------------- | -------------------------- |
| GET    | `/watchlist`         | List watchlist             |
| GET    | `/watchlist/:type/:id` | Check membership         |
| POST   | `/watchlist`         | Add item                   |
| DELETE | `/watchlist/:type/:id` | Remove item              |
| DELETE | `/watchlist`         | Clear entire watchlist     |

### History — `/api/history` (session required)

| Method | Endpoint             | Description                        |
| ------ | -------------------- | ---------------------------------- |
| GET    | `/history`           | Recent 50 watched items            |
| POST   | `/history`           | Upsert item (updates `watchedAt`)  |
| DELETE | `/history/:type/:id` | Remove item                        |
| DELETE | `/history`           | Clear all history                  |

Watchlist and history are stored in Firestore under
`users/{uid}/watchlist` and `users/{uid}/history`.

### Session

Authentication uses a cookie-based session (`session` cookie, 5-day expiry,
`httpOnly` + `secure` + `sameSite: Lax`). The session middleware verifies the
Firebase session cookie and sets the `uid` on the request context.

---

## Routes / Pages

| Path                  | Page                        | Guard       |
| --------------------- | --------------------------- | ----------- |
| `/`                   | Home (rows + personalized)  | Public      |
| `/login`              | Login / Register            | Guest       |
| `/search?q=`          | Search results              | Public      |
| `/watchlist`          | Your watchlist              | Auth        |
| `/profile`            | Profile & settings          | Auth        |
| `/movie/:id`          | Movie player + details      | Public      |
| `/tv/:id`             | TV show overview            | Public      |
| `/tv/:id/:season/:episode` | Episode player         | Public      |

---

## Design System

The theme lives in `src/styles/theme.css` using Tailwind v4 `@theme` tokens:

- **Surfaces** — luxury black (`#050505`) with raised/overlay tiers
- **Accent** — premium gold (`#d4af37`)
- **Glass effects** — `glass`, `glass-strong`, `glow-accent`, `glow-soft` utilities
- **Typography** — Sora (preloaded from Bunny Fonts)
- **Animations** — fade, scale, shimmer, pulse-glow, float, icon-bounce
- **Radii** — card `1rem`, button `0.75rem`

### Component Conventions

- **Closed prop surfaces** — no `className`, no `HTMLAttributes` extends
- **Semantic slots** — `info`, `actions` props instead of `children`
- **Component owns decisions** — position, width, padding fixed internally
- **Iconify** for all icons
- **BaseUI** for accessible primitives (Menu, Tooltip, Toast, etc.)
- **React 19 idioms** — `ref` is a regular prop; no `forwardRef`, no
  `displayName`, no manual `memo` (React Compiler handles it)
- **Promotion rule** — 1 feature = local, 2 = tolerated, 3+ = promote to `src/ui/`

---

## Deployment

The app is configured for **Vercel**:

- `vercel.json` rewrites `/api/*` to the serverless API and everything else to
  `index.html` (SPA fallback).
- The `prebuild` script compiles `server/index.ts` to `_api` (Node target) for
  the serverless function.
- `routeTree.gen.ts` (generated by TanStack Router) and `_api` (built server
  output) are build artifacts and are gitignored.

Set all [environment variables](#environment-variables) in your Vercel project
settings.

---

## Notes

- The streaming player embeds content from a third-party service keyed by TMDB
  IDs; Miko does not host any media.
- Vitest is installed and a test-token helper (`server/lib/test-utils.ts`)
  exists, but no tests have been written yet.
