# fizzy-md-selfhost

fork of [fizzy-md](https://github.com/zainfathoni/fizzy-md) with **self-hosted Fizzy** support.

## What's Different?

| Feature | Original fizzy-md | fizzy-md-selfhost |
|---------|-------------------|-------------------|
| **Cloud Fizzy (fizzy.do)** | ✅ Works | ✅ Works |
| **Self-Hosted Fizzy** | ❌ Not supported | ✅ Works via wrapper |
| **Markdown Conversion** | ✅ | ✅ |

## How It Works

When `FIZZY_SELFHOST=true` is set, instead of calling `fizzy-cli`, it calls a wrapper script (`fizzy-local`) that interfaces with your self-hosted Fizzy via Docker.

## Installation

### Download Binary

```bash
# Download from releases (coming soon)
curl -L https://github.com/YOUR_FORK/fizzy-md-selfhost/releases/latest/download/fizzy-md-selfhost_Linux_x86_64.tar.gz | tar xz
sudo mv fizzy-md-selfhost /usr/local/bin/
```

### Build from Source

```bash
git clone https://github.com/YOUR_FORK/fizzy-md-selfhost.git
cd fizzy-md-selfhost
go build -o fizzy-md-selfhost .
sudo mv fizzy-md-selfhost /usr/local/bin/
```

## Configuration

Set environment variables for self-hosted mode:

```bash
# Enable self-hosted mode
export FIZZY_SELFHOST=true

# Path to wrapper script (default: ~/.openclaw/workspace-hermione/fizzy/fizzy-local)
export FIZZY_WRAPPER_PATH=/path/to/fizzy-local

# Default board name (optional)
export FIZZY_BOARD="AI Agents Workspace"

# Docker container name (optional)
export FIZZY_DOCKER_CONTAINER="fizzy-web-1"
```

## Usage

### Self-Hosted Mode

```bash
# Set env vars first
export FIZZY_SELFHOST=true
export FIZZY_WRAPPER_PATH=~/.local/bin/fizzy-local

# Now use fizzy-md-selfhost like fizzy-md!
fizzy-md-selfhost status
fizzy-md-selfhost card list
fizzy-md-selfhost card create --title "My Card" --body "**Bold** text" --column "To Do"
```

### Cloud Mode (fizzy.do)

Without `FIZZY_SELFHOST=true`, it works exactly like original fizzy-md:

```bash
# Uses fizzy-cli for fizzy.do cloud service
fizzy-md-selfhost board list
fizzy-md-selfhost card create --title "Cloud Card" --description "Markdown works!"
```

## Required: Wrapper Script

You need the `fizzy-local` wrapper script for self-hosted mode. Get it from:

```bash
# Install wrapper script
curl -L https://raw.githubusercontent.com/YOUR_FORK/fizzy-md-selfhost/main/scripts/fizzy-local > ~/.local/bin/fizzy-local
chmod +x ~/.local/bin/fizzy-local
```

Or create your own wrapper that communicates with your self-hosted Fizzy instance.

## Example Workflow

```bash
# Set up environment
export FIZZY_SELFHOST=true
export FIZZY_WRAPPER_PATH=~/.local/bin/fizzy-local

# Create a card with Markdown
fizzy-md-selfhost card create \
  --title "Bug Fix: Login Issue" \
  --body "## Problem

Users can't log in when password contains \`&\` character.

## Root Cause

- URL encoding not applied
- Special chars break form submission

## Fix

Added \`encodeURIComponent()\` to password field.

**Status:** ✅ Resolved" \
  --column "To Do"

# Move card to In Progress
fizzy-md-selfhost card move 14 "In Progress"

# Close when done
fizzy-md-selfhost card close 14
```

## Credits

- Original: [zainfathoni/fizzy-md](https://github.com/zainfathoni/fizzy-md) by Zain Fathoni
- Fizzy CLI: [basecamp/fizzy-cli](https://github.com/basecamp/fizzy-cli) by Basecamp/37signals
- Fizzy: [basecamp/fizzy](https://github.com/basecamp/fizzy) - Self-hosted Kanban by 37signals

## License

MIT License (same as original fizzy-md)