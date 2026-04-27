# fizzy-md Self-Hosted Support

> Fork of [zainfathoni/fizzy-md](https://github.com/zainfathoni/fizzy-md) with **self-hosted Fizzy** support

---

## 🎯 What This Does

**Problem:** `fizzy-cli` and `fizzy-md` are designed for [fizzy.do](https://fizzy.do) cloud service only. They cannot connect to self-hosted Fizzy instances.

**Solution:** This fork adds self-hosted support via a wrapper script that communicates directly with your local Fizzy Docker container.

---

## 📊 Comparison

| Feature | Original fizzy-md | This Fork |
|---------|-------------------|-----------|
| **fizzy.do Cloud** | ✅ Works | ✅ Works |
| **Self-Hosted Fizzy** | ❌ 404 errors | ✅ Works |
| **Markdown → HTML** | ✅ | ✅ |
| **All fizzy commands** | ✅ | ✅ |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     YOUR MACHINE                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   fizzy-md-selfhost                                          │
│   ├── FIZZY_SELFHOST=false → fizzy-cli → fizzy.do (cloud)   │
│   └── FIZZY_SELFHOST=true  → fizzy-local → Docker container │
│                                                              │
│   fizzy-local (wrapper script)                               │
│   └── docker exec fizzy-web-1 rails runner "..."            │
│                                                              │
│   Docker Container: fizzy-web-1                              │
│   └── Fizzy Rails App (localhost:3000)                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 Prerequisites

Before using this fork, you need:

1. **Self-hosted Fizzy** running on localhost
   - See: [basecamp/fizzy](https://github.com/basecamp/fizzy)
   - Docker deployment recommended

2. **Go 1.23+** (for building)

3. **Docker** with Fizzy container running

---

## 🚀 Quick Start

### Step 1: Deploy Self-Hosted Fizzy

```bash
# Create docker-compose.yml
mkdir -p ~/fizzy && cd ~/fizzy

cat > docker-compose.yml << 'EOF'
services:
  web:
    image: ghcr.io/basecamp/fizzy:main
    restart: unless-stopped
    ports:
      - "127.0.0.1:3000:80"  # localhost only
    environment:
      - SECRET_KEY_BASE=your_secret_key_here
      - DISABLE_SSL=true
      - BASE_URL=http://localhost:3000
      - MAILER_FROM_ADDRESS=fizzy@localhost
      # SMTP (optional)
      - SMTP_ADDRESS=smtp.example.com
      - SMTP_PORT=465
      - SMTP_USERNAME=your_username
      - SMTP_PASSWORD=your_password
      - SMTP_TLS=true
    volumes:
      - fizzy_data:/rails/storage

volumes:
  fizzy_data:
EOF

# Start Fizzy
docker compose up -d
```

### Step 2: Create Fizzy Account

```bash
# Create user account via Rails
docker exec fizzy-web-1 /rails/bin/rails runner "
user = User.create!(name: 'Your Name')
identity = Identity::EmailAddress.create!(email: 'your@email.com', user: user)
account = Account.create!(name: 'My Workspace')
Access.create!(user: user, account: account, level: 'owner')
puts 'Account created: ' + account.slug
"

# Generate API token
docker exec fizzy-web-1 /rails/bin/rails runner "
identity = User.last.identity
token = Identity::AccessToken.create!(
  identity: identity,
  description: 'CLI Token',
  permission: 'write'
)
puts 'Token: ' + token.token
"
```

### Step 3: Install fizzy-md-selfhost

```bash
# Clone this fork
git clone https://github.com/patronous-cmd/fizzy-md.git
cd fizzy-md

# Build
go build -o fizzy-md-selfhost .

# Install
sudo mv fizzy-md-selfhost /usr/local/bin/
```

### Step 4: Install Wrapper Script

```bash
# Download wrapper
curl -L https://raw.githubusercontent.com/patronous-cmd/fizzy-md/main/scripts/fizzy-local > ~/.local/bin/fizzy-local
chmod +x ~/.local/bin/fizzy-local

# Or create manually (see below)
```

### Step 5: Configure Environment

```bash
# Add to ~/.bashrc or ~/.zshrc
export FIZZY_SELFHOST=true
export FIZZY_WRAPPER_PATH=~/.local/bin/fizzy-local
export FIZZY_BOARD="My Workspace"  # optional
export FIZZY_DOCKER_CONTAINER="fizzy-web-1"  # optional
```

### Step 6: Test

```bash
# Check status
fizzy-md-selfhost status

# List cards
fizzy-md-selfhost card list
```

---

## 🔧 Wrapper Script: fizzy-local

The wrapper script (`fizzy-local`) translates fizzy-md commands to Rails commands that run inside your Docker container.

### Create Your Own Wrapper

Save this as `~/.local/bin/fizzy-local`:

```bash
#!/bin/bash
# fizzy-local - Wrapper for self-hosted Fizzy
# Translates fizzy commands to Docker Rails commands

set -e

FIZZY_CONTAINER="${FIZZY_DOCKER_CONTAINER:-fizzy-web-1}"
BOARD_NAME="${FIZZY_BOARD:-AI Agents Workspace}"

rails_run() {
    docker exec $FIZZY_CONTAINER /rails/bin/rails runner "$1" 2>&1
}

CMD="$1"
shift

case "$CMD" in
    status)
        rails_run "
user = User.last
account = Account.last
board = Board.find_by(name: '$BOARD_NAME') || Board.first
puts 'User: ' + user.name
puts 'Account: ' + account.name
puts 'Board: ' + board.name + ' (' + board.cards.count.to_s + ' cards)'
"
        ;;
    
    board)
        case "$1" in
            list)
                rails_run "
Board.all.each { |b| puts b.name + ' (' + b.cards.count.to_s + ' cards)' }
"
                ;;
            show)
                rails_run "
board = Board.find_by(name: '$BOARD_NAME') || Board.first
puts 'Board: ' + board.name
puts 'Columns: ' + board.columns.map(&:name).join(', ')
"
                ;;
        esac
        ;;
    
    card)
        case "$1" in
            list)
                rails_run "
board = Board.find_by(name: '$BOARD_NAME') || Board.first
board.cards.each do |c|
  status = c.closed_at ? 'CLOSED' : 'OPEN'
  puts '#' + c.number.to_s + ' | ' + status + ' | ' + c.column.name + ' | ' + c.title
end
"
                ;;
            
            create)
                TITLE=""
                BODY=""
                COLUMN="To Do"
                
                while [[ $# -gt 0 ]]; do
                    case "$1" in
                        --title|-t) TITLE="$2"; shift 2 ;;
                        --body|-b) BODY="$2"; shift 2 ;;
                        --column|-c) COLUMN="$2"; shift 2 ;;
                        *) shift ;;
                    esac
                done
                
                rails_run "
board = Board.find_by(name: '$BOARD_NAME') || Board.first
column = board.columns.find_by(name: '$COLUMN') || board.columns.first
user = User.last
card = Card.create!(board: board, column: column, title: '$TITLE', description: '$BODY', creator: user)
puts 'Created: #' + card.number.to_s + ' - ' + card.title
"
                ;;
            
            show)
                NUM="$2"
                rails_run "
board = Board.find_by(name: '$BOARD_NAME') || Board.first
card = board.cards.find_by(number: $NUM)
puts '#' + card.number.to_s + ' [' + card.column.name + '] ' + card.title
"
                ;;
            
            move)
                NUM="$2"
                COL="$3"
                rails_run "
board = Board.find_by(name: '$BOARD_NAME') || Board.first
card = board.cards.find_by(number: $NUM)
column = board.columns.find_by(name: '$COL')
card.update!(column: column)
puts 'Moved #' + card.number.to_s + ' to: ' + column.name
"
                ;;
            
            close)
                NUM="$2"
                rails_run "
board = Board.find_by(name: '$BOARD_NAME') || Board.first
card = board.cards.find_by(number: $NUM)
card.update!(closed_at: Time.current)
puts 'Closed #' + card.number.to_s
"
                ;;
            
            reopen)
                NUM="$2"
                rails_run "
board = Board.find_by(name: '$BOARD_NAME') || Board.first
card = board.cards.find_by(number: $NUM)
card.update!(closed_at: nil)
puts 'Reopened #' + card.number.to_s
"
                ;;
        esac
        ;;
    
    --help|-h|help)
        echo "fizzy-local - Self-hosted Fizzy wrapper"
        echo ""
        echo "Commands:"
        echo "  status              Show system status"
        echo "  board list          List all boards"
        echo "  board show          Show current board"
        echo "  card list           List all cards"
        echo "  card create         Create new card"
        echo "  card show <num>     Show card details"
        echo "  card move <num> <col> Move card"
        echo "  card close <num>    Close card"
        echo "  card reopen <num>   Reopen card"
        ;;
    
    *)
        echo "Unknown command: $CMD"
        echo "Run: fizzy-local help"
        ;;
esac
```

---

## 📝 Available Commands

### System Commands

| Command | Description |
|---------|-------------|
| `fizzy-md-selfhost status` | Show user, account, board info |
| `fizzy-md-selfhost --selfhost-info` | Show self-host config |
| `fizzy-md-selfhost --version` | Show version |

### Board Commands

| Command | Description |
|---------|-------------|
| `fizzy-md-selfhost board list` | List all boards |
| `fizzy-md-selfhost board show` | Show board details |

### Card Commands

| Command | Description |
|---------|-------------|
| `fizzy-md-selfhost card list` | List all cards |
| `fizzy-md-selfhost card create --title "X" --body "Y"` | Create card |
| `fizzy-md-selfhost card show <num>` | Show card |
| `fizzy-md-selfhost card move <num> <column>` | Move card |
| `fizzy-md-selfhost card close <num>` | Close card |
| `fizzy-md-selfhost card reopen <num>` | Reopen card |

---

## 🎨 Markdown Support

fizzy-md-selfhost automatically converts Markdown to HTML:

| Markdown | HTML Output |
|----------|-------------|
| `**bold**` | `<strong>bold</strong>` |
| `*italic*` | `<em>italic</em>` |
| `## Header` | `<h2>Header</h2>` |
| `- item` | `<ul><li>item</li></ul>` |
| `[link](url)` | `<a href="url">link</a>` |
| `| table |` | `<table>...</table>` |

### Example

```bash
fizzy-md-selfhost card create \
  --title "Bug Report" \
  --body "## Problem

Users can't login.

## Steps to Reproduce

1. Open app
2. Enter credentials
3. Click login

**Status:** 🔴 Critical" \
  --column "To Do"
```

---

## 🔀 Switching Between Cloud & Self-Hosted

### Use Self-Hosted

```bash
export FIZZY_SELFHOST=true
fizzy-md-selfhost card list  # → connects to localhost:3000
```

### Use Cloud (fizzy.do)

```bash
export FIZZY_SELFHOST=false
# or unset it
unset FIZZY_SELFHOST

fizzy-md-selfhost card list  # → connects to fizzy.do
```

---

## 🐛 Troubleshooting

### "Container not found"

```bash
# Check container is running
docker ps | grep fizzy

# Start if stopped
cd ~/fizzy && docker compose up -d
```

### "Board not found"

```bash
# List available boards
docker exec fizzy-web-1 /rails/bin/rails runner "Board.all.each { |b| puts b.name }"

# Set correct board name
export FIZZY_BOARD="Your Board Name"
```

### "Wrapper not found"

```bash
# Check wrapper path
which fizzy-local

# Set correct path
export FIZZY_WRAPPER_PATH=/path/to/fizzy-local
```

### Check Self-Host Config

```bash
fizzy-md-selfhost --selfhost-info
```

Output:
```
Self-Hosted Configuration:
  Enabled: true
  Wrapper Path: ~/.local/bin/fizzy-local
  API URL: http://localhost:3000
  Board: AI Agents Workspace
  Docker Container: fizzy-web-1
```

---

## 🔗 Related Projects

| Project | URL | Description |
|---------|-----|-------------|
| **Fizzy** | [basecamp/fizzy](https://github.com/basecamp/fizzy) | Self-hosted Kanban by 37signals |
| **fizzy-cli** | [basecamp/fizzy-cli](https://github.com/basecamp/fizzy-cli) | Official CLI for fizzy.do |
| **fizzy-md** | [zainfathoni/fizzy-md](https://github.com/zainfathoni/fizzy-md) | Markdown wrapper for fizzy-cli |

---

## 🙏 Credits

- **Original fizzy-md:** [Zain Fathoni](https://github.com/zainfathoni)
- **Fizzy & fizzy-cli:** [Basecamp/37signals](https://github.com/basecamp)
- **Self-hosted support:** [patronous-cmd](https://github.com/patronous-cmd)

---

## 📄 License

MIT License (same as original fizzy-md)