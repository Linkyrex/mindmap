# MindMap — PostgreSQL MCP Server

A Claude MCP (Model Context Protocol) server for interacting with a PostgreSQL database containing a mindmap application.

---

## Setup

### Prerequisites

- Docker & Docker Compose
- Node.js (for running the MCP server)
- PostgreSQL (runs in Docker)

### Installation

1. **Clone or set up the repository:**
   ```bash
   git clone https://github.com/Linkyrex/mindmap
   cd mindmap
   ```

2. **Configure environment variables:**
   ```bash
   cp .env.template .env
   ```
   Update the password in `.env` to your desired value.

3. **Start the database services:**
   ```bash
   docker compose up -d
   ```

   This starts:
   - **PostgreSQL** on `localhost:5432`
   - **pgweb** (database admin UI) on `http://localhost:8081`
   - **pgbackup** (automatic hourly backups) - backups stored in `./backups/`

4. **Configure Claude Desktop:**
   Copy your `claude_desktop_config.json` to the Claude Desktop configuration directory:
   
   **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
   
   **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
   
   **Linux:** `~/.config/Claude/claude_desktop_config.json`

   The config enables the MCP server to:
   - Connect to PostgreSQL via `mcp-postgres-server`
   - Access all mindmap database tables
   - Execute queries and view schema information

5. **Restart Claude Desktop** for the MCP connection to activate.

6. **Reference in Claude Prompt** (Optional but recommended)

Add this to your Claude system prompt for enhanced integration:

```
Mindmap MCP (Postgres — persistent memory/worklog)
- READ: mindmap:query with {sql}. WRITE: mindmap:execute with parameterized SQL ($1,$2…) + params array. Never string-interpolate values.
- Discover schema first: list tables, then column names via information_schema before writing.
- BEFORE assuming any project/infra state: query mindmap first (ILIKE search on relevant keywords/hostnames).
- When asked to "save/remember something": INSERT a node with title, content, category, and timestamps; keep entries concise and self-contained.
- Mindmap is authoritative over model memory — on conflict, trust the DB.
```

---

## Usage

Once configured, you can use Claude to:

- Query the mindmap database
- Explore table schemas
- Run SQL operations
- Debug database issues
- Export data

**Example queries:**
- "What did I do on July 10th?"
- "How did I set up the MCP server in this project?"
- "Show me all notes tagged with 'deployment'"
- "Save this architecture decision to my mindmap"
- "What was the issue we had with database migrations last week?"

---

## Services

### PostgreSQL (`db`)
- Main database service
- Port: `5432`
- Auto-restart enabled

### pgweb UI (`http://localhost:8081`)
- Web-based database administration
- Zero authentication (local only)
- Auto-restart enabled

### pgbackup
- Automatic hourly backups
- Retention: 3 days
- Timezone: Asia/Bangkok

---

## Stopping Services

```bash
docker compose down
```

To stop and remove volumes:
```bash
docker compose down -v
```

---

## Troubleshooting

### MCP server connection fails
- Ensure Docker containers are running: `docker compose ps`
- Check that PostgreSQL is listening on 5432: `docker compose logs db`
- Verify the password in `claude_desktop_config.json` matches your `.env`

### Database is empty
- SQL dumps are included in `mindmap-latest.sql.gz`
- Restore with: `gunzip < mindmap-latest.sql.gz | docker compose exec -T db psql -U mindmap -d mindmap`

### Cannot access pgweb
- Check that port 8081 is not in use: `lsof -i :8081`
- Restart the service: `docker compose restart pgweb`

---

## Project Files

| File | Purpose |
|------|---------|
| `claude_desktop_config.json` | MCP server configuration for Claude Desktop |
| `compose.yml` | Docker Compose services definition |
| `.env` | Environment variables (ignored in git) |
| `.env.template` | Template for `.env` variables |
| `backups/` | Automatic database backups directory |
| `mindmap-latest.sql.gz` | SQL database dump for initialization |
| `README.md` | This file |
