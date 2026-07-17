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

4. **Configure Claude Desktop:** Copy your `claude_desktop_config.json` to the Claude Desktop configuration directory:

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

## Offsite Backup (restic) — optional

Encrypted, deduplicated offsite copies of the local `pgbackup` dumps via [restic](https://restic.net) over SFTP, using the [mazzolino/restic](https://github.com/djmaze/resticker) scheduler image. Defined separately in `compose.restic.yml` so it's opt-in.

### Prerequisites

- An SFTP-only user on the remote host, restricted via `ForceCommand internal-sftp` in `sshd_config`
- An SSH keypair authorized for that user
- A restic repository password
- A `.ssh` helper directory containing `known_hosts` and an ssh `config` pointing at the mounted key (kept outside the repo, not committed)

### Setup

1. Init the repo once:

   ```bash
   restic -r sftp:USER@HOST:/path/to/repo --password-file /path/to/password init
   ```

2. Prepare the ssh helper dir outside the repo:

   ```bash
   mkdir -p ~/.config/mindmap-restic
   ssh-keyscan -H HOST > ~/.config/mindmap-restic/known_hosts
   cat > ~/.config/mindmap-restic/config <<'EOF'
   Host *
       User USER
       IdentityFile /run/secrets/.ssh/sangat
       StrictHostKeyChecking yes
   EOF
   ```

3. Add to `.env`:

   ```
   RESTIC_REPOSITORY=sftp:USER@HOST:/path/to/repo
   RESTIC_SSH_DIR=/absolute/path/to/mindmap-restic
   RESTIC_SSH_KEY=/absolute/path/to/private/key
   RESTIC_PASSWORD_FILE=/absolute/path/to/password/file
   ```

4. Start:

   ```bash
   docker compose -f compose.yml -f compose.restic.yml up -d
   ```

### Schedule

- `restic-backup`: hourly
- `restic-prune`: daily at 02:00, `--keep-daily 7 --keep-weekly 4`

---

## Stopping Services

```bash
docker compose down
```

To stop and remove volumes:

```bash
docker compose down -v
```

If running with the restic add-on:

```bash
docker compose -f compose.yml -f compose.restic.yml down
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

| File                         | Purpose                                          |
| ----------------------------- | ------------------------------------------------ |
| `claude_desktop_config.json`  | MCP server configuration for Claude Desktop      |
| `compose.yml`                 | Docker Compose services definition               |
| `compose.restic.yml`          | Optional offsite backup services (restic)        |
| `.env`                        | Environment variables (ignored in git)           |
| `.env.template`               | Template for `.env` variables                    |
| `backups/`                    | Automatic database backups directory             |
| `README.md`                   | This file                                        |
