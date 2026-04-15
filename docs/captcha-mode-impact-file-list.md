# Captcha Mode Impact File List

This document lists the main files touched or behaviorally affected during the recent captcha/debug/runtime work, grouped by scope.

---

## A. Files primarily affecting `personal` mode

## 1. `src/services/browser_captcha_personal.py`

### Why it matters
Core runtime logic for personal/resident-tab captcha mode.

### Recent behavior changes
- browser startup diagnostics
- launchable vs usable state separation
- startup retry behavior
- stricter resident warmup success boundary
- stronger invalidation of stale runtime-ready state

### Scope
**Personal mode only**

---

## B. Files affecting shared admin / diagnostics behavior

## 2. `src/api/admin.py`

### Why it matters
Hosts admin APIs used by the management page.

### Recent behavior changes
- runtime health endpoint additions
- browser diagnostics endpoints
- debug log tail endpoint
- debug config update endpoint changed to persist via database and reload runtime config

### Scope
**All modes indirectly** via admin visibility and runtime debug control

---

## 3. `static/manage.html`

### Why it matters
Single-page management console UI.

### Recent behavior changes
- health page additions
- browser diagnostics panel
- debug log tail panel
- health/debug metadata rendering

### Scope
**All modes indirectly** via admin/operator UX only

---

## C. Files affecting shared config/runtime behavior

## 4. `src/core/config.py`

### Why it matters
Runtime configuration object used by service logic.

### Recent behavior changes
- added debug-related setters for runtime hot updates:
  - `set_debug_enabled`
  - `set_debug_log_requests`
  - `set_debug_log_responses`
  - `set_debug_mask_token`

### Scope
**All modes indirectly**

---

## 5. `src/core/database.py`

### Why it matters
Database-backed runtime config source of truth after startup.

### Recent behavior changes
- `reload_config_to_memory()` now syncs full debug subfields instead of only `enabled`
- debug config persistence/update path is used as runtime truth source

### Scope
**All modes indirectly** via runtime config behavior

---

## 6. `src/core/logger.py`

### Why it matters
Implements `debug_logger` used by multiple service layers.

### Recent behavior changes
- still writes to `logs.txt`
- now also emits to stream/console output

### Scope
**All modes indirectly**, but only when debug is enabled

---

## D. Files affecting deployment/runtime environment globally

## 7. `Dockerfile`

### Why it matters
Default build file used by Zeabur in the current deployment path.

### Recent behavior changes
- aligned with headed/browser-friendly runtime
- installs Chromium system libraries
- installs Playwright Chromium
- uses headed entrypoint script

### Scope
**All deployments globally**

---

## 8. `Dockerfile.headed`

### Why it matters
Dedicated headed/browser-capable build path.

### Recent behavior changes
- explicit Chromium runtime dependency installation

### Scope
Primarily browser-capable deployments, but conceptually shared with default image strategy

---

## 9. `template.yaml`

### Why it matters
Zeabur deployment template and inline Docker build definition.

### Recent behavior changes
- browser-capable runtime settings
- Chromium runtime dependencies
- personal-mode-oriented defaults in previous iterations

### Scope
**Template-driven deployments globally**

---

## E. Files used for understanding / observation, not core behavior rewrites

## 10. `src/main.py`

### Why it matters
Startup lifecycle and warmup orchestration.

### Relevant to recent work
- proves where `reload_config_to_memory()` is called
- proves where personal warmup is triggered
- helps explain why DB config overrides TOML at runtime

### Scope
No large direct logic rewrite in the recent changes, but critical for understanding runtime behavior

---

## 11. `src/services/flow_client.py`

### Why it matters
Low-level upstream request construction and debug logger call sites.

### Relevant to recent work
- request/response debug logs are gated by `config.debug_enabled`
- critical for understanding why `logs.txt` remained empty

### Scope
No major direct rewrite in the recent work described here, but central to debugging behavior

---

## Scope Summary Table

| File | Primary Scope | Directly changes solving logic? |
|---|---|---|
| `src/services/browser_captcha_personal.py` | `personal` only | Yes |
| `src/api/admin.py` | shared admin/diagnostics | No |
| `static/manage.html` | shared admin/diagnostics | No |
| `src/core/config.py` | shared runtime config | Indirectly |
| `src/core/database.py` | shared runtime config | Indirectly |
| `src/core/logger.py` | shared debug logging | Indirectly |
| `Dockerfile` | deployment-wide | No business logic change, yes runtime env change |
| `Dockerfile.headed` | browser runtime env | No business logic change |
| `template.yaml` | deployment template | No business logic change |
| `src/main.py` | startup orchestration | Mostly observational/contextual |
| `src/services/flow_client.py` | upstream request path | Mostly observational/contextual in this specific analysis |

---

## Bottom Line

If you want to know which file most likely changed actual captcha runtime behavior:

- **`src/services/browser_captcha_personal.py`** is the main one.

If you want to know which files changed shared operator behavior:

- **`src/api/admin.py`**
- **`static/manage.html`**
- **`src/core/logger.py`**
- **`src/core/database.py`**

If you want to know which files changed deployment/runtime environment:

- **`Dockerfile`**
- **`Dockerfile.headed`**
- **`template.yaml`**
