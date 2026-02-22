# CLI Guide

Complete guide to the ResonantGenesis command-line interface, CLI commands, and terminal usage.

## Overview

The ResonantGenesis CLI provides a powerful command-line interface for managing agents, sessions, and platform resources directly from your terminal. This guide covers installation, configuration, and all available commands.

## Installation

### macOS / Linux

```bash
# Using curl
curl -fsSL https://cli.resonantgenesis.xyz/install.sh | bash

# Using Homebrew
brew install resonantgenesis/tap/rg

# Using pip
pip install resonantgenesis-cli
```

### Windows

```powershell
# Using PowerShell
iwr https://cli.resonantgenesis.xyz/install.ps1 -useb | iex

# Using Chocolatey
choco install resonantgenesis-cli

# Using pip
pip install resonantgenesis-cli
```

### Verify Installation

```bash
rg --version
# resonantgenesis-cli v1.0.0
```

## Configuration

### Authentication

```bash
# Login with API key
rg auth login --api-key YOUR_API_KEY

# Login interactively
rg auth login
# Enter API key: ****

# Login with OAuth
rg auth login --oauth

# Check authentication status
rg auth status
# Authenticated as: user@example.com
# Organization: Acme Corp
# API Key: rg_****...****
```

### Configuration File

```bash
# View current config
rg config list

# Set configuration
rg config set default_model gpt-4-turbo
rg config set output_format json
rg config set timeout 60

# Get specific config
rg config get default_model

# Reset to defaults
rg config reset
```

### Configuration File Location

| OS | Path |
|----|------|
| macOS | `~/.config/resonantgenesis/config.yaml` |
| Linux | `~/.config/resonantgenesis/config.yaml` |
| Windows | `%APPDATA%\resonantgenesis\config.yaml` |

### Example Config File

```yaml
# ~/.config/resonantgenesis/config.yaml
api_key: rg_****
default_model: gpt-4-turbo
output_format: table
timeout: 60
color: true
verbose: false
```

## Agent Commands

### List Agents

```bash
# List all agents
rg agents list

# List with filters
rg agents list --status active --limit 10

# Output as JSON
rg agents list --output json

# Output as table
rg agents list --output table
```

### Create Agent

```bash
# Create agent interactively
rg agents create

# Create with options
rg agents create \
  --name "My Agent" \
  --system-prompt "You are a helpful assistant." \
  --model gpt-4-turbo \
  --tools web_search,code_exec

# Create from file
rg agents create --from-file agent.yaml
```

### Get Agent

```bash
# Get agent details
rg agents get agent_123

# Get specific fields
rg agents get agent_123 --fields name,status,model
```

### Update Agent

```bash
# Update agent
rg agents update agent_123 \
  --name "Updated Name" \
  --model gpt-4-turbo

# Update from file
rg agents update agent_123 --from-file agent.yaml
```

### Delete Agent

```bash
# Delete agent
rg agents delete agent_123

# Delete with confirmation skip
rg agents delete agent_123 --yes

# Delete multiple
rg agents delete agent_123 agent_456 agent_789
```

### Execute Agent

```bash
# Execute agent
rg agents execute agent_123 --goal "Help me write a poem"

# Execute with streaming
rg agents execute agent_123 --goal "Write a story" --stream

# Execute with input file
rg agents execute agent_123 --input input.txt

# Execute and save output
rg agents execute agent_123 --goal "Analyze data" --output result.txt
```

## Session Commands

### List Sessions

```bash
# List sessions
rg sessions list

# List for specific agent
rg sessions list --agent agent_123

# List with status filter
rg sessions list --status completed
```

### Get Session

```bash
# Get session details
rg sessions get sess_123

# Get session steps
rg sessions get sess_123 --steps

# Get session output
rg sessions get sess_123 --output
```

### Create Session

```bash
# Create new session
rg sessions create --agent agent_123

# Create with goal
rg sessions create --agent agent_123 --goal "Help me with coding"
```

### Cancel Session

```bash
# Cancel running session
rg sessions cancel sess_123
```

### Replay Session

```bash
# Replay session
rg sessions replay sess_123

# Replay with modifications
rg sessions replay sess_123 --modify-goal "Different goal"
```

## Team Commands

### List Teams

```bash
# List teams
rg teams list
```

### Create Team

```bash
# Create team
rg teams create \
  --name "Support Team" \
  --agents agent_1,agent_2,agent_3 \
  --coordinator agent_1
```

### Execute Team

```bash
# Execute team task
rg teams execute team_123 --goal "Complex multi-step task"
```

## Webhook Commands

### List Webhooks

```bash
# List webhooks
rg webhooks list
```

### Create Webhook

```bash
# Create webhook
rg webhooks create \
  --url https://example.com/webhook \
  --events agent.completed,session.failed \
  --secret my_secret
```

### Test Webhook

```bash
# Test webhook
rg webhooks test webhook_123
```

### Delete Webhook

```bash
# Delete webhook
rg webhooks delete webhook_123
```

## Template Commands

### List Templates

```bash
# List templates
rg templates list

# List marketplace templates
rg templates list --marketplace
```

### Use Template

```bash
# Create agent from template
rg templates use template_123 --name "My Agent"
```

### Create Template

```bash
# Create template from agent
rg templates create --from-agent agent_123 --name "My Template"
```

## Organization Commands

### View Organization

```bash
# View organization details
rg org info
```

### List Members

```bash
# List organization members
rg org members list
```

### Invite Member

```bash
# Invite member
rg org members invite user@example.com --role member
```

### Remove Member

```bash
# Remove member
rg org members remove user_123
```

## Billing Commands

### View Usage

```bash
# View current usage
rg billing usage

# View usage for period
rg billing usage --period 2026-02
```

### View Invoices

```bash
# List invoices
rg billing invoices list

# Download invoice
rg billing invoices download inv_123 --output invoice.pdf
```

## Logs Commands

### View Logs

```bash
# View recent logs
rg logs

# View logs for agent
rg logs --agent agent_123

# View logs for session
rg logs --session sess_123

# Follow logs in real-time
rg logs --follow

# Filter by level
rg logs --level error
```

### Export Logs

```bash
# Export logs
rg logs export --start 2026-02-01 --end 2026-02-21 --output logs.json
```

## Interactive Mode

### Start Interactive Shell

```bash
# Start interactive mode
rg shell

# In shell:
rg> agents list
rg> agents execute agent_123 --goal "Hello"
rg> exit
```

### Chat Mode

```bash
# Start chat with agent
rg chat agent_123

# Chat session:
You: Hello, how are you?
Agent: I'm doing well! How can I help you today?
You: /exit
```

## Output Formats

### Table Format (Default)

```bash
rg agents list --output table

┌────────────┬──────────────┬────────┬─────────────┐
│ ID         │ Name         │ Status │ Model       │
├────────────┼──────────────┼────────┼─────────────┤
│ agent_123  │ My Agent     │ active │ gpt-4-turbo │
│ agent_456  │ Support Bot  │ active │ gpt-4       │
└────────────┴──────────────┴────────┴─────────────┘
```

### JSON Format

```bash
rg agents list --output json

[
  {
    "id": "agent_123",
    "name": "My Agent",
    "status": "active",
    "model": "gpt-4-turbo"
  }
]
```

### YAML Format

```bash
rg agents list --output yaml

- id: agent_123
  name: My Agent
  status: active
  model: gpt-4-turbo
```

### CSV Format

```bash
rg agents list --output csv

id,name,status,model
agent_123,My Agent,active,gpt-4-turbo
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `RG_API_KEY` | API key for authentication |
| `RG_API_URL` | Custom API URL |
| `RG_OUTPUT` | Default output format |
| `RG_TIMEOUT` | Request timeout in seconds |
| `RG_NO_COLOR` | Disable colored output |
| `RG_DEBUG` | Enable debug mode |

```bash
# Example usage
export RG_API_KEY=rg_your_api_key
export RG_OUTPUT=json
rg agents list
```

## Scripting

### Bash Scripts

```bash
#!/bin/bash

# Create agent and capture ID
AGENT_ID=$(rg agents create \
  --name "Script Agent" \
  --system-prompt "You are helpful." \
  --output json | jq -r '.id')

echo "Created agent: $AGENT_ID"

# Execute agent
rg agents execute $AGENT_ID --goal "Hello world"

# Cleanup
rg agents delete $AGENT_ID --yes
```

### Using with jq

```bash
# Get agent names
rg agents list --output json | jq '.[].name'

# Filter active agents
rg agents list --output json | jq '.[] | select(.status == "active")'

# Count agents
rg agents list --output json | jq 'length'
```

### Using with xargs

```bash
# Delete all inactive agents
rg agents list --status inactive --output json | \
  jq -r '.[].id' | \
  xargs -I {} rg agents delete {} --yes
```

## Aliases

### Create Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc
alias rga="rg agents"
alias rgs="rg sessions"
alias rgx="rg agents execute"
alias rgl="rg logs --follow"
```

### Common Aliases

```bash
alias rg-list="rg agents list"
alias rg-run="rg agents execute"
alias rg-chat="rg chat"
alias rg-logs="rg logs --follow"
```

## Completions

### Bash

```bash
# Add to ~/.bashrc
source <(rg completion bash)
```

### Zsh

```bash
# Add to ~/.zshrc
source <(rg completion zsh)
```

### Fish

```bash
# Add to ~/.config/fish/config.fish
rg completion fish | source
```

### PowerShell

```powershell
# Add to profile
rg completion powershell | Out-String | Invoke-Expression
```

## Troubleshooting

### Debug Mode

```bash
# Enable debug output
rg --debug agents list

# Verbose output
rg --verbose agents list
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Authentication failed | Run `rg auth login` |
| Connection timeout | Check network, increase `--timeout` |
| Command not found | Verify installation, check PATH |
| Permission denied | Check API key permissions |

### Reset CLI

```bash
# Clear all config and cache
rg config reset --all

# Re-authenticate
rg auth login
```

## API Reference

### Global Flags

| Flag | Description |
|------|-------------|
| `--api-key` | Override API key |
| `--output, -o` | Output format (table, json, yaml, csv) |
| `--timeout` | Request timeout |
| `--debug` | Enable debug mode |
| `--verbose, -v` | Verbose output |
| `--no-color` | Disable colors |
| `--help, -h` | Show help |
| `--version` | Show version |

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Authentication error |
| 4 | Network error |
| 5 | Resource not found |

---

**Need CLI help?** Run `rg help` or contact support@resonantgenesis.xyz.
