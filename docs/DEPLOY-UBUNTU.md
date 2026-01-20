# Deploying Gastown on Ubuntu 25.04

This guide covers deploying `gt` on a clean Ubuntu 25.04 (Plucky Puffin) server.

## Prerequisites

- Ubuntu 25.04 (amd64)
- SSH access with sudo privileges
- GitHub account for `gh` authentication
- Anthropic API key for Claude Code

## Quick Install

```bash
# Update system
sudo apt update

# Install core dependencies
sudo apt install -y \
    nodejs npm \
    golang-go \
    gh \
    unzip \
    ffmpeg 7zip jq poppler-utils fd-find ripgrep fzf zoxide imagemagick

# Install yazi (file manager)
YAZI_VERSION=$(curl -s https://api.github.com/repos/sxyazi/yazi/releases/latest | grep tag_name | cut -d\" -f4)
curl -sL "https://github.com/sxyazi/yazi/releases/download/${YAZI_VERSION}/yazi-x86_64-unknown-linux-gnu.zip" -o /tmp/yazi.zip
cd /tmp && unzip -o yazi.zip
sudo mv yazi-x86_64-unknown-linux-gnu/yazi /usr/local/bin/
sudo mv yazi-x86_64-unknown-linux-gnu/ya /usr/local/bin/
rm -rf yazi.zip yazi-x86_64-unknown-linux-gnu

# Install beads (bd)
curl -sL "https://github.com/steveyegge/beads/releases/download/v0.47.1/beads_0.47.1_linux_amd64.tar.gz" -o /tmp/beads.tar.gz
cd /tmp && tar xzf beads.tar.gz
sudo mv bd /usr/local/bin/
rm -rf beads.tar.gz LICENSE README.md

# Install Claude Code CLI
sudo npm install -g @anthropic-ai/claude-code

# Build gt from source
mkdir -p ~/projects && cd ~/projects
git clone https://github.com/joyshmitz/gastown.git
cd gastown
make build
sudo mv gt /usr/local/bin/
```

## Authentication Setup

### GitHub CLI

```bash
gh auth login
```

Follow the interactive prompts to authenticate.

### Claude Code

```bash
claude
```

Follow the prompts to authenticate with your Anthropic API key or OAuth.

## Verification

```bash
# Check all tools
node -v          # v20.x
npm -v           # 9.x
go version       # go1.24.x
gh --version     # 2.46.x
yazi --version   # 26.x
bd --version     # 0.47.x
claude --version # 2.x
gt version       # current version
```

## Optional: Yazi Configuration

Copy your local yazi config to the server:

```bash
# From local machine
scp -r ~/.config/yazi/* server:~/.config/yazi/
```

## Web Dashboard

Start the gt web dashboard:

```bash
gt dashboard --port 8080
```

Access at `http://server-ip:8080`

## Updating gt

### From fork (origin)

```bash
cd ~/projects/gastown
git checkout main
git pull origin main
make build
sudo mv gt /usr/local/bin/
```

### Sync with upstream (steveyegge/gastown)

```bash
cd ~/projects/gastown

# Add upstream if not exists
git remote add upstream https://github.com/steveyegge/gastown.git 2>/dev/null || true

# Sync
git fetch upstream
git checkout main
git merge upstream/main --ff-only
git push origin main

# Rebuild
make build
sudo mv gt /usr/local/bin/
```

## Installed Versions (Reference)

| Tool | Version | Source |
|------|---------|--------|
| Node.js | 20.18.1 | apt |
| npm | 9.2.0 | apt |
| Go | 1.24.2 | apt |
| gh | 2.46.0 | apt |
| yazi | 26.1.4 | GitHub releases |
| beads (bd) | 0.47.1 | GitHub releases |
| Claude Code | 2.1.12 | npm |
| gt | latest | Built from source |

## Troubleshooting

### fd command not found

On Ubuntu, `fd` is installed as `fdfind`. Create an alias:

```bash
echo 'alias fd=fdfind' >> ~/.bashrc
```

### Go modules download slow

Set Go proxy:

```bash
go env -w GOPROXY=https://proxy.golang.org,direct
```
