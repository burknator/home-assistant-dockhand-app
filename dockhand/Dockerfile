# Stage 1 — extract Dockhand artifacts
FROM fnsys/dockhand:v1.0.55 AS dockhand

# Stage 2 — HA Debian base (glibc, s6-overlay, bashio)
FROM ghcr.io/home-assistant/base-debian:trixie-2026.10.3

# Add Docker's official GPG key and apt repository
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl \
    && install -m 0755 -d /etc/apt/keyrings \
    && curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc \
    && chmod a+r /etc/apt/keyrings/docker.asc \
    && tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Install nginx, docker-cli, and Node.js 24 (NodeSource)
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        git \
        nginx \
        docker-ce \
        docker-ce-cli \
        containerd.io \
        docker-buildx-plugin \
        docker-compose-plugin \
    && curl -fsSL https://deb.nodesource.com/setup_24.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

# Create dockhand user (matching Dockhand's uid/gid 1001)
RUN groupadd -g 1001 dockhand \
    && useradd -u 1001 -g dockhand -m -d /home/dockhand dockhand

# Copy Dockhand application from Stage 1
COPY --from=dockhand --chown=dockhand:dockhand /app /app
COPY --from=dockhand --chown=dockhand:dockhand /home/dockhand /home/dockhand

# Copy rootfs (nginx config, s6 services, cont-init scripts)
COPY rootfs /
RUN chmod +x /etc/cont-init.d/*.sh \
    && chmod +x /etc/services.d/*/run /etc/services.d/*/finish

ENV DATA_DIR=/data \
    PORT=3000 \
    HOST=127.0.0.1 \
    NODE_ENV=production
