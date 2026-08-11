# syntax=docker/dockerfile:1

# --- Builder: clone pyserve, compile deps (scipy/prophet/pyodbc), pre-warm cmdstan ---
FROM python:3.12-slim-bookworm AS builder

ARG PYSERVE_REPO_URL=https://bitbucket.org/baosystems/pyserve.git
ARG PYSERVE_REF=main

RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        cmake \
        git \
        ca-certificates \
        unixodbc-dev \
        curl \
    && rm -rf /var/lib/apt/lists/*

RUN --mount=type=secret,id=bitbucket_token \
    if [ -s /run/secrets/bitbucket_token ]; then \
        TOKEN="$(cat /run/secrets/bitbucket_token)"; \
        CLONE_URL="$(printf '%s' "$PYSERVE_REPO_URL" | sed "s#https://#https://x-token-auth:${TOKEN}@#")"; \
    else \
        CLONE_URL="$PYSERVE_REPO_URL"; \
    fi; \
    git clone --depth 1 --branch "$PYSERVE_REF" "$CLONE_URL" /src

RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r /src/requirements.txt

# Pre-install cmdstan so Prophet never has to build/download it at container runtime.
ENV CMDSTAN=/opt/cmdstan
RUN python -c "from cmdstanpy import install_cmdstan; install_cmdstan(dir='/opt/cmdstan', overwrite=True)"

# --- Runtime: slim image with only what's needed to run the service ---
FROM python:3.12-slim-bookworm AS runtime

RUN apt-get update && apt-get install -y --no-install-recommends \
        libgomp1 \
        unixodbc \
        libpq5 \
        curl \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && groupadd -g 1001 bao-admin \
    && useradd -u 1001 -g bao-admin -m -s /usr/sbin/nologin bao-admin

COPY --from=builder /opt/venv /opt/venv
COPY --from=builder /src/main.py /src/schema.py /app/
COPY --from=builder --chown=bao-admin:bao-admin /opt/cmdstan /home/bao-admin/.cmdstan

ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    CMDSTAN=/home/bao-admin/.cmdstan

WORKDIR /app
USER bao-admin

EXPOSE 8000

HEALTHCHECK --interval=10s --timeout=5s --retries=5 \
    CMD curl -f http://localhost:8000/docs || exit 1

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
