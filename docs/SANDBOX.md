# Sandbox & Security Isolation Guide

Complete guide to Docker sandboxing, security isolation, and resource limits in ResonantGenesis.

## Overview

ResonantGenesis uses Docker-based sandboxing to provide secure, isolated execution environments for AI agents. This ensures:

- **Security**: Agents cannot access host systems or other agents
- **Resource Control**: CPU, memory, and network limits per execution
- **Reproducibility**: Consistent execution environments
- **Auditability**: Full logging of all sandbox activities

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Agent Engine                          │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  Sandbox 1  │  │  Sandbox 2  │  │  Sandbox 3  │     │
│  │  (Agent A)  │  │  (Agent B)  │  │  (Agent C)  │     │
│  │             │  │             │  │             │     │
│  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │     │
│  │ │ Python  │ │  │ │  Node   │ │  │ │ Python  │ │     │
│  │ │ Runtime │ │  │ │ Runtime │ │  │ │ Runtime │ │     │
│  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │     │
│  │             │  │             │  │             │     │
│  │ CPU: 1 core │  │ CPU: 2 core │  │ CPU: 1 core │     │
│  │ RAM: 512MB  │  │ RAM: 1GB    │  │ RAM: 512MB  │     │
│  │ Net: None   │  │ Net: Limited│  │ Net: None   │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
```

## Sandbox Types

### Code Execution Sandbox

For running user-provided code:

| Feature | Description |
|---------|-------------|
| **Isolation** | Full container isolation |
| **Languages** | Python, JavaScript, Bash |
| **Network** | Disabled by default |
| **Filesystem** | Read-only with temp workspace |
| **Timeout** | 30 seconds default |

### Tool Execution Sandbox

For running agent tools:

| Feature | Description |
|---------|-------------|
| **Isolation** | Process isolation |
| **Network** | Limited to approved endpoints |
| **Filesystem** | Scoped to agent workspace |
| **Timeout** | Tool-specific limits |

### Browser Automation Sandbox

For web automation tasks:

| Feature | Description |
|---------|-------------|
| **Isolation** | Headless browser container |
| **Network** | Filtered, logged |
| **Screenshots** | Captured for audit |
| **Timeout** | 60 seconds default |

## Resource Limits

### By Trust Tier

| Tier | CPU | Memory | Timeout | Network |
|------|-----|--------|---------|---------|
| T0 | 0.5 cores | 256MB | 10s | None |
| T1 | 1 core | 512MB | 30s | None |
| T2 | 2 cores | 1GB | 60s | Limited |
| T3 | 4 cores | 2GB | 120s | Filtered |
| T4 | 8 cores | 4GB | 300s | Full |

### Configuration

```python
# Configure sandbox limits
agent = client.agents.create(
    name="Data Processor",
    tools=["code_exec"],
    tool_config={
        "code_exec": {
            "cpu_limit": 2.0,        # CPU cores
            "memory_limit": "1GB",   # Memory limit
            "timeout_seconds": 60,   # Execution timeout
            "network_enabled": False # Network access
        }
    }
)
```

## Security Features

### Container Isolation

Each sandbox runs in an isolated Docker container with:

- **Separate namespace**: Process, network, mount, user
- **Read-only root filesystem**: Prevents system modifications
- **No privileged access**: No root capabilities
- **Seccomp profile**: Restricted system calls
- **AppArmor/SELinux**: Mandatory access control

### Network Security

```yaml
# Network policy
network:
  mode: "none"  # none, limited, filtered, full
  
  # For limited/filtered modes
  allowed_hosts:
    - "api.openai.com"
    - "api.anthropic.com"
  
  blocked_hosts:
    - "*.internal"
    - "169.254.169.254"  # AWS metadata
  
  rate_limit:
    requests_per_minute: 60
    bandwidth_mbps: 10
```

### Filesystem Security

```yaml
# Filesystem policy
filesystem:
  root: "read-only"
  
  writable_paths:
    - "/tmp"
    - "/workspace"
  
  max_file_size: "10MB"
  max_total_size: "100MB"
  
  blocked_paths:
    - "/etc/passwd"
    - "/etc/shadow"
    - "/proc"
    - "/sys"
```

### Process Security

```yaml
# Process policy
process:
  max_processes: 10
  max_threads: 50
  
  blocked_syscalls:
    - "mount"
    - "umount"
    - "ptrace"
    - "reboot"
  
  capabilities:
    drop:
      - "ALL"
    add:
      - "NET_BIND_SERVICE"  # Only if network enabled
```

## Sandbox Lifecycle

### Creation

```python
# Sandbox is created automatically when needed
result = agent.execute(
    goal="Run this Python code",
    context={"code": "print('Hello')"}
)

# Sandbox lifecycle:
# 1. Container created from base image
# 2. Workspace mounted
# 3. Code executed
# 4. Results captured
# 5. Container destroyed
```

### Execution Flow

```
1. Request received
   ↓
2. Validate permissions (trust tier)
   ↓
3. Create isolated container
   ↓
4. Apply resource limits
   ↓
5. Mount workspace (read-only + temp)
   ↓
6. Execute code/tool
   ↓
7. Capture output/errors
   ↓
8. Destroy container
   ↓
9. Return results
```

### Cleanup

Sandboxes are automatically cleaned up:

- **On completion**: Immediate destruction
- **On timeout**: Force killed and destroyed
- **On error**: Destroyed after logging
- **Orphaned**: Cleaned by garbage collector

## Code Execution

### Python Execution

```python
# Execute Python code in sandbox
result = agent.execute(
    goal="Calculate statistics",
    context={
        "code": """
import pandas as pd
import numpy as np

data = [1, 2, 3, 4, 5]
print(f"Mean: {np.mean(data)}")
print(f"Std: {np.std(data)}")
""",
        "language": "python"
    }
)
```

### Available Packages

**Python (pre-installed):**
- numpy, pandas, scipy
- requests (if network enabled)
- matplotlib, seaborn
- scikit-learn
- beautifulsoup4, lxml

**JavaScript (pre-installed):**
- lodash, axios (if network enabled)
- moment, date-fns
- cheerio

### Custom Packages

```python
# Request additional packages (T2+ only)
result = agent.execute(
    goal="Process data with custom package",
    context={
        "code": "import custom_package; ...",
        "packages": ["custom_package==1.0.0"]
    }
)
```

## Monitoring & Logging

### Execution Logs

```python
# Get sandbox execution logs
logs = client.sandbox.get_logs(
    execution_id="exec_123"
)

print(f"Stdout: {logs.stdout}")
print(f"Stderr: {logs.stderr}")
print(f"Exit code: {logs.exit_code}")
print(f"Duration: {logs.duration_ms}ms")
print(f"Memory peak: {logs.memory_peak_mb}MB")
```

### Audit Trail

All sandbox executions are logged:

```json
{
  "execution_id": "exec_123",
  "agent_id": "agent_456",
  "user_id": "user_789",
  "timestamp": "2026-02-21T03:19:00Z",
  "sandbox_type": "code_exec",
  "language": "python",
  "duration_ms": 1234,
  "exit_code": 0,
  "resources": {
    "cpu_seconds": 0.5,
    "memory_peak_mb": 128,
    "network_bytes": 0
  },
  "security_events": []
}
```

### Security Events

```python
# Get security events
events = client.sandbox.get_security_events(
    execution_id="exec_123"
)

for event in events:
    print(f"{event.type}: {event.message}")
    # Examples:
    # - "blocked_syscall: ptrace"
    # - "network_blocked: 10.0.0.1"
    # - "file_access_denied: /etc/passwd"
```

## Error Handling

### Timeout Errors

```python
try:
    result = agent.execute(goal="Long task")
except SandboxTimeoutError as e:
    print(f"Execution timed out after {e.timeout_seconds}s")
    print(f"Partial output: {e.partial_output}")
```

### Resource Limit Errors

```python
try:
    result = agent.execute(goal="Memory intensive task")
except SandboxResourceError as e:
    print(f"Resource limit exceeded: {e.resource}")
    print(f"Limit: {e.limit}, Used: {e.used}")
```

### Security Violations

```python
try:
    result = agent.execute(goal="Task with blocked operation")
except SandboxSecurityError as e:
    print(f"Security violation: {e.violation_type}")
    print(f"Details: {e.details}")
```

## Best Practices

### For Developers

1. **Minimize code complexity** - Simpler code = faster execution
2. **Handle timeouts gracefully** - Design for interruption
3. **Use appropriate limits** - Don't over-provision
4. **Test locally first** - Debug before sandbox execution

### For Administrators

1. **Monitor resource usage** - Track patterns and anomalies
2. **Review security events** - Investigate violations
3. **Update base images** - Keep dependencies current
4. **Set appropriate limits** - Balance security and usability

### Security Guidelines

1. **Never trust user input** - Always sanitize
2. **Minimize network access** - Disable when not needed
3. **Use read-only filesystems** - Prevent persistence
4. **Log everything** - Enable full audit trails
5. **Regular security audits** - Review configurations

## Configuration Reference

### Environment Variables

```bash
# Sandbox configuration
SANDBOX_ENABLED=true
SANDBOX_DOCKER_SOCKET=/var/run/docker.sock
SANDBOX_BASE_IMAGE=resonantgenesis/sandbox:latest
SANDBOX_NETWORK_MODE=none
SANDBOX_DEFAULT_TIMEOUT=30
SANDBOX_MAX_MEMORY=1GB
SANDBOX_MAX_CPU=2.0
```

### Docker Compose

```yaml
services:
  sandbox-manager:
    image: resonantgenesis/sandbox-manager:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - SANDBOX_POOL_SIZE=10
      - SANDBOX_CLEANUP_INTERVAL=60
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
```

## API Reference

### Execute in Sandbox

```bash
POST /api/v1/sandbox/execute
```

### Get Execution Status

```bash
GET /api/v1/sandbox/executions/{execution_id}
```

### Get Execution Logs

```bash
GET /api/v1/sandbox/executions/{execution_id}/logs
```

### Get Security Events

```bash
GET /api/v1/sandbox/executions/{execution_id}/security-events
```

### List Active Sandboxes

```bash
GET /api/v1/sandbox/active
```

---

**Need help?** Contact security@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
