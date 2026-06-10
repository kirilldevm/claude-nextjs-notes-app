# NextNotes

A note-taking web app where authenticated users can create, edit, and delete rich-text notes, and optionally share them via a public URL.

## Features

- **Authentication** — Email/password sign up, login, and logout via [better-auth](https://www.better-auth.com/)
- **Rich text editing** — TipTap editor with bold, italic, headings (H1–H3), bullet lists, inline code, code blocks, and horizontal rules
- **Note management** — Create, view, edit, and delete notes from a personal dashboard
- **Public sharing** — Toggle a note public to get a shareable link at `/p/{slug}`; disable sharing to revoke access

## Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js 16 (App Router) |
| Runtime | [Bun](https://bun.sh/) |
| Language | TypeScript |
| Styling | Tailwind CSS 4 |
| Database | SQLite via Bun's native SQLite client |
| Auth | better-auth |
| Editor | TipTap |
| Validation | Zod |

## Prerequisites

- [Bun](https://bun.sh/) 1.x

## Getting Started

### 1. Install dependencies

```bash
bun install
```

### 2. Configure environment variables

Create a `.env.local` file in the project root:

```env
BETTER_AUTH_SECRET=your-random-secret-at-least-32-chars
BETTER_AUTH_URL=http://localhost:3000
```

`BETTER_AUTH_SECRET` is required. Generate one with:

```bash
openssl rand -base64 32
```

`BETTER_AUTH_URL` should match the URL where the app is served. For local development, `http://localhost:3000` is fine.

Optional:

```env
DB_PATH=data/app.db
```

Defaults to `data/app.db` if not set. The database file and schema are created automatically on first run.

### 3. Start the development server

```bash
bun dev
```

Open [http://localhost:3000](http://localhost:3000).

## Scripts

| Command | Description |
|---------|-------------|
| `bun dev` | Start development server |
| `bun run build` | Build for production |
| `bun start` | Start production server |
| `bun run lint` | Run ESLint |
| `bun run format` | Format code with oxfmt |
| `bun test` | Run tests in watch mode |
| `bun run test:run` | Run tests once |

## Routes

| Route | Access | Description |
|-------|--------|-------------|
| `/` | Public | Landing page |
| `/authenticate` | Public | Login and sign up |
| `/dashboard` | Authenticated | List of your notes |
| `/notes/new` | Authenticated | Create a new note |
| `/notes/[id]` | Authenticated (owner) | View a note |
| `/notes/[id]/edit` | Authenticated (owner) | Edit a note |
| `/p/[slug]` | Public | Read-only view of a shared note |

## Project Structure

```
app/
  api/auth/          # better-auth API handler
  authenticate/      # Login / sign up page
  dashboard/         # Notes list
  notes/             # Note CRUD pages and server actions
  p/[slug]/          # Public shared note page
components/          # UI components (editor, header, share toggle, etc.)
lib/
  auth.ts            # better-auth server config
  auth-client.ts     # better-auth React client
  db.ts              # SQLite connection and schema initialization
  content.ts         # TipTap content helpers
  sanitize.ts        # HTML sanitization
  validation.ts      # Zod schemas
__tests__/           # Vitest unit tests
```

## How It Works

- **Data storage** — Notes are stored as TipTap JSON in SQLite. The schema (auth tables + `notes`) is initialized in `lib/db.ts` when the app starts.
- **Mutations** — Note create, update, delete, and sharing use Next.js Server Actions rather than REST endpoints.
- **Authorization** — All note queries filter by `user_id`. Public notes are only accessible when `is_public = 1` and a valid `public_slug` is set.
- **Rendering** — Shared and read-only views render TipTap JSON through a dedicated renderer with sanitization.

## Testing

```bash
bun run test:run
```

Tests cover content parsing, validation schemas, and the TipTap renderer component.

## Production

```bash
bun run build
bun start
```

Set `BETTER_AUTH_URL` to your production domain. Ensure the `data/` directory (or your `DB_PATH` location) is writable and persisted across deploys.
