# python-base

Base image for **FastAPI** (Uvicorn) and **Flask** (Gunicorn) applications.

**Image:** `orbisdata/python-base:lts`  
**Base:** `python:3.12-slim`  
**User:** `appuser` (non-root, uid 1001)

## What's included

- Python 3.12 slim
- `curl` (for health checks in docker-compose)
- Non-root `appuser`
- Optimized Python flags: no `.pyc` files, unbuffered stdout, fault handler enabled

## FastAPI + Uvicorn

```dockerfile
# syntax=docker/dockerfile:1

# Stage 1 — install dependencies
FROM orbisdata/python-base:lts AS deps
COPY requirements.txt ./
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2 — final image
FROM orbisdata/python-base:lts AS runner
COPY --from=deps /home/appuser/.local /home/appuser/.local
COPY --chown=appuser:appuser . .

ENV PATH=/home/appuser/.local/bin:$PATH

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "1"]
```

## Flask + Gunicorn

```dockerfile
# syntax=docker/dockerfile:1

FROM orbisdata/python-base:lts AS deps
COPY requirements.txt ./
RUN pip install --user --no-cache-dir -r requirements.txt

FROM orbisdata/python-base:lts AS runner
COPY --from=deps /home/appuser/.local /home/appuser/.local
COPY --chown=appuser:appuser . .

ENV PATH=/home/appuser/.local/bin:$PATH

EXPOSE 8000
CMD ["gunicorn", "app:create_app()", "--bind", "0.0.0.0:8000", "--workers", "2", "--worker-class", "gthread", "--threads", "2"]
```

## Workers configuration

| Runtime | Recommended workers | Formula |
|---------|-------------------|---------|
| Uvicorn | 1–2 per container | Let ECS/EKS scale horizontally |
| Gunicorn | `(2 * CPU) + 1` | Adjust via env var |

Pass worker count at runtime:

```dockerfile
CMD ["sh", "-c", "gunicorn app:create_app() --bind 0.0.0.0:8000 --workers ${GUNICORN_WORKERS:-2}"]
```

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PYTHONDONTWRITEBYTECODE` | `1` | No `.pyc` files |
| `PYTHONUNBUFFERED` | `1` | Stdout/stderr unbuffered (required for CloudWatch logs) |
| `PYTHONFAULTHANDLER` | `1` | Dump traceback on crash |
