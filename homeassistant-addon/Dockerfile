# syntax=docker/dockerfile:1
# Home Assistant MCP Server Add-on — Saxin fork
# Installs ha-mcp directly from upstream git (no src/ sync needed)
# Patches start.py to enable filesystem tools (HAMCP_ENABLE_FILESYSTEM_TOOLS)

# --- Build stage ---
FROM ghcr.io/astral-sh/uv:0.11.0-python3.13-trixie-slim@sha256:5b216b72b3bc10f983f82b39b5386455bfa08d2139afb4cb3f6c9f060484ea5d AS builder

WORKDIR /app
ENV UV_COMPILE_BYTECODE=1 UV_LINK_MODE=copy

# Use CACHE_BUSTER to force re-clone when upstream changes
ARG CACHE_BUSTER=1
RUN apt-get update && apt-get install -y --no-install-recommends git && rm -rf /var/lib/apt/lists/*
RUN --mount=type=cache,target=/root/.cache/uv \
    uv venv && \
    uv pip install "ha-mcp @ git+https://github.com/homeassistant-ai/ha-mcp.git"

# --- Runtime stage ---
FROM python:3.13-slim@sha256:3de9a8d7aedbb7984dc18f2dff178a7850f16c1ae7c34ba9d7ecc23d0755e35f

WORKDIR /app

COPY --from=builder /app/.venv /app/.venv
COPY start.py /

# Enable filesystem tools (not yet wired into upstream addon config)
RUN sed -i 's/os.environ\["ENABLE_YAML_CONFIG_EDITING"\] = str(enable_yaml_config_editing).lower()/os.environ["ENABLE_YAML_CONFIG_EDITING"] = str(enable_yaml_config_editing).lower()\n    os.environ["HAMCP_ENABLE_FILESYSTEM_TOOLS"] = "true"/' /start.py

ENV PATH="/app/.venv/bin:$PATH"

ARG BUILD_VERSION
ARG BUILD_ARCH
LABEL \
    io.hass.name="Home Assistant MCP Server" \
    io.hass.description="AI assistant integration via Model Context Protocol" \
    io.hass.version="${BUILD_VERSION}" \
    io.hass.type="addon" \
    io.hass.arch="${BUILD_ARCH}"

CMD ["python3", "/start.py"]
