# MindMap - PostgreSQL MCP Server

A Claude MCP (Model Context Protocol) server for interacting with a PostgreSQL database containing a mindmap application.

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

## Usage

Once configured, you can use Claude to:

- Query the mindmap database
- Explore table schemas
- Run SQL operations
- Debug database issues
- Export data

Example: Ask Claude "Show me the tables in the mindmap database" or "Query the latest entries from [table_name]"

## Services

### PostgreSQL (`db`)
- Main database service
- Port: 5432
- Auto-restart enabled

### pgweb (`http://localhost:8081`)
- Web UI for database administration
- No authentication required locally
- Auto-restart enabled

### pgbackup
- Automatic hourly backups
- Keeps backups for 3 days
- Timezone: Asia/Bangkok

## Stopping Services

```bash
docker compose down
```

To stop and remove volumes:
```bash
docker compose down -v
```

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

## Files

- `claude_desktop_config.json` - MCP server configuration for Claude Desktop
- `compose.yml` - Docker Compose configuration
- `.env` - Environment variables (passwords, etc.) — in .gitignore
- `.env.template` - Template for environment variables
- `backups/` - Directory containing automatic database backups
- `mindmap-latest.sql.gz` - SQL database dump for initialization
