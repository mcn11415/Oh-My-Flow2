# SERVICES MODULE GUIDE

## OVERVIEW
`src/services` contains the operational core: upstream Flow HTTP calls, generation orchestration, token rotation, proxy selection, cache management, concurrency control, and browser captcha runtimes.

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Change upstream request headers / proxy logic | `flow_client.py` | central HTTP client; very high blast radius |
| Add or adjust supported models | `generation_handler.py` | `MODEL_CONFIG` and generation orchestration start here |
| Change token rotation / snapshots | `token_manager.py` | coordinates usable tokens and refresh state |
| Change load selection | `load_balancer.py` | token choice strategy |
| Change concurrency limits or gating | `concurrency_manager.py` | request admission / slot control |
| Change cache retention / cleanup | `file_cache.py` | `/tmp` serving behavior depends on this |
| Change proxy behavior | `proxy_manager.py` | request/media proxy config plumbing |
| Change browser captcha behavior | `browser_captcha.py`, `browser_captcha_personal.py` | headed/on-demand vs resident-tab personal mode |

## STRUCTURE
```text
src/services/
├── generation_handler.py         # model map + request orchestration
├── flow_client.py                # upstream HTTP client + payload building
├── token_manager.py              # token lifecycle and refresh orchestration
├── load_balancer.py              # token/account selection logic
├── concurrency_manager.py        # slot and concurrency controls
├── proxy_manager.py              # request/media proxy resolution
├── file_cache.py                 # cached artifact lifecycle
├── browser_captcha.py            # headed browser captcha mode
├── browser_captcha_personal.py   # personal/resident-tab captcha mode
└── __init__.py
```

## CONVENTIONS
- Keep product capability definitions in `MODEL_CONFIG`; orchestration code consumes that table rather than reinventing per-model branches elsewhere.
- Preserve async boundaries. Most methods coordinate network, DB, and browser state and are designed for cooperative concurrency.
- `FlowClient` owns low-level upstream payload and header behavior; higher-level handlers should not reconstruct raw request details unnecessarily.
- Browser-related modes are operationally different on purpose. `personal` mode optimizes resident tabs; `browser` mode is a separate path.
- Request pacing, concurrency gates, and token selection are explicit design concerns; avoid “simplifying” them without reading the surrounding coordination code.

## ANTI-PATTERNS
- Do not add a new model in `generation_handler.py` without checking resolver logic, tier behavior, API compatibility, and tests.
- Do not bypass `FlowClient` for ad hoc upstream requests; shared headers, proxy routing, fingerprints, and auth handling live there.
- Do not collapse browser captcha modes into one abstraction if their runtime requirements differ.
- Do not treat `generation_handler.py` or `flow_client.py` as safe refactor targets during bugfixes; both are high-complexity files with broad downstream impact.
- Do not break the legacy-vs-project-scoped upload distinctions without updating upload tests.

## NOTES
- `generation_handler.py` and `flow_client.py` are the largest operational hotspots in the repository.
- Existing tests emphasize resolver semantics, upload payloads, and personal-browser behavior; use those as regression anchors when changing service logic.
