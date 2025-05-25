FROM python:3.10-slim

# Install system dependencies needed for building Python packages and runtime
RUN apt-get update && apt-get install -y \
    build-essential \
    python3-dev \
    libffi-dev \
    libssl-dev \
    espeak-ng \
    espeak-ng-data \
    git \
    libsndfile1 \
    curl \
    ffmpeg \
    g++ \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* \
 && mkdir -p /usr/share/espeak-ng-data \
 && ln -s /usr/lib/*/espeak-ng-data/* /usr/share/espeak-ng-data/

# Install UV CLI tool
RUN curl -LsSf https://astral.sh/uv/install.sh | sh && \
    mv /root/.local/bin/uv /usr/local/bin/ && \
    mv /root/.local/bin/uvx /usr/local/bin/

# Create non-root user
RUN useradd -m -u 1000 appuser

USER appuser
WORKDIR /app

# Copy all required files early before install
COPY --chown=appuser:appuser pyproject.toml ./pyproject.toml
COPY --chown=appuser:appuser README.md ./README.md
COPY --chown=appuser:appuser api ./api
COPY --chown=appuser:appuser web ./web
COPY --chown=appuser:appuser docker/scripts/ ./

# Install Rust for building native extensions
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
ENV PATH="/home/appuser/.cargo/bin:$PATH"

# Create virtual environment and install dependencies using uv
RUN --mount=type=cache,target=/home/appuser/.cache/uv uv venv --python 3.10
RUN --mount=type=cache,target=/home/appuser/.cache/uv uv sync --extra cpu --verbose

# Install your package into the virtual environment
RUN /app/.venv/bin/pip install --upgrade pip setuptools wheel
RUN /app/.venv/bin/pip install .

# Make entrypoint executable
RUN chmod +x ./entrypoint.sh

ENV PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app:/app/api \
    PATH="/app/.venv/bin:$PATH" \
    UV_LINK_MODE=copy \
    USE_GPU=false \
    PHONEMIZER_ESPEAK_PATH=/usr/bin \
    PHONEMIZER_ESPEAK_DATA=/usr/share/espeak-ng-data \
    ESPEAK_DATA_PATH=/usr/share/espeak-ng-data \
    DOWNLOAD_MODEL=true \
    DEVICE="cpu"

# Download model if enabled
RUN if [ "$DOWNLOAD_MODEL" = "true" ]; then \
    python download_model.py --output api/src/models/v1_0; \
    fi

CMD ["./entrypoint.sh"]
