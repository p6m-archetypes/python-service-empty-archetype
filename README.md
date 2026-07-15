# python-service-empty-archetype

Python **Service Platform Overlay** — generates only the platform *servicing layer* for a
service and nothing else. Run it against an **existing** Python project to retrofit it with:

- `.github/workflows/` — CI build + cut-tag (`python-ci`)
- `.platform/kubernetes/**` — PlatformApplication CRD + dev/stg/prd kustomize overlays
  (`platform-application-manifests`), including `resourceRequirements` when you select a
  resource
- `.platform/docker/{local,prd}/Dockerfile` — container builds (idiomatic starting points;
  adapt to your build)
- `.editorconfig`, `.gitattributes`, `.gitignore`

It generates **no** project scaffolding and **no** domain code — no `pyproject.toml`, no
`src/`, no `tests/`. Everything renders **in place** at the destination root.

```bash
archetect render git@github.com:p6m-archetypes/python-service-empty-archetype.git#dev /path/to/existing-project
```

## Resources

You are prompted for persistence / cache / messaging so the PlatformApplication can
declare the matching `resourceRequirements`. **No connection code is generated** — this
overlay never touches project source. (Object storage is intentionally omitted: it drives
only code weaving, which does not apply to a retrofit.)

See `docs/specs/empty-service-archetype.md` in the archetype-ecosystem repo for the full
contract. This is the empty-tier sibling of `python-rest-service-archetype` (full) and
`python-service-basic-archetype` (minimal complete service).

## Acceptance tests

This archetype ships a [prova](https://github.com/prova-rs/prova) acceptance suite (`tests/`,
driven by `prova.toml`) that renders the overlay in-process and validates it. Because the
overlay generates only the platform servicing layer (no project scaffolding, nothing to build
or boot), the suite is purely **static**: it checks that the expected platform files render,
that no project/domain code is emitted, that the Kubernetes manifests parse and are wired with
the project identity + image registry, and that no template markers leak through. No language
toolchain is required.

### Install prova

```shell
brew tap prova-rs/tap
brew install prova
```

Or download a binary for your platform from the
[prova releases](https://github.com/prova-rs/prova/releases) and put it on your `PATH`.

### Prerequisites

- **Network access + git** — the suite renders the overlay in-process (prova embeds archetect),
  which fetches the composed libraries over HTTPS from GitHub. No other toolchain is needed.

### Run

From the repo root:

```shell
prova                               # run the whole suite (uses ./prova.toml)
prova --profile ci                  # the profile CI runs (JSON output)
prova tests/empty_overlay_test.lua  # a single test file
```

CI runs the same suite via [`prova-rs/run-action`](https://github.com/prova-rs/run-action)
(see `.github/workflows/acceptance.yaml`).
