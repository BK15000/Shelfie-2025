# Build stage
FROM python:3.11-slim AS builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Runtime stage
FROM python:3.11-slim

# Set environment variables
ENV PYTHONPATH=/src \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /src

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    libpq5 \
    && rm -rf /var/lib/apt/lists/* && \
    # Create non-root user
    groupadd -r appgroup && \
    useradd -r -g appgroup -s /sbin/nologin -M appuser && \
    # Create necessary directories with proper permissions
    mkdir -p /src/auth_backend /src/data/images && \
    chown -R appuser:appgroup /src

# Copy Python packages from builder
COPY --from=builder /usr/local/lib/python3.11/site-packages/ /usr/local/lib/python3.11/site-packages/
COPY --from=builder /usr/local/bin/ /usr/local/bin/

# Copy application code
COPY --chown=appuser:appgroup . /src/auth_backend/

# Switch to non-root user
USER appuser

# Expose port
EXPOSE ${PORT:-8080}

# Command to run the application
CMD ["uvicorn", "auth_backend.main:app", "--host", "0.0.0.0", "--port", "8080", "--proxy-headers", "--forwarded-allow-ips=*"]
