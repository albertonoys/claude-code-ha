ARG BUILD_FROM
FROM ${BUILD_FROM}

# Install packages in a single layer to minimize image size
# Node.js and npm kept for image-service dependency
RUN apk add --no-cache \
    nodejs \
    npm \
    bash \
    curl \
    nano \
    python3 \
    py3-pip \
    py3-requests \
    py3-aiohttp \
    py3-yaml \
    py3-beautifulsoup4 \
    jq \
    yq \
    git \
    vim \
    wget \
    tree

# Install Claude Code CLI
# Try native installer first (recommended by Anthropic, uses Bun runtime).
# Falls back to npm if native install fails (e.g., CPU lacks AVX support).
RUN (curl -fsSL https://claude.ai/install.sh | bash \
    && ln -sf /root/.local/bin/claude /usr/local/bin/claude) \
    || npm install -g @anthropic-ai/claude-code

# Install Home Assistant CLI (ha command)
# Pinned version with checksum verification
ARG HA_CLI_VERSION=5.1.0
ARG TARGETARCH
RUN case "${TARGETARCH}" in \
        amd64)   HA_ARCH="amd64"; HA_CLI_SHA256="46cf957ed8332dc167e13dde0f8fffe9f3553e4ce10a2987c10e3f74e20f57e3" ;; \
        arm64)   HA_ARCH="aarch64"; HA_CLI_SHA256="fd7359ea9ae6e108c5b7769f1db699edf09ab9dd19aded2cbb12172af4c96cd5" ;; \
        *)       echo "Unsupported architecture: ${TARGETARCH}" && exit 1 ;; \
    esac && \
    echo "Installing Home Assistant CLI ${HA_CLI_VERSION} for ${HA_ARCH}..." && \
    curl -fsSL -o /usr/bin/ha \
        "https://github.com/home-assistant/cli/releases/download/${HA_CLI_VERSION}/ha_${HA_ARCH}" && \
    echo "${HA_CLI_SHA256}  /usr/bin/ha" | sha256sum -c - && \
    chmod +x /usr/bin/ha && \
    echo "Home Assistant CLI ${HA_CLI_VERSION} installed successfully"

# Install GitHub CLI (gh command)
# Pinned version with checksum verification
ARG GH_VERSION=2.93.0
RUN case "${TARGETARCH}" in \
        amd64)   GH_ARCH="amd64"; GH_SHA256="02d1290eba130e0b896f3709ffff22e1c75a51475ddb70476a85abc6b5807af0" ;; \
        arm64)   GH_ARCH="arm64"; GH_SHA256="c55feb33684abba57e9909737340d5b39282257c0363e1edde6785ac4a413be7" ;; \
        *)       echo "Unsupported architecture for gh: ${TARGETARCH}" && exit 1 ;; \
    esac && \
    echo "Installing GitHub CLI v${GH_VERSION} for ${GH_ARCH}..." && \
    curl -fsSL -o /tmp/gh.tar.gz \
        "https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_${GH_ARCH}.tar.gz" && \
    echo "${GH_SHA256}  /tmp/gh.tar.gz" | sha256sum -c - && \
    tar -xzf /tmp/gh.tar.gz -C /tmp && \
    mv /tmp/gh_${GH_VERSION}_linux_${GH_ARCH}/bin/gh /usr/bin/gh && \
    chmod +x /usr/bin/gh && \
    rm -rf /tmp/gh* && \
    echo "GitHub CLI v${GH_VERSION} installed successfully"

# Create directory for Claude configuration and set working directory
RUN mkdir -p /config/claude-config
WORKDIR /config

# Copy and install image service
COPY image-service/ /opt/image-service/
RUN cd /opt/image-service \
    && npm install --production --no-optional \
    && npm cache clean --force \
    && chmod +x server.js

# Copy startup script and modular scripts
COPY run.sh /run.sh
COPY scripts/ /opt/scripts/
COPY .claude/ /opt/.claude/
RUN chmod +x /run.sh \
    && chmod +x /opt/scripts/*.sh

# Use exec form for better signal handling
CMD ["/run.sh"]
