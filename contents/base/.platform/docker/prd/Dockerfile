# prd/Dockerfile — the artifact CI publishes and the platform deploys
#
# Retrofit overlay: this file assumes nothing about the application's layout or about where its
# build installs things. It decides that itself, so the two answers can't disagree with it:
#   - The builder activates a virtual environment at /opt/venv before the build runs, so
#     `pip install .`, `uv pip install .` and `uv sync` all install into it.
#   - A check step makes sure the answered entrypoint (`runtime_artifact`) is inside that
#     environment, and fails naming the answer if it isn't.
#   - The runtime stage copies /opt/venv and the repo (/app). It never copies the builder's
#     /usr/local.
# Lines a team may need to change are marked `CHANGE:`.

# ── Builder: install the application into /opt/venv ──────────────────────────────────────────────
# CHANGE: the Python version. Keep the builder and the runtime on the same minor version; the
# environment links to the interpreter.
FROM python:3.12-slim AS builder
WORKDIR /app
# CHANGE: system packages your dependencies compile against at install time.
RUN apt-get update && apt-get install -y --no-install-recommends gcc && rm -rf /var/lib/apt/lists/*
RUN pip install --no-cache-dir uv
# The environment the build installs into, active for everything below. uv honors VIRTUAL_ENV for
# `uv pip` and UV_PROJECT_ENVIRONMENT for `uv sync`; pip follows PATH.
ENV VIRTUAL_ENV=/opt/venv \
    UV_PROJECT_ENVIRONMENT=/opt/venv \
    UV_LINK_MODE=copy \
    UV_COMPILE_BYTECODE=1 \
    PATH=/opt/venv/bin:$PATH
RUN python -m venv /opt/venv
COPY . .
# The application's own build, from the repo root (answer: build_command).
RUN {{ build_command }}

# ── Check: the answered entrypoint was installed into the environment ────────────────────────────
# Fails the build naming the answer, instead of failing later on a missing file.
RUN entry="$(command -v '{{ runtime_artifact }}' || true)"; \
    case "$entry" in \
      /opt/venv/*|/app/*) ;; \
      "") echo "p6m: runtime_artifact '{{ runtime_artifact }}' was not installed by build_command '{{ build_command }}'." >&2; \
          echo "p6m: console scripts in /opt/venv/bin: $(ls /opt/venv/bin | grep -vE '^(python|pip|activate|Activate|deactivate)' | tr '\n' ' ')" >&2; \
          exit 1 ;; \
      *) echo "p6m: runtime_artifact '{{ runtime_artifact }}' resolved to $entry, outside /opt/venv. build_command must install into the active environment (drop --system / --target)." >&2; \
         exit 1 ;; \
    esac

# ── Runtime: the environment and the repo, nothing else from the builder ─────────────────────────
# CHANGE: keep in step with the builder's Python version.
FROM python:3.12-slim
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates \
    && rm -rf /var/lib/apt/lists/*
RUN addgroup --system --gid 1001 appgroup && \
    adduser --system --uid 1001 --gid 1001 appuser
ENV VIRTUAL_ENV=/opt/venv \
    PATH=/opt/venv/bin:$PATH \
    PYTHONUNBUFFERED=1
COPY --from=builder /opt/venv /opt/venv
# The repo too: editable installs (`uv sync`'s default) and run-from-source apps resolve against it.
COPY --from=builder /app /app
USER appuser
EXPOSE {{ service_port }} {{ management_port }}
ENTRYPOINT ["{{ runtime_artifact }}"]
