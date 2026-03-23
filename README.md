# Splendor Online

<p align="center">
  <img src="https://raw.githubusercontent.com/Masniper/splendor-client/main/public/images/startup-bg.jpg" alt="Splendor Online cover" width="720" />
</p>

<p align="center">
  <strong>Browser-based multiplayer Splendor</strong> — real-time rooms, bets, profiles, and full game flow over Socket.io.
</p>

<p align="center">
  <a href="https://github.com/Masniper/splendor-app"><img src="https://img.shields.io/badge/repo-splendor--app-181717?logo=github" alt="Repository" /></a>
  <a href="https://github.com/Masniper/splendor-client"><img src="https://img.shields.io/badge/client-splendor--client-181717?logo=github" alt="Client" /></a>
  <a href="https://github.com/Masniper/splendor-server"><img src="https://img.shields.io/badge/server-splendor--server-181717?logo=github" alt="Server" /></a>
  <img src="https://img.shields.io/badge/node-%3E%3D18-339933?logo=nodedotjs&logoColor=white" alt="Node.js" />
  <img src="https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black" alt="React" />
  <img src="https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/version-1.0.0-blue" alt="Version" />
  <img src="https://img.shields.io/badge/license-mixed-lightgrey" alt="License" />
</p>

This **parent** repository wires two [Git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) into one runnable stack:

| Path | GitHub repository | Role |
|------|-------------------|------|
| [`front-end/`](./front-end) | [Masniper/splendor-client](https://github.com/Masniper/splendor-client) | React + Vite client |
| [`back-end/`](./back-end) | [Masniper/splendor-server](https://github.com/Masniper/splendor-server) | Express + Socket.io + Prisma API |

---

## Features

- **Accounts** — Register, login, guest play, guest → full account upgrade
- **Rooms** — Create/join by code, public room browser, host controls, bets and settlement
- **Realtime game** — Splendor actions over Socket.io (tokens, cards, nobles, reserve, rematch)
- **Room chat** — In-room messages, quick phrases, emoji picker
- **Audio** — Web Audio UI and game cues, user mute
- **UX** — Toasts, chat notifications, responsive layout, card and token flight animations
- **PWA** — Installable client (`vite-plugin-pwa`); see [`front-end/README.md`](./front-end/README.md)
- **API docs** — OpenAPI/Swagger UI on the backend (`/api-docs`)

---

## Tech stack

| Area | Technologies |
|------|----------------|
| **Frontend** | React 19, Vite 6, Tailwind CSS v4, Framer Motion, Socket.io client, React Router, PWA |
| **Backend** | Node.js, Express 5, Socket.io, Prisma, Zod, JWT, bcrypt |
| **Database** | PostgreSQL 16 (Prisma); Docker Compose provides a named volume |
| **Tooling** | TypeScript, Vitest (backend tests) |
| **Infrastructure** | Git submodules ([`.gitmodules`](./.gitmodules)); Docker Compose (API + Nginx static UI + Postgres) |

---

## Prerequisites

- **Node.js** ≥ 18 (LTS recommended)
- **npm** (or compatible package manager)
- **Git** with submodule support
- **Docker** (optional) — for Compose-only Postgres or full stack

---

## Installation and local setup

### Clone with submodules

```bash
git clone --recurse-submodules https://github.com/Masniper/splendor-app.git
cd splendor-app
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

### Standalone client or server

You may also clone [splendor-client](https://github.com/Masniper/splendor-client) or [splendor-server](https://github.com/Masniper/splendor-server) directly and run `npm install` / `npm run dev` there.

### 1. Backend

Start PostgreSQL (Compose example):

```bash
docker compose up -d postgres
```

Then:

```bash
cd back-end
npm install
# Create back-end/.env — see Environment variables
npx prisma migrate dev
npm run dev
```

Default API: **`http://localhost:5001`**

### 2. Frontend

```bash
cd front-end
npm install
npm run dev
```

Default client: **`http://localhost:3000`**

Optional `front-end/.env` when the API is not same-origin:

```env
VITE_API_BASE_URL=http://localhost:5001/api
VITE_SOCKET_URL=http://localhost:5001
```

### Frontend static assets (optional)

The client ships core **sprite sheets** under `front-end/public/images/` (e.g. `cards.jpg`, `nobles.jpg`, `numbers_sheet.png`, `gems.png`, `tokens.png`, backgrounds, favicons).

**Per-card JPEGs**, **per-noble JPEGs**, and **deck table textures** live under:

- `public/images/cards/` — `/images/cards/{cardId}.jpg`
- `public/images/nobles/` — `/images/nobles/{nobleId}.jpg`
- `public/images/textures/` — `diagmonds-dark.png`, `diagmonds-light.png`

These folders may be empty in a minimal checkout. To populate **placeholder** art locally:

```bash
cd front-end
bash scripts/download-splendor-assets.sh
```

If files are missing, the UI still runs: `cardAssets.ts` falls back to remote placeholder images when local files fail to load.

### Submodule bumps (maintainers)

After pushing to **splendor-client** or **splendor-server**, update this repo:

```bash
cd front-end && git pull origin main && cd ..
cd back-end && git pull origin main && cd ..
git add front-end back-end
git commit -m "Bump client and/or server submodules"
git push
```

---

## Environment variables

Never commit real secrets. Use placeholders in documentation and examples.

### Root (Docker Compose)

Copy [`.env.example`](./.env.example) to `.env` next to `docker-compose.yml`.

| Variable | Required | Description |
|----------|----------|-------------|
| `JWT_SECRET` | **Yes** | Secret passed into the API container; must be a long random string |
| `PROVIDER` | **Yes** | Prisma datasource provider (default: `postgresql`) |
| `WEB_PORT` | No | Host port for the Nginx web container (default: `8080`) |

### Backend (`back-end/.env`)

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | **Yes** | Prisma PostgreSQL URL, e.g. `postgresql://USER:PASSWORD@localhost:5432/splendor?schema=public` |
| `PROVIDER` | **Yes** | Prisma datasource provider (e.g. `postgresql`), used by `prisma/schema.prisma` |
| `JWT_SECRET` | Strongly recommended | JWT signing secret |
| `PORT` | No | HTTP and Socket.io port (default: `5001`) |

Example:

```env
DATABASE_URL="postgresql://splendor:splendor@localhost:5432/splendor?schema=public"
PROVIDER="postgresql"
JWT_SECRET="change-me-to-a-long-random-secret"
PORT=5001
```

### Frontend (`front-end/.env`, optional)

| Variable | Required | Description |
|----------|----------|-------------|
| `VITE_API_BASE_URL` | No | REST base URL. Default: `/api`. Local API: `http://localhost:5001/api` |
| `VITE_SOCKET_URL` | No | Socket.io origin (no path). Local: `http://localhost:5001` |

More detail: [`back-end/README.md`](./back-end/README.md), [`front-end/README.md`](./front-end/README.md).

---

## Docker (full stack)

From the repository root, with submodules checked out. Requires Docker and Compose v2.

1. Copy and edit environment:

   ```bash
   cp .env.example .env
   # Set JWT_SECRET (required for Compose)
   ```

2. Build and start Postgres, API, and static web:

   ```bash
   docker compose up --build -d
   ```

3. Open **`http://localhost:8080`** (or `WEB_PORT` from `.env`).

The Nginx image serves the built client with same-origin `/api` and Socket.io. Swagger UI is available at **`/api-docs`** (proxied to the API). Data persists in Docker volume **`postgres_data`**.

---

## Project structure

```
splendor-app/
├── .gitmodules              # Submodule URLs (client + server)
├── .env.example             # Docker Compose template (JWT, web port)
├── docker-compose.yml       # Postgres + API + web
├── README.md
├── front-end/               # Submodule: React + Vite client
│   ├── public/              # Static assets (favicons, images, icons)
│   │   ├── icons/           # Gem PNGs (/icons/*.png)
│   │   └── images/          # Sprites, backgrounds; optional cards/, nobles/, textures/
│   ├── scripts/
│   │   └── download-splendor-assets.sh   # Optional: per-card/noble/texture placeholders
│   ├── src/
│   └── package.json
└── back-end/                # Submodule: Express API
    ├── prisma/
    │   ├── schema.prisma
    │   └── migrations/
    ├── src/
    └── package.json
```

---

## API documentation

- **REST** — `http://localhost:<PORT>/api`
- **Swagger UI** — `http://localhost:<PORT>/api-docs`
- **Realtime** — Socket.io on the same port as HTTP

See [`back-end/README.md`](./back-end/README.md) for routes and socket events.

---

## License

- **Parent repo** — documentation and integration; no single license file at root.
- **Client** ([splendor-client](https://github.com/Masniper/splendor-client)) — **MIT** (see `front-end/LICENSE` when the submodule is present).
- **Server** ([splendor-server](https://github.com/Masniper/splendor-server)) — **ISC** (see `back-end/package.json`).

---

## Related documentation

- [`front-end/README.md`](./front-end/README.md) — Vite scripts, PWA, env vars
- [`front-end/public/README.md`](./front-end/public/README.md) — Static asset layout
- [`back-end/README.md`](./back-end/README.md) — Prisma, env, chat reference
- [`PRODUCTION_CHECKLIST.md`](./PRODUCTION_CHECKLIST.md) — Production deployment notes
