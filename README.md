# Splendor Online

<p align="center">
  <img src="https://raw.githubusercontent.com/Masniper/splendor-client/main/public/images/startup-bg.jpg" alt="Splendor Online — cover art" width="720" />
</p>

<p align="center">
  <em>Cover image from the client package: <code>front-end/public/images/startup-bg.jpg</code> (<a href="https://github.com/Masniper/splendor-client">splendor-client</a> submodule).</em>
</p>

[![Repository](https://img.shields.io/badge/parent-splendor--app-181717?logo=github)](https://github.com/Masniper/splendor-app)
[![Client](https://img.shields.io/badge/client-splendor--client-181717?logo=github)](https://github.com/Masniper/splendor-client)
[![Server](https://img.shields.io/badge/server-splendor--server-181717?logo=github)](https://github.com/Masniper/splendor-server)
![Node.js](https://img.shields.io/badge/node-%3E%3D18-339933?logo=nodedotjs&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-mixed-lightgrey)

**Splendor Online** is a browser-based multiplayer implementation of the Splendor board game. This **parent** repository ties together two [Git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules):

| Path | GitHub repository | Role |
|------|---------------------|------|
| [`front-end/`](./front-end) | [**Masniper/splendor-client**](https://github.com/Masniper/splendor-client) | React + Vite client |
| [`back-end/`](./back-end) | [**Masniper/splendor-server**](https://github.com/Masniper/splendor-server) | Express + Socket.io + Prisma API |

---

## Features

- **Accounts** — Register, login, guest play, guest → full account upgrade
- **Rooms** — Create/join by code, public room browser, host controls, bets & settlement
- **Realtime game** — Full Splendor actions over Socket.io (tokens, cards, nobles, reserve, rematch)
- **Room chat** — Ephemeral in-room messages (in-memory buffer on the server), quick phrases & emoji picker on the client
- **Audio** — Procedural UI and game sound effects (Web Audio), user mute control
- **UX** — Stacked toasts, chat notifications when the game chat panel is closed, responsive layout
- **PWA** — Installable client (`vite-plugin-pwa`); see [client README](./front-end/README.md#pwa-install--add-to-home-screen)
- **Docs** — OpenAPI/Swagger UI on the backend (`/api-docs`)

---

## Tech stack

| Area | Technologies |
|------|----------------|
| **Frontend** | React 19, Vite 6, Tailwind CSS v4, Framer Motion, Socket.io client, React Router, PWA |
| **Backend** | Node.js, Express 5, Socket.io, Prisma, Zod, JWT, bcrypt |
| **Database** | **PostgreSQL** (Prisma); Docker Compose ships Postgres 16 with a named volume |
| **Tooling** | TypeScript, Vitest (backend tests) |
| **Infrastructure** | Three GitHub repositories + **submodules** ([`.gitmodules`](./.gitmodules)); **Docker Compose** (multi-stage API + Nginx static UI + Postgres) |

---

## Prerequisites

- **Node.js** ≥ 18 (LTS recommended)
- **npm** (or compatible package manager)
- **Git** with submodule support

---

## Installation & local setup

### Clone this repository (with submodules)

```bash
git clone --recurse-submodules https://github.com/Masniper/splendor-app.git
cd splendor-app
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

### Work on client or server **standalone**

You can also clone and develop each repository on its own (without this parent folder):

- Client: `git clone https://github.com/Masniper/splendor-client.git`
- Server: `git clone https://github.com/Masniper/splendor-server.git`

Use the same `npm install` / `npm run dev` steps below inside the respective folder.

### 1. Backend

You need a running **PostgreSQL** instance and `DATABASE_URL` (see [Environment variables](#environment-variables)). Quick option: start only the database from this repo’s Compose file:

```bash
docker compose up -d postgres
```

Then:

```bash
cd back-end
npm install
# Create .env with DATABASE_URL and JWT_SECRET — see below
npx prisma migrate dev
npm run dev
```

Default server URL: **`http://localhost:5001`**

### 2. Frontend

```bash
cd front-end
npm install
npm run dev
```

Default client URL: **`http://localhost:3000`**

Optional `front-end/.env` when the API is not same-origin (typical local dev):

```env
VITE_API_BASE_URL=http://localhost:5001/api
VITE_SOCKET_URL=http://localhost:5001
```

### Bumping submodule pointers (maintainers)

After you push new commits to **splendor-client** or **splendor-server**, update this parent repo so others get the same revisions:

```bash
cd front-end && git pull origin main && cd ..
cd back-end && git pull origin main && cd ..
git add front-end back-end
git commit -m "Bump client and/or server submodules"
git push
```

---

## Environment variables

Do **not** commit real secrets. Use placeholders in examples.

### Backend (`back-end/.env`)

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | **Yes** | Prisma **PostgreSQL** URL, e.g. `postgresql://USER:PASSWORD@localhost:5432/splendor?schema=public` |
| `JWT_SECRET` | Strongly recommended | Secret for signing/verifying JWTs; use a long random string in production |
| `PORT` | No | HTTP and Socket.io port (default: `5001`) |

Example:

```env
DATABASE_URL="postgresql://splendor:splendor@localhost:5432/splendor?schema=public"
JWT_SECRET="change-me-to-a-long-random-secret"
PORT=5001
```

### Frontend (`front-end/.env`, optional)

| Variable | Required | Description |
|----------|----------|-------------|
| `VITE_API_BASE_URL` | No | REST base URL. Default: `/api` (same origin). For local API on port 5001: `http://localhost:5001/api` |
| `VITE_SOCKET_URL` | No | Socket.io origin (no path). Default: browser origin. For local: `http://localhost:5001` |

More detail: [`back-end/README.md`](./back-end/README.md), [`front-end/README.md`](./front-end/README.md).

---

## Docker (full stack)

From the **parent** directory (with submodules checked out). Requires [Docker](https://docs.docker.com/get-docker/) and Docker Compose v2.

1. Copy environment template and set a strong `JWT_SECRET`:

   ```bash
   cp .env.example .env
   # edit .env — set JWT_SECRET (required for Compose)
   ```

2. Build and start **Postgres**, **API**, and **Nginx** (static client with reverse proxy to the API):

   ```bash
   docker compose up --build -d
   ```

3. Open the app at **`http://localhost:8080`** (or the host port you set in `WEB_PORT` in `.env`).

The client is built with default Vite env (**same-origin** `/api` and Socket.io), matching the Nginx routes in `front-end/nginx.docker.conf`. Swagger UI is proxied at **`/api-docs`**.

Persisted data: Docker volume **`postgres_data`**.

---

## Project structure

```
splendor-app/                 # Parent (this repo)
├── .gitmodules               # Submodule URLs for client + server
├── docker-compose.yml        # Postgres + API + web (Nginx)
├── .env.example              # Template for Docker Compose secrets
├── README.md
├── .gitignore
├── front-end/                # → submodule: splendor-client
│   ├── public/
│   │   └── images/
│   │       └── startup-bg.jpg
│   ├── src/
│   ├── package.json
│   └── ...
└── back-end/                 # → submodule: splendor-server
    ├── prisma/
    ├── src/
    ├── package.json
    └── ...
```

---

## API documentation

- **REST base** — `http://localhost:<PORT>/api`
- **Swagger UI** — `http://localhost:<PORT>/api-docs`
- **Realtime** — Socket.io on the same port as HTTP

See [`back-end/README.md`](./back-end/README.md) for route prefixes and socket chat events.

---

## License

This **parent** repository contains mostly integration docs and submodule references.

- **Client** ([splendor-client](https://github.com/Masniper/splendor-client)) — **MIT** (see `front-end/LICENSE` when submodules are checked out).
- **Server** ([splendor-server](https://github.com/Masniper/splendor-server)) — **ISC** (see `back-end/package.json`).

---

## Related documentation

- [`front-end/README.md`](./front-end/README.md) — Vite scripts, PWA, env vars, UI layout  
- [`back-end/README.md`](./back-end/README.md) — Prisma, env, Socket.io room chat reference
