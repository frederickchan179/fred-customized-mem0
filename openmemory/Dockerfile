# ##############################################################################
# Here is my Dockerfile to self-host OpenMemory MCP on Coolify.
# I will run this instead of the original Dockerfile.
# ##############################################################################

# ─────────────────────────────────────────────────────────────────────────────
# 1. Base: Node 20 slim
FROM node:20-slim

# Define build arguments
ARG OPENAI_API_KEY
ARG USER
ARG NEXT_PUBLIC_API_URL

# 2. Make Python logs unbuffered and set Uvicorn log level
ENV PYTHONUNBUFFERED=1
ENV UVICORN_LOG_LEVEL=debug

# 3. Install runtimes & tools, including venv support
RUN apt-get update && apt-get install -y --no-install-recommends \
      python3 \
      python3-venv \
      python3-pip \
      git \
      make \
      g++ \
      gettext \
      curl \
  && rm -rf /var/lib/apt/lists/*

# 4. Install pnpm globally
RUN npm install -g pnpm

# ─────────────────────────────────────────────────────────────────────────────
# 5. Set working directory and clone repo
WORKDIR /app
RUN git clone --depth 1 https://github.com/frederickchan179/fred-customized-mem0.git .

# ─────────────────────────────────────────────────────────────────────────────
# 6. Prepare & install API in a venv
WORKDIR /app/openmemory/api

# Create and configure environment file from environment variables
RUN echo "OPENAI_API_KEY=${OPENAI_API_KEY}" > .env

# Create + activate venv for future RUNs
RUN python3 -m venv .venv
ENV PATH="/app/openmemory/api/.venv/bin:${PATH}"

# Now pip will install into the venv
RUN pip install --no-cache-dir --verbose -r requirements.txt

# ─────────────────────────────────────────────────────────────────────────────
# 7. Prepare UI environment
WORKDIR /app/openmemory/ui

# Create and configure environment file from environment variables
RUN echo "NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}" > .env && \
    echo "NEXT_PUBLIC_USER_ID=${USER}" >> .env

# Install & build the UI
RUN pnpm install
RUN pnpm build

# ─────────────────────────────────────────────────────────────────────────────
# 8. Set working directory for container start
WORKDIR /app/openmemory

# Expose both ports
EXPOSE 8765 3000

# ─────────────────────────────────────────────────────────────────────────────
# 9. Start services
CMD ["bash", "-c", "\
  (cd /app/openmemory/api && python -m uvicorn main:app --host 0.0.0.0 --port 8765 --log-level debug) & \
  (cd /app/openmemory/ui && pnpm start --port 3000) \
"]