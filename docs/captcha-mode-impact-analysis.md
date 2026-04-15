# Captcha Mode Impact Analysis

## Background

This project recently received a series of changes focused on making the `personal` captcha mode work more reliably in a containerized Zeabur environment. During debugging, several supporting improvements were also added around runtime diagnostics, health visibility, Docker image contents, and debug logging behavior.

Because those changes span multiple files, it is useful to separate:

- changes that are **strictly personal-mode specific**
- changes that are **shared infrastructure / admin / diagnostics**
- changes that are **deployment-wide**, even if motivated by personal mode

This document summarizes the practical impact of those changes on other captcha modes.

---

## Executive Summary

### What changed mainly for `personal`

The most substantial runtime behavior changes are concentrated in `src/services/browser_captcha_personal.py`.

These include:

- stricter startup readiness semantics
- finite startup retry behavior
- separation between "browser launchable" and "personal usable"
- resident-tab warmup success requirements
- richer browser startup diagnostics

These do **not** directly affect:

- third-party API captcha modes (`yescaptcha`, `capmonster`, `ezcaptcha`, `capsolver`)
- `remote_browser`
- the ordinary `browser` mode implementation path

### What changed globally

Some changes were made at shared layers and therefore affect all modes indirectly:

- debug logger behavior
- debug config persistence / hot reload
- admin health page visibility
- default Docker image contents and startup path

These do not rewrite the business logic of other captcha modes, but they do change operational behavior such as:

- where logs appear
- whether debug flags persist across restart
- image size / build time / browser dependencies included in the default image

---

## Impact by Captcha Mode

## 1. `personal`

This is the primary mode targeted by the recent changes.

### Directly impacted

- startup and warmup flow
- browser startup retry logic
- runtime readiness / usability state
- browser diagnostics reporting
- health-page status semantics

### Risk profile

This mode now behaves differently than before by design. That is expected and intended.

The main purpose of the changes was to reduce false-positive health states such as:

- browser process launched, but no usable resident tab exists
- startup probe succeeds, but reCAPTCHA flow is not actually usable

---

## 2. `browser`

The `browser` mode logic itself was **not** intentionally refactored in this round.

### Direct runtime logic impact

- minimal

### Indirect operational impact

- default Docker image now contains browser runtime dependencies
- debug logging may now also appear in container logs if debug is enabled
- admin diagnostics pages expose more runtime information

### Practical conclusion

`browser` mode should not have its core captcha workflow changed by these edits, but it now shares a more browser-friendly default deployment image.

---

## 3. Third-party API captcha modes

Includes:

- `yescaptcha`
- `capmonster`
- `ezcaptcha`
- `capsolver`

### Direct runtime logic impact

- none in captcha-solving flow

### Indirect operational impact

- debug config persistence/hot reload now affects whether related request debug output appears
- health page now exposes more runtime/admin diagnostics
- default Docker image contains extra browser dependencies even if these modes do not need them

### Practical conclusion

These modes should keep working as before at the business-logic level. The main difference is operational: more consistent diagnostics and a heavier default image.

---

## 4. `remote_browser`

### Direct runtime logic impact

- none in remote-browser request flow

### Indirect operational impact

- health page can better distinguish whether local browser runtime is relevant or not
- debug config and logger behavior are shared

### Practical conclusion

`remote_browser` was not the target of the recent fix series. It mainly benefits from improved admin diagnostics and clearer runtime visibility.

---

## Shared / Cross-Cutting Changes

## 1. Debug configuration now behaves more consistently

Recent changes fixed a mismatch between:

- `config/setting.toml`
- database-backed runtime config
- in-memory `config`

Previously, debug settings could appear enabled in one place but remain ineffective in the running process.

Now:

- debug settings persist to the database
- runtime reload pulls all debug-related subfields back into memory
- health page metadata better reflects runtime truth

This is a **global improvement**, not personal-only.

---

## 2. `debug_logger` now also emits to container-visible output

Previously, `debug_logger` wrote only to `logs.txt`.

Now it also has a stream handler, which means debug output can appear in platform/container logs as long as debug mode is enabled.

This affects all modes equally.

---

## 3. Default Docker image is now browser-capable

The root `Dockerfile` was aligned with the headed/browser runtime path, including:

- browser runtime system libraries
- Playwright Chromium installation
- headed entrypoint usage

This was driven by personal-mode needs, but it affects the default deployment image globally.

### Operational side effects

- larger image size
- slower build time
- more browser-related packages in every deployment image

### Functional side effects

It should not break non-browser modes by itself, but it changes the baseline deployment environment for all modes.

---

## Overall Risk Assessment

### Low risk to other captcha business logic

The recent code changes do **not** intentionally rewrite the solving logic of:

- third-party API captcha modes
- `remote_browser`
- standard `browser` flow internals

### Medium operational impact globally

The following global behaviors changed and may affect operator expectations:

- debug logging visibility
- debug persistence semantics
- admin health diagnostics
- default image composition

### Highest behavioral change area

The highest behavioral change remains:

- `personal` mode startup, readiness, and warmup lifecycle

---

## Recommendation

When evaluating regressions after this work:

1. treat `personal` as the primary changed mode
2. treat admin/debug/deployment changes as shared infrastructure changes
3. do not assume a regression in another mode means its solving logic was rewritten
4. first check whether the issue is caused by:
   - new debug behavior
   - new Docker image/runtime assumptions
   - or only the `personal` state machine

---

## Bottom Line

The recent changes are **mostly personal-mode specific at the runtime logic level**, but **global at the diagnostics and deployment level**.

So the safest summary is:

- **business logic impact**: mostly `personal`
- **observability / debug impact**: all modes
- **default deployment image impact**: all modes
