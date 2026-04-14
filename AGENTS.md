# PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-15 Asia/Shanghai
**Commit:** d844f48
**Branch:** main

## OVERVIEW
Flow2API is a Python 3.8+ FastAPI service that exposes OpenAI-compatible and Gemini-compatible generation APIs for Flow/VideoFX. The repo is Docker-first, stores runtime state in SQLite, and mixes API routing, browser-assisted captcha flows, token orchestration, and model-specific request shaping.

## STRUCTURE
```text
.
├── main.py                  # thin uvicorn launcher; real app wiring lives in src/main.py
├── src/
│   ├── main.py              # FastAPI app, lifespan, dependency wiring
│   ├── api/                 # public generation endpoints + admin APIs
│   ├── core/                # config, auth, db, models, resolver, logger
│   └── services/            # upstream client, orchestration, captcha, cache, balancing
├── config/                  # TOML defaults and deployment-specific settings
├── tests/                   # unittest suites for resolver, upload, browser-captcha logic
├── static/                  # login/manage/test HTML pages
├── docker/                  # headed container entrypoint helpers
└── .github/workflows/       # Docker publish pipelines
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Start the server | `main.py`, `src/main.py` | root file only runs uvicorn; app lifecycle is in `src/main.py` |
| Add or change public generation behavior | `src/api/routes.py` | OpenAI + Gemini request normalization lives here |
| Change admin or diagnostics APIs | `src/api/admin.py` | large file; includes health checks and management endpoints |
| Add supported models | `src/services/generation_handler.py`, `src/core/model_resolver.py` | config map + alias resolution both matter |
| Change upstream HTTP behavior | `src/services/flow_client.py` | request headers, proxy choice, upload/generation payloads |
| Change token rotation / concurrency | `src/services/token_manager.py`, `src/services/load_balancer.py`, `src/services/concurrency_manager.py` | these modules coordinate request admission |
| Change browser captcha flows | `src/services/browser_captcha.py`, `src/services/browser_captcha_personal.py` | `browser` and `personal` are intentionally separate modes |
| Change persistent config or schema | `src/core/database.py`, `src/core/config.py`, `config/setting.toml` | startup syncs TOML into SQLite-backed runtime state |
| Update web UI pages | `static/login.html`, `static/manage.html`, `static/test.html` | server only serves files; UI logic stays in static assets |
| Check deployment / release behavior | `Dockerfile*`, `docker-compose*.yml`, `.github/workflows/*.yml` | repo is built around Docker variants |

## CODE MAP
| Symbol / Asset | Type | Location | Role |
|----------------|------|----------|------|
| `app` | FastAPI app | `src/main.py` | top-level ASGI app exported to uvicorn |
| `lifespan` | async context manager | `src/main.py` | startup DB init, browser warmup, cleanup tasks |
| `router` | APIRouter | `src/api/routes.py` | generation and model listing endpoints |
| `router` | APIRouter | `src/api/admin.py` | admin, config, token, health endpoints |
| `Config` | class | `src/core/config.py` | TOML-backed runtime config accessors |
| `Database` | class | `src/core/database.py` | SQLite schema, migrations, runtime state storage |
| `resolve_model_name` | function | `src/core/model_resolver.py` | model alias / aspect-ratio resolution |
| `MODEL_CONFIG` | dict | `src/services/generation_handler.py` | authoritative model capability table |
| `GenerationHandler` | class | `src/services/generation_handler.py` | orchestrates request handling and caching |
| `FlowClient` | class | `src/services/flow_client.py` | upstream HTTP client and payload construction |

## CONVENTIONS
- Keep the root `main.py` thin; real application behavior belongs in `src/main.py` or deeper modules.
- The codebase is async-first. Database access, upstream calls, and browser interactions all follow `async`/`await` patterns.
- Runtime config is not just file-based: `config/setting.toml` seeds SQLite on first boot, then admin-managed values can override in-memory config.
- Public API compatibility is dual-track: OpenAI-compatible shapes and Gemini official `generateContent` / `streamGenerateContent` shapes share internal normalization.
- Tests use standard-library `unittest`, with `IsolatedAsyncioTestCase` plus `AsyncMock` for async behavior.
- Deployment conventions assume Docker first; local Python execution exists but is secondary.

## ANTI-PATTERNS (THIS PROJECT)
- Do not treat root `main.py` as the place for feature logic; it is only a launcher.
- Do not add model support in only one place. New or changed models often require updates in both `MODEL_CONFIG` and resolver / payload logic.
- Do not mix captcha modes conceptually. `browser`, `personal`, remote browser, and API-solvers have distinct operational paths.
- Do not exceed upstream media constraints documented by the project: R2V currently caps at 3 reference images; lite interpolation / lite i2v have stricter image-count rules.
- Do not rely on standard `docker-compose.yml` for headed browser captcha flows; headed behavior belongs to the dedicated headed image / compose variant.
- Do not assume streaming is optional for generation paths; examples and endpoint design are oriented around streamed progress/results.
- Do not leave default admin credentials in place on a real deployment.

## UNIQUE STYLES
- Inline comments are often bilingual or Chinese-first around operational nuances.
- Large service files encode product behavior directly in dictionaries and helper branches rather than abstract plugin layers.
- Browser warmup, resident tabs, token snapshots, and concurrency shaping are treated as first-class runtime concerns rather than side utilities.

## COMMANDS
```bash
python -m unittest discover tests
python main.py
docker compose up -d --build
docker compose -f docker-compose.headed.yml up -d --build
docker compose -f docker-compose.local.yml up -d --build
docker compose -f docker-compose.proxy.yml up -d
docker compose logs -f
```

## NOTES
- There is no lint/type tool configuration checked in; follow existing local style and verify with tests / runtime-oriented checks.
- `src/api/admin.py`, `src/core/database.py`, `src/services/flow_client.py`, and `src/services/generation_handler.py` are major complexity hotspots.
- GitHub Actions only publish Docker images; they are not the primary source of test discipline in this repo.
- Child AGENTS files under `src/api`, `src/core`, and `src/services` contain the local navigation guidance that would be too noisy here.
