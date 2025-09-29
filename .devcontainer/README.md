# MCP Server DevContainer - Secure Isolated Environment

## 🔒 Security Overview

This devcontainer provides an isolated environment for running MCP (Model Context Protocol) servers with the following security features:

### Isolation Layers
- **Filesystem Isolation**: Only `/workspace` directory is accessible
- **Network Isolation**: Custom Docker network with restricted access
- **Capability Dropping**: Minimal Linux capabilities (drops ALL, adds only SYS_PTRACE)
- **Read-only Mounts**: Sensitive files (.env, .ssh) mounted as read-only
- **User Isolation**: Runs as non-root `vscode` user

### Security Benefits vs Host Machine
| Feature | Host Machine | DevContainer |
|---------|--------------|--------------|
| File Access | Full system | Only /workspace |
| Network Access | Unrestricted | Isolated network |
| Process Isolation | None | Container boundaries |
| Credential Exposure | Direct access | Read-only mounts |
| System Commands | Full privileges | Dropped capabilities |
| Cleanup | Manual | Delete container |

## 🚀 Quick Start

### Prerequisites
- Docker Desktop installed and running
- VS Code with Dev Containers extension
- Your environment variables in `~/.env`

### Setup Steps

1. **Prepare your environment file**:
   ```bash
   # Copy the example and edit with your values
   cp .devcontainer/.env.example ~/.env
   # Edit ~/.env with your actual API keys and settings
   ```

2. **Open in VS Code**:
   ```bash
   # From the PAI directory
   code .
   ```

3. **Start the DevContainer**:
   - Press `F1` or `Cmd+Shift+P`
   - Select "Dev Containers: Reopen in Container"
   - Wait for the container to build (first time takes ~2-3 minutes)

4. **Verify the setup**:
   ```bash
   # Inside the container terminal
   cat ~/.mcp.json  # Check configuration
   bunx --version    # Verify bun is available
   ```

## 📁 Directory Structure

```
.devcontainer/
├── devcontainer.json    # VS Code configuration
├── docker-compose.yml   # Container orchestration
├── Dockerfile          # Container image definition
├── setup-mcp-servers.sh # Environment verification script
├── .env.example        # Environment template
├── .gitignore         # Prevent secret commits
└── README.md          # This file

.claude/
└── .mcp.json          # MCP server configuration (mounted read-only)
```

## 🛠️ MCP Server Configuration

### Native Claude Code Integration
The devcontainer uses your existing `.claude/.mcp.json` configuration file. Claude Code manages MCP servers automatically based on this configuration.

### Current Configuration
Your `.claude/.mcp.json` is mounted read-only and copied to `~/.mcp.json` in the container. MCP servers defined there will be available to Claude Code.

### Adding New MCP Servers
Edit `.claude/.mcp.json` in your host environment:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "bunx",
      "args": ["@playwright/mcp@latest", "--extension"],
      "description": "Browser automation"
    },
    "filesystem": {
      "command": "bunx",
      "args": ["@modelcontextprotocol/server-filesystem"],
      "env": {
        "ALLOWED_PATHS": "/workspace"
      }
    }
  }
}
```

**Note:** Servers are installed on-demand via `bunx` when Claude Code needs them.

## 🔐 Security Best Practices

### DO's ✅
- Keep `.env` file in your home directory (outside project)
- Use read-only mounts for sensitive data
- Regularly update the container image
- Review logs for suspicious activity
- Use separate containers for different security contexts
- Limit network access to only required services

### DON'Ts ❌
- Don't commit `.env` files to git
- Don't mount your entire home directory
- Don't run with `--privileged` flag
- Don't expose container ports publicly
- Don't store credentials in the container
- Don't disable security features

## 🎭 Use Cases

### High-Risk Operations (Use DevContainer)
- Web scraping unknown sites
- Running untrusted code
- Security testing/pentesting
- Processing user-uploaded files
- Interacting with external APIs
- Browser automation tasks

### Low-Risk Operations (Can Use Host)
- Local file editing
- Git operations on trusted repos
- Reading documentation
- Running trusted tests
- Building known projects

## 🔧 Configuration

### Environment Variables for MCP Servers

Add server-specific environment variables in your `.claude/.mcp.json`:

```json
{
  "mcpServers": {
    "your-server": {
      "command": "bunx",
      "args": ["@your-org/mcp-server"],
      "env": {
        "API_KEY": "${YOUR_API_KEY}",
        "WORKSPACE": "/workspace"
      }
    }
  }
}
```

Environment variables from `~/.env` will be available.

### Adjusting Security Policies

Modify `docker-compose.yml` for different security levels:

```yaml
# More restrictive
cap_drop:
  - ALL
security_opt:
  - no-new-privileges
  - apparmor:docker-default

# Less restrictive (not recommended)
cap_add:
  - SYS_ADMIN
  - NET_ADMIN
```

### Adding Volume Mounts

In `devcontainer.json`, add specific directories:

```json
"mounts": [
  "source=/path/to/data,target=/data,type=bind,readonly"
]
```

## 🐛 Troubleshooting

### Container won't start
```bash
# Check Docker is running
docker ps

# Check for port conflicts
lsof -i :3000 -i :8080

# Rebuild container
# In VS Code: F1 -> "Dev Containers: Rebuild Container"
```

### MCP servers not responding
```bash
# Check if MCP configuration is loaded
cat ~/.mcp.json

# Verify bun/bunx is working
bunx --version

# Test a specific MCP server
bunx @playwright/mcp@latest --help

# Claude Code manages servers automatically
# If issues persist, rebuild the container
```

### Permission issues
```bash
# Fix ownership
sudo chown -R $(id -u):$(id -g) /workspace

# Check mount permissions
mount | grep workspace
```

## 📊 Performance Considerations

- **File I/O**: Cached volume mount improves performance
- **Memory**: Each MCP server uses ~50-200MB RAM
- **CPU**: Browser automation is CPU-intensive
- **Network**: Isolated network may add minor latency

## 🔄 Updating

To update MCP servers or dependencies:

```bash
# Rebuild the container with no cache
# In VS Code: F1 -> "Dev Containers: Rebuild Container Without Cache"

# Or manually:
docker-compose -f .devcontainer/docker-compose.yml build --no-cache
```

## 📝 Environment Variables

Key variables to configure in `~/.env`:

```bash
# Required for most operations
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...

# Optional based on usage
GITHUB_TOKEN=ghp_...
DATABASE_URL=postgresql://...
MCP_LOG_LEVEL=debug  # info, warn, error
MCP_TIMEOUT_MS=30000  # 30 seconds
```

## 🚨 Emergency Procedures

### Kill all MCP processes
```bash
pkill -f "node.*mcp"
```

### Exit container immediately
```bash
exit
# Or in VS Code: F1 -> "Dev Containers: Reopen Folder Locally"
```

### Clean up everything
```bash
# Outside container
docker-compose -f .devcontainer/docker-compose.yml down -v
docker system prune -a
```

## 📚 Additional Resources

- [MCP Documentation](https://modelcontextprotocol.io)
- [VS Code Dev Containers](https://code.visualstudio.com/docs/remote/containers)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)

## 💡 Tips

1. **Use aliases** for common commands:
   ```bash
   alias mcp-start='~/.mcp/start-servers.sh'
   alias mcp-stop='~/.mcp/stop-servers.sh'
   alias mcp-logs='tail -f ~/.mcp/logs/*.log'
   ```

2. **Monitor resource usage**:
   ```bash
   docker stats mcp-workspace
   ```

3. **Test isolation**:
   ```bash
   # This should fail (no access outside /workspace)
   ls /etc/passwd
   cat ~/.ssh/id_rsa  # Read-only
   ```

4. **Use separate containers** for different trust levels:
   - Development work: `devcontainer-dev`
   - Testing untrusted code: `devcontainer-sandbox`
   - Production deployments: `devcontainer-prod`

---

**Remember**: The DevContainer provides defense-in-depth. Even if an MCP server is compromised, the attacker is confined to the container with limited capabilities and access.