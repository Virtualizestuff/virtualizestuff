# syntax=docker/dockerfile:1
FROM mcr.microsoft.com/devcontainers/base:ubuntu-22.04

ARG USER=vscode
ARG DEBIAN_FRONTEND=noninteractive

# Install required tools
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    ca-certificates \
    golang-go \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js 20 LTS (required for blowfish-tools)
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/*

# Use bash for maximum CI parity (recommended)
RUN usermod -s /bin/bash ${USER}

# Hugo configuration
ARG VARIANT=hugo_extended
ARG VERSION=0.154.5

# Install Hugo
RUN echo "Installing Hugo ${VERSION}" && \
    ARCH=64bit && \
    [ "$(uname -m)" = "aarch64" ] && ARCH=ARM64 || true && \
    wget -O hugo.tar.gz \
      https://github.com/gohugoio/hugo/releases/download/v${VERSION}/${VARIANT}_${VERSION}_Linux-${ARCH}.tar.gz && \
    tar xf hugo.tar.gz && \
    mv hugo /usr/local/bin/hugo && \
    rm -f hugo.tar.gz

# Hugo dev server port
EXPOSE 1313

USER ${USER}
WORKDIR /home/${USER}

# Create X11 tmp dir for VS Code attach (rootless Podman)
RUN mkdir -p /tmp/.X11-unix