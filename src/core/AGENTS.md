# CORE MODULE GUIDE

## OVERVIEW
`src/core` holds infrastructure primitives: config loading, SQLite persistence, auth, shared data models, model-name resolution, and debug logging.

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Read runtime config | `config.py` | TOML-backed accessor properties live here |
| Change DB schema / migrations | `database.py` | startup migration and config seeding logic is centralized here |
| Change auth behavior | `auth.py` | API/admin auth helpers |
| Change shared pydantic/domain models | `models.py` | route/service contracts depend on these |
| Change model alias resolution | `model_resolver.py` | aspect ratio / tier / alias mapping logic |
| Change tier rules | `account_tiers.py` | used by resolver / generation logic |
| Change debug logging | `logger.py` | request/response masking behavior lives here |

## STRUCTURE
```text
src/core/
├── config.py
├── database.py
├── model_resolver.py
├── models.py
├── auth.py
├── account_tiers.py
├── logger.py
└── __init__.py
```

## CONVENTIONS
- `Config` is a runtime accessor layer over TOML, but database-backed values can overwrite in-memory behavior after startup.
- `Database` is more than CRUD; it owns first-boot initialization, compatibility checks, and schema evolution.
- Resolver logic is product logic. Alias mapping, aspect-ratio coercion, and tier upgrades are expected here, not scattered across routes/services.
- Keep shared models stable because both OpenAI-compatible and Gemini-compatible request paths depend on them.

## ANTI-PATTERNS
- Do not add duplicated config parsing in services; expose it as `Config` accessors instead.
- Do not mutate schema assumptions in route/service code without updating `database.py` migration behavior.
- Do not bury model alias rules inside request handlers or client methods; centralize them in `model_resolver.py`.
- Do not add silent fallback behavior around auth/config/schema errors unless the repo already has an explicit operational reason.

## NOTES
- `database.py` is one of the most stateful files in the repo and a major risk area for regressions.
- If changing model resolution semantics, update `tests/test_veo_lite_support.py` or similar resolver-focused tests.
