# asana2fizzy

Migrate Asana Tasks to Fizzy Cards - a command-line tool that transfers tasks from Asana to [Fizzy](https://fizzy.do) with full comment history preservation.

## Features

- Migrate Asana tasks to Fizzy cards with title and description
- Convert Asana tags to Fizzy tags (auto-created)
- Preserve all comments with author attribution and timestamps
- Filter tasks by completion status
- Migrate specific tasks by GID
- Dry-run mode to preview migrations
- Post-migration action: mark tasks complete in Asana

## Prerequisites

### Required Tools

1. **curl** - HTTP client (usually pre-installed on macOS/Linux)
```bash
curl --version
```

2. **Fizzy CLI (`fizzy`)** - [Installation](https://github.com/robzolkos/fizzy-cli)
```bash
fizzy --version
fizzy auth login YOUR_TOKEN
```

3. **jq** - JSON processor
```bash
# macOS
brew install jq

# Linux
apt-get install jq  # Debian/Ubuntu
yum install jq      # RHEL/CentOS
```

### Optional Tools

4. **cmark** or **pandoc** - Markdown to HTML conversion

Asana task notes use Markdown formatting. For proper rendering in Fizzy, install one of these converters:
```bash
# cmark (recommended - lightweight)
# macOS
brew install cmark

# Linux
apt-get install cmark  # Debian/Ubuntu

# OR pandoc (full-featured)
# macOS
brew install pandoc

# Linux
apt-get install pandoc
```

If neither is installed, content will be migrated as plain text (Markdown syntax visible but not rendered).

### Environment Variables

```bash
# Required
export ASANA_TOKEN="YOUR_ASANA_TOKEN"

# Optional
export FIZZY_ACCOUNT="YOUR_ACCOUNT_ID"
```

Or use a `.env` file (see [Configuration](#configuration)).

## Installation

```bash
# Clone the repository
git clone https://github.com/robzolkos/asana2fizzy.git
cd asana2fizzy
chmod +x asana2fizzy

# Optionally, move to a directory in your PATH
cp asana2fizzy /usr/local/bin/
```

## Configuration

Create a `.env` file in the script directory:

```bash
cp .env.example .env
# Edit .env with your Asana token
```

The `.env` file format:
```
ASANA_TOKEN="your_asana_token_here"
FIZZY_ACCOUNT="your_fizzy_account_id"  # optional
```

Get your Asana token from: https://app.asana.com/0/my-apps

## Quick Start

```bash
# Migrate all incomplete tasks from a project
./asana2fizzy -p PROJECT_GID --board BOARD_ID

# Preview what would be migrated (dry run)
./asana2fizzy -p PROJECT_GID --board BOARD_ID --dry-run

# Migrate with verbose output
./asana2fizzy -p PROJECT_GID --board BOARD_ID --verbose
```

## Usage

```
asana2fizzy --project PROJECT_GID --board BOARD_ID [OPTIONS]
```

### Required Arguments

| Argument | Description |
|----------|-------------|
| `--project, -p PROJECT_GID` | Asana project GID (required unless using `--task`) |
| `--board BOARD_ID` | Fizzy board ID to create cards in |

### Asana Options

| Argument | Short | Description | Default |
|----------|-------|-------------|---------|
| `--task` | `-t` | Specific task GID (repeatable, alternative to `--project`) | None |
| `--completed` | | Include completed tasks | Only incomplete |
| `--limit` | | Maximum tasks to migrate | 100 |

### Fizzy Options

| Argument | Description | Default |
|----------|-------------|---------|
| `--account` | Fizzy account ID | `$FIZZY_ACCOUNT` |
| `--column` | Place cards in specific column | Triage |

### Behavior Options

| Argument | Short | Description |
|----------|-------|-------------|
| `--dry-run` | | Preview without making changes |
| `--verbose` | `-v` | Show detailed output |
| `--quiet` | `-q` | Suppress non-error output |
| `--complete-tasks` | | Mark Asana tasks complete after migration |

## Examples

### Basic Migration

```bash
# Migrate all incomplete tasks
./asana2fizzy -p 1234567890 --board abc123

# Include completed tasks
./asana2fizzy -p 1234567890 --board abc123 --completed
```

### Specific Tasks

```bash
# Migrate specific tasks by GID
./asana2fizzy -p 1234567890 --board abc123 -t 111111 -t 222222 -t 333333
```

### With Post-Migration Actions

```bash
# Migrate and mark tasks complete in Asana
./asana2fizzy -p 1234567890 --board abc123 --complete-tasks
```

### Targeting a Specific Column

```bash
# Place migrated cards in a specific column
./asana2fizzy -p 1234567890 --board abc123 --column col_456
```

### Dry Run (Preview)

```bash
# See what would be migrated without making changes
./asana2fizzy -p 1234567890 --board abc123 --dry-run --verbose
```

## Data Mapping

| Asana | Fizzy |
|-------|-------|
| Task name | Card title |
| Task notes | Card description |
| Task tags | Card tags (auto-created) |
| Task comments | Single card comment with attribution |

### Comment Format

Asana comments are combined into a single Fizzy comment:

```
**Migrated Comments from Asana**

---
**John Smith** commented on 2024-01-15 10:30:

Original comment text here...

---
**Jane Doe** commented on 2024-01-16 14:22:

Another comment with full attribution...
```

## Finding Asana GIDs

### From URLs

Asana URLs contain the GIDs:
```
https://app.asana.com/1/WORKSPACE_GID/project/PROJECT_GID/...
```

### Using curl

```bash
# List your workspaces
curl -s "https://app.asana.com/api/1.0/workspaces" \
  -H "Authorization: Bearer $ASANA_TOKEN" | jq '.data[] | {gid, name}'

# List projects in a workspace
curl -s "https://app.asana.com/api/1.0/workspaces/WORKSPACE_GID/projects" \
  -H "Authorization: Bearer $ASANA_TOKEN" | jq '.data[] | {gid, name}'
```

## Finding Fizzy Board and Column IDs

```bash
# List all boards
fizzy board list

# List columns for a board
fizzy column list --board BOARD_ID
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 2 | Invalid arguments |
| 3 | Missing dependencies |
| 4 | Authentication error |
| 5 | Partial failure (some tasks failed) |

## Troubleshooting

### "ASANA_TOKEN environment variable not set"

```bash
# Set via environment
export ASANA_TOKEN="your_token_here"

# Or create a .env file
cp .env.example .env
# Edit .env with your token
```

Get your token from: https://app.asana.com/0/my-apps

### "Fizzy CLI not authenticated"

```bash
# Get token from https://app.fizzy.do/my/profile
fizzy auth login YOUR_TOKEN
fizzy identity show  # Verify
```

### "Missing required tools"

Install the missing tools listed in the error message. See [Prerequisites](#prerequisites).

### "No tasks found matching the criteria"

- Verify the project GID is correct
- Check if tasks are completed (use `--completed` to include them)
- Ensure your Asana token has access to the project

## License

MIT
