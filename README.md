# Splendor Online

![Node.js](https://img.shields.io/badge/node-%3E%3D18-339933?logo=nodedotjs&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-mixed-lightgrey)

**Splendor Online** is a browser-based multiplayer implementation of the Splendor board game. This repository is a **monorepo** with a **React + Vite** client and an **Express + Socket.io + Prisma** server.

| Package | Path | Description |
|---------|------|-------------|
| **Frontend** | [`front-end/`](./front-end) | React 19 UI, real-time gameplay, room chat, procedural audio |
| **Backend** | [`back-end/`](./back-end) | REST API, JWT auth, Socket.io rooms & game sync, SQLite/Prisma |

---

## Features

- **Accounts** — Register, login, guest play, guest → full account upgrade
- **Rooms** — Create/join by code, public room browser, host controls, bets & settlement
- **Realtime game** — Full Splendor actions over Socket.io (tokens, cards, nobles, reserve, rematch)
- **Room chat** — Ephemeral in-room messages (in-memory buffer on the server), quick phrases & emoji picker on the client
- **Audio** — Procedural UI and game sound effects (Web Audio), user mute control
- **UX** — Stacked toasts, chat notifications when the game chat panel is closed, responsive layout
- **Docs** — OpenAPI/Swagger UI on the backend (`/api-docs`)

---

## Tech stack

| Area | Technologies |
|------|----------------|
| **Frontend** | React 19, Vite 6, Tailwind CSS v4, Framer Motion, Socket.io client, React Router |
| **Backend** | Node.js, Express 5, Socket.io, Prisma, Zod, JWT, bcrypt |
| **Database** | SQLite (default); schema can target PostgreSQL |
| **Tooling** | TypeScript, Vitest (backend tests) |

---

## Prerequisites

- **Node.js** ≥ 18 (LTS recommended)
- **npm** (or compatible package manager)

---

## Installation & local setup

### 1. Backend

```bash
cd back-end
npm install
# Create .env — see back-end README (DATABASE_URL, JWT_SECRET, …)
npx prisma migrate dev   # or prisma db push — see back-end README
npm run dev
```

Server default: **`http://localhost:5001`**

### 2. Frontend

```bash
cd front-end
npm install
npm run dev
```

Client default: **`http://localhost:3000`**

For local dev against a backend on another host/port, create `front-end/.env` (optional):

```env
VITE_API_BASE_URL=http://localhost:5001/api
VITE_SOCKET_URL=http://localhost:5001
```

---

## Environment variables

Do **not** commit real secrets. Summaries below; full tables live in each package README.

| Scope | Location | Highlights |
|-------|----------|------------|
| **Backend** | `back-end/.env` | `DATABASE_URL`, `JWT_SECRET`, `PORT` |
| **Frontend** | `front-end/.env` (optional) | `VITE_API_BASE_URL`, `VITE_SOCKET_URL` — defaults to same-origin in production builds |

---

## Project structure

```
splendor-app/
├── front-end/          # Vite + React client
├── back-end/           # Express + Socket.io API
└── README.md           # This file
```

---

## API & documentation

- **REST base** — `http://localhost:<PORT>/api`
- **Swagger UI** — `http://localhost:<PORT>/api-docs`
- **Realtime** — Socket.io on the same port as HTTP

Details: [`back-end/README.md`](./back-end/README.md)

---

## License

This project uses **per-package** licenses (see `front-end/package.json` and `back-end/package.json`). Frontend is **private**; backend is **ISC** unless you change it.

---

## Related

- [Frontend README](./front-end/README.md) — scripts, Vite env, UI structure  
- [Backend README](./back-end/README.md) — Prisma, env, room chat socket events
