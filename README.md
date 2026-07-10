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
