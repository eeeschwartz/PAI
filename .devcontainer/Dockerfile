FROM mcr.microsoft.com/devcontainers/typescript-node:1-20-bullseye

# Install additional dependencies
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    jq \
    ripgrep \
    fd-find \
    build-essential \
    python3-pip \
    chromium \
    chromium-driver \
    && rm -rf /var/lib/apt/lists/*

# Install bun
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/home/vscode/.bun/bin:${PATH}"

# Install uv for Python package management
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/home/vscode/.cargo/bin:${PATH}"

# Install Claude CLI
RUN npm install -g @anthropic-ai/claude-code
ENV PATH="/home/vscode/.local/bin:${PATH}"

# The base image has a node user with UID 1000
# Create vscode user for devcontainer compatibility
RUN groupadd -g 1001 vscode && \
    useradd -u 1001 -g vscode -m -s /bin/bash vscode

# Create necessary directories and set permissions
RUN mkdir -p /home/vscode/.mcp \
    && mkdir -p /home/vscode/.config \
    && mkdir -p /home/vscode/.cache \
    && chown -R vscode:vscode /home/vscode

# Switch to vscode user
USER vscode

# Set working directory
WORKDIR /workspace

# MCP servers will be installed on-demand via bunx
# based on .claude/.mcp.json configuration

# Set environment variables for browser automation
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium
ENV PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1
ENV PLAYWRIGHT_BROWSERS_PATH=/usr/bin

# Setup script will be run post-create
# (MCP directory already created above)

ENTRYPOINT ["/bin/bash"]
