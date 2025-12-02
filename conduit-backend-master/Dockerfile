# Backend Dockerfile (Django + Gunicorn) - multi-stage

# --- builder: install dependencies and wheels
FROM python:3.6-slim AS builder
WORKDIR /build

# system deps required for common Python packages (adjust if you need more)
RUN apt-get update && \
    apt-get install -y gcc libpq-dev build-essential --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*

# copy requirement files
COPY conduit-backend-master/requirements.txt /build/requirements.txt

# create a wheelhouse to speed future builds & isolate installs
RUN python -m pip install --upgrade pip setuptools wheel
RUN pip wheel --wheel-dir=/build/wheels -r /build/requirements.txt

# --- final: slim runtime image
FROM python:3.6-slim AS final
WORKDIR /app

# runtime packages (kept minimal)
# runtime packages (kept minimal)
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 curl && rm -rf /var/lib/apt/lists/*

# copy wheelhouse and install
COPY --from=builder /build/wheels /wheels
COPY --from=builder /build/requirements.txt /app/requirements.txt
RUN pip install --no-index --find-links=/wheels -r /app/requirements.txt

# copy application code
# Assumes your Django project root (manage.py) is inside conduit-backend-master/
COPY conduit-backend-master/ /app/

# create directories for DB, media and logs
RUN mkdir -p /app/data /app/media /var/log/conduit
ENV CONDUIT_DATA_DIR=/app/data
ENV CONDUIT_MEDIA_DIR=/app/media
ENV PYTHONUNBUFFERED=1

# Expose port for gunicorn
EXPOSE 8000

# Collect static into /app/static (collectstatic will read settings)
RUN python manage.py collectstatic --noinput || true

# ENTRYPOINT: start gunicorn WSGI server (production)
# Bind to all interfaces so container is reachable
CMD ["gunicorn", "conduit.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "3", "--log-level", "info", "--access-logfile", "/var/log/conduit/access.log", "--error-logfile", "/var/log/conduit/error.log"]
