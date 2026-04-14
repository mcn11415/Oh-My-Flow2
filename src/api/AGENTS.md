# API MODULE GUIDE

## OVERVIEW
`src/api` defines the external HTTP surface: public generation/model endpoints in `routes.py` and management/diagnostic endpoints in `admin.py`.

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| OpenAI-compatible request parsing | `routes.py` | message/image extraction and normalization live here |
| Gemini-compatible request parsing | `routes.py` | `/models/*:generateContent` compatibility also lives here |
| Model list / catalog output | `routes.py` | uses `MODEL_CONFIG` + base aliases |
| Media URL/data-url loading | `routes.py` | image byte acquisition helpers live near route normalization |
| Admin auth/session endpoints | `admin.py` | token-based admin session management |
| Health / runtime diagnostics | `admin.py` | includes browser/runtime health checks |
| Token / proxy / config management | `admin.py` | wired to database-backed managers |

## STRUCTURE
```text
src/api/
├── routes.py   # public generation endpoints, model listing, compatibility shims
├── admin.py    # admin routes, health checks, token/proxy/config operations
└── __init__.py
```

## CONVENTIONS
- `src/main.py` injects dependencies after constructing services. Avoid hidden singleton wiring inside route files beyond the existing setter pattern.
- Route-layer code is responsible for request normalization and HTTP-shape compatibility, not deep business decisions.
- Keep OpenAI and Gemini compatibility paths aligned through shared internal request shapes instead of duplicating orchestration logic downstream.
- Prefer raising `HTTPException` with concrete request-context errors at the boundary rather than leaking lower-level exceptions.

## ANTI-PATTERNS
- Do not hardcode model metadata in routes when it should come from `MODEL_CONFIG` or resolver helpers.
- Do not bypass normalization helpers for image/data-url parsing; compatibility logic is already fragmented enough.
- Do not move runtime orchestration into API files. Browser startup, balancing, token refresh, and cache policy belong in `services` or `core`.
- Do not add admin diagnostics that drift from actual runtime state; pull from managers / config / DB instead.

## NOTES
- `admin.py` is a large mixed-responsibility file. When touching it, keep related helper functions near their endpoint cluster.
- If a request-shape change affects uploads or model payloads, update `tests/test_flow_client_upload.py` or adjacent request-shape tests.
