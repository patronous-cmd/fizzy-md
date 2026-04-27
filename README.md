# fizzy-md - Unified Binary

> **One binary for both fizzy.do cloud and self-hosted Fizzy**

Fork of [zainfathoni/fizzy-md](https://github.com/zainfathoni/fizzy-md) with self-hosted support added.

---

## Why This Fork?

**Problem:** Original `fizzy-md` only works with fizzy.do cloud service. Self-hosted Fizzy users get 404 errors.

**Solution:** This unified binary works for BOTH:
- **Cloud mode** (default) → connects to fizzy.do
- **Self-hosted mode** → connects to your local Fizzy via wrapper

---

## Quick Start

### For Cloud Users (fizzy.do)

```bash
# No configuration needed - works like original
fizzy-md card create --title "My Card" --description "**Bold** text"
```

### For Self-Hosted Users

```bash
# Set one environment variable
export FIZZY_SELFHOST=true
export FIZZY_WRAPPER_PATH=~/.local/bin/fizzy-local

# Same commands work!
fizzy-md card create --title "My Card" --body "**Bold** text"
```

---

## Installation

### Download Binary

```bash
# Linux x86_64
curl -L https://github.com/patronous-cmd/fizzy-md/releases/latest/download/fizzy-md_Linux_x86_64.tar.gz | tar xz
sudo mv fizzy-md /usr/local/bin/
```

### Build from Source

```bash
git clone https://github.com/patronous-cmd/fizzy-md.git
cd fizzy-md
go build -o fizzy-md .
sudo mv fizzy-md /usr/local/bin/
```

---

## Self-Hosted Setup

### Step 1: Deploy Fizzy (localhost)

```bash
mkdir ~/fizzy && cd ~/fizzy
cat > docker-compose.yml << 'EOF'
services:
  web:
    image: ghcr.io/basecamp/fizzy:main
    restart: unless-stopped
    ports:
      - "127.0.0.1:3000:80"
    environment:
      - SECRET_KEY_BASE=your_secret_key
      - DISABLE_SSL=true
      - BASE_URL=http://localhost:3000
    volumes:
      - fizzy_data:/rails/storage
volumes:
  fizzy_data:
EOF
docker compose up -d
```

### Step 2: Install Wrapper Script

```bash
curl -L https://raw.githubusercontent.com/patronous-cmd/fizzy-md/master/scripts/fizzy-local > ~/.local/bin/fizzy-local
chmod +x ~/.local/bin/fizzy-local
```

### Step 3: Set Environment

```bash
export FIZZY_SELFHOST=true
export FIZZY_WRAPPER_PATH=~/.local/bin/fizzy-local
```

---

## Usage

| Mode | Env Var | Backend |
|------|---------|---------|
| Cloud | (none) | fizzy.do API |
| Self-hosted | `FIZZY_SELFHOST=true` | localhost:3000 |

### Commands (Same for Both)

```bash
fizzy-md board list
fizzy-md card list
fizzy-md card create --title "Task" --body "## Overview\n\n**Important** item"
fizzy-md card show 12
fizzy-md card move 12 "In Progress"
fizzy-md card close 12
```

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `FIZZY_SELFHOST` | For self-hosted | Set to `true` to enable self-hosted mode |
| `FIZZY_WRAPPER_PATH` | For self-hosted | Path to `fizzy-local` wrapper script |
| `FIZZY_BOARD` | Optional | Default board name |
| `FIZZY_DOCKER_CONTAINER` | Optional | Docker container name (default: fizzy-web-1) |

---

## Wrapper Script (fizzy-local)

The `fizzy-local` script translates commands to Docker Rails commands:

```bash
#!/bin/bash
# fizzy-local - Wrapper for self-hosted Fizzy
FIZZY_CONTAINER="${FIZZY_DOCKER_CONTAINER:-fizzy-web-1}"

rails_run() {
  docker exec $FIZZY_CONTAINER /rails/bin/rails runner "$1" 2>&1
}

case "$1" in
  card)
    case "$2" in
      list)
        rails_run "board = Board.first; board.cards.each { |c| puts '#' + c.number.to_s + ' | ' + c.column.name + ' | ' + c.title }"
        ;;
      create)
        # ... handles --title, --body, --column flags
        ;;
    esac
    ;;
  status)
    rails_run "puts 'User: ' + User.last.name; puts 'Board: ' + Board.first.name"
    ;;
esac
```

Full script: [scripts/fizzy-local](./scripts/fizzy-local)

---

## Markdown Support

All Markdown is converted to HTML automatically:

| Markdown | HTML Output |
|----------|-------------|
| `**bold**` | `<strong>bold</strong>` |
| `*italic*` | `<em>italic</em>` |
| `## Header` | `<h2>Header</h2>` |
| `- item` | `<ul><li>item</li></ul>` |
| `[link](url)` | `<a href="url">link</a>` |

---

## Diagnostics

```bash
# Check configuration
fizzy-md --selfhost-info

# Output (self-hosted mode):
Self-Hosted Configuration:
  Enabled: true
  Wrapper Path: ~/.local/bin/fizzy-local
  Board: AI Agents Workspace
  Docker Container: fizzy-web-1
```

---

## Comparison

| Feature | Original fizzy-md | This Fork |
|---------|-------------------|-----------|
| fizzy.do cloud | ✓ | ✓ |
| Self-hosted Fizzy | ✗ (404 errors) | ✓ |
| Markdown → HTML | ✓ | ✓ |
| One binary? | ✓ | ✓ |

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for:
- Conventional commits format
- Pull request guidelines

---

## Credits

- Original: [Zain Fathoni](https://github.com/zainfathoni/fizzy-md)
- Fizzy: [Basecamp/37signals](https://github.com/basecamp/fizzy)
- Wrapper pattern: [OpenClaw](https://openclaw.ai)

---

## License

MIT License (same as original)