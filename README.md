# SkillSwap

SkillSwap is a web application for exchanging skills and learning resources. Users can create profiles, list skills they can offer or want to learn, search for other users and skills, chat in real time, and share learning resources through a browser-based peer-to-peer workflow.

## Features

- Account registration and login
- User profiles with profile photo and skill lists
- Dashboard statistics and recent conversations
- Skill discovery and personal skill management
- Real-time one-to-one chat over WebSockets
- File attachments in conversations
- Skill-linked resource publishing and discovery
- Peer-to-peer resource coordination with WebSockets and WebRTC
- Resource requests, approvals, connection management, swarm statistics, and ratings

## Architecture

- **Backend:** Go HTTP server with PostgreSQL persistence
- **Frontend:** Static HTML, CSS, and browser JavaScript served by the Go server
- **Real-time communication:** WebSockets for chat and P2P coordination; WebRTC is used by the browser P2P client for direct peer connections
- **Database:** PostgreSQL 15 or later
- **Migrations:** Core, P2P, and skill-connection tables are created automatically when the backend starts

## Requirements

For a local installation:

- Go 1.24 or later (the module specifies toolchain Go 1.24.5)
- PostgreSQL 15 or later
- A modern browser with WebSocket and WebRTC support

Docker Compose is also included, but see [Docker notes](#docker-notes) before using it.

## Local Setup

1. Create the local environment file:

   ```bash
   cp .env.example .env
   ```

2. If PostgreSQL is running on the host, edit `.env` and set:

   ```dotenv
   DB_HOST=localhost
   DB_PORT=5432
   DB_USER=postgres
   DB_PASSWORD=postgres
   DB_NAME=skillswap
   ```

   Create the database if it does not already exist:

   ```bash
   createdb -U postgres skillswap
   ```

   The default credentials above are intended for local development. Use a dedicated database user and a strong password outside development.

3. Build the backend from its Go module directory:

   ```bash
   cd backend
   go mod download
   go build -o skillswap .
   cd ..
   ```

4. Start the server from the repository root:

   ```bash
   ./backend/skillswap
   ```

   Starting from the repository root is important because the server serves pages from `./frontend` and uploaded content from `./uploads` and `./p2p_resources`.

5. Open [http://localhost:8080](http://localhost:8080) and create an account.

On startup, the backend connects to PostgreSQL and creates or updates the required tables and indexes. If the database is unavailable, the process exits with a connection error.

## Docker Notes

The included `docker-compose.yml` starts PostgreSQL and builds the backend container:

```bash
docker compose up --build
```

The database service is configured for `postgres` / `postgres` and exposes port `5432`; the backend exposes port `8080`.

The current backend image only copies the contents of `backend/`. Since the server serves `frontend/` using relative paths, the Compose backend service does not currently include the frontend files and is therefore not a complete browser-serving deployment. For a working local development setup, use the [Local Setup](#local-setup) instructions. A production Docker deployment should mount or copy the repository's `frontend/` directory into the backend image and provide persistent storage for uploads and P2P resources.

To stop the Compose services:

```bash
docker compose down
```

Add `-v` only when you intentionally want to delete the PostgreSQL volume and all local database data.

## Configuration

The backend reads these environment variables, falling back to the shown defaults when they are not set:

| Variable | Default | Description |
| --- | --- | --- |
| `DB_HOST` | `localhost` | PostgreSQL hostname; use `db` inside Compose |
| `DB_PORT` | `5432` | PostgreSQL port |
| `DB_USER` | `postgres` | PostgreSQL user |
| `DB_PASSWORD` | `postgres` | PostgreSQL password |
| `DB_NAME` | `skillswap` | PostgreSQL database name |

Do not commit `.env`. Use `.env.example` as the development template.

## Application Routes

The Go server serves the main pages at:

`/login`, `/signup`, `/dashboard`, `/profile`, `/chat`, `/my-skills`, `/find-skills`, `/find-resources`, `/p2p-dashboard`, and `/manage-connections`.

The API is available under `/api`. The main groups are:

- Authentication: `/api/signup`, `/api/login`
- Profiles and discovery: `/api/profile`, `/api/users/search`, `/api/skills/*`
- Chat and files: `/api/chats`, `/api/history`, `/api/upload`, `/api/file`
- WebSockets: `/api/ws`, `/api/p2p/ws`
- P2P resources and transfers: `/api/p2p/*`
- Skill resources and connections: `/api/skills/resources`, `/api/p2p/connections/*`

The frontend API configuration is centralized in `frontend/js/api-config.js`.

## Database Migrations

The backend performs its current migrations during startup. SQL files in `backend/db/` are retained for initialization and manual migration workflows, including:

- `init.sql` for the base schema
- `migrate.sql` for updates to existing user and message columns
- `p2p_migrate.sql` and `p2p_workflow_migrate.sql` for P2P workflow schema changes
- `skill_connections_migration.sql` for skill-resource connection management

For an existing database where a manual migration is required, the repository includes platform-specific scripts:

```bash
cd backend
./run_migration.sh
```

On Windows, run `backend\\run_migration.bat` from a PostgreSQL-enabled command prompt.

## Development Checks

Format and compile the Go backend with:

```bash
cd backend
find . -name '*.go' -print0 | xargs -0 gofmt -w
go test ./...
go build ./...
```

There is currently no automated test suite configured in `package.json`; the root package includes the `ws` dependency for WebSocket-related development tooling.

## Project Layout

```text
backend/
  db/          PostgreSQL schema and migration files
  handlers/    HTTP and WebSocket handlers
  models/      Go data models
  main.go      Server routes and static file serving
frontend/      HTML pages, CSS, browser JavaScript, and static assets
docker-compose.yml
.env.example
package.json
```

## License

No project license has been declared yet.