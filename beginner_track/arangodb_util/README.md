# ArangoDB Setup Scripts

This folder contains cross-platform setup scripts to initialize ArangoDB containers with a database and user account.

## Files

- **setup-arango.sh** - Bash script for Linux/macOS
- **setup-arango.ps1** - PowerShell script for Windows

## What These Scripts Do

1. Wait for ArangoDB container to be healthy
2. Create a new database (default: `mcp_arangodb_test`)
3. Create a database user (default: `mcp_arangodb_user`)
4. Grant read/write permissions to the user
5. Optionally seed sample data

## Usage

### Linux/macOS (Bash)

Make the script executable:
```bash
chmod +x setup-arango.sh
```

Run with defaults:
```bash
./setup-arango.sh
```

Run with custom parameters:
```bash
./setup-arango.sh \
  --root-password "my-root-password" \
  --db-name "my_database" \
  --username "my_user" \
  --password "my_password" \
  --seed
```

Get help:
```bash
./setup-arango.sh --help
```

### Windows (PowerShell)

Run with defaults:
```powershell
.\setup-arango.ps1
```

Run with custom parameters:
```powershell
.\setup-arango.ps1 `
  -RootPassword "my-root-password" `
  -DbName "my_database" `
  -User "my_user" `
  -Password "my_password" `
  -Seed
```

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `root-password` / `-RootPassword` | `changeme` | ArangoDB root password (set in docker-compose.yml) |
| `db-name` / `-DbName` | `mcp_arangodb_test` | Database name to create |
| `username` / `-User` | `mcp_arangodb_user` | Database user to create |
| `password` / `-Password` | `mcp_arangodb_password` | User password |
| `seed` / `-Seed` | false | Whether to seed with sample data |

## Prerequisites

1. Docker and Docker Compose installed
2. ArangoDB container running: `docker compose up -d`
3. Container must be named `mcp_arangodb_test`

## Example Workflow

### 1. Start ArangoDB Container
```bash
docker compose up -d
```

### 2. Initialize Database (choose one)

**Bash (Linux/macOS):**
```bash
./arangodb_util/setup-arango.sh --seed
```

**PowerShell (Windows):**
```powershell
.\arangodb_util\setup-arango.ps1 -Seed
```

### 3. Verify Setup
```bash
python -m mcp_arangodb_async --health
```

Expected output:
```json
{"ok": true, "db": "mcp_arangodb_test", "user": "mcp_arangodb_user"}
```

### 4. Access Web UI

Open browser to: http://localhost:8529

Login with:
- Username: `root`
- Password: `changeme` (or your custom root password)

## Troubleshooting

### "Container not healthy" error

Wait a few seconds and try again. ArangoDB takes 10-15 seconds to start:
```bash
docker compose logs arangodb
docker compose ps
```

### "Failed to create database/user"

Check the ArangoDB logs:
```bash
docker compose logs arangodb
```

Verify the root password matches your docker-compose.yml:
```yaml
environment:
  ARANGO_ROOT_PASSWORD: changeme
```

### Permission denied (Linux/macOS)

Make sure the script is executable:
```bash
chmod +x setup-arango.sh
```

## Notes

- The scripts are idempotent - running them multiple times won't cause errors
- User passwords must be securely managed (don't commit to version control)
- For production use, rotate passwords and use strong credentials
- The scripts clean up temporary files automatically (bash) or from temp directory (PowerShell)
