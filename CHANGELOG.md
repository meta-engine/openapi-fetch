# Changelog

All notable changes to `@metaengine/openapi-fetch` will be documented in this file.

## [1.0.1] - 2026-05-04

### Bug Fixes

- **Fixed: `--error-handling` widens nullable types twice** — when `--error-handling` was enabled, nullable response types could be typed as `T | null | null`. The duplicate widening is now suppressed so the response type is `T | null` as intended.

## [1.0.0] - 2026-04-24

Initial public release.

### Features

- **Framework-agnostic** — runs on Node, browsers, Vite, SvelteKit, Next.js, Bun, Deno, and any TS runtime that ships `fetch`
- **Native Fetch API** — no runtime dependencies in generated code
- **Tree-shakeable** — separate file per model and per service
- **Production-ready out of the box** — Bearer auth, timeouts, retries with exponential backoff, custom headers from env vars, and middleware hooks all available via CLI flags

### CLI Flags

#### Core
- `--include-tags <tags>` — filter operations by OpenAPI tag (comma-separated, case-insensitive)
- `--base-url-env <name>` — environment variable name for base URL (default: `API_BASE_URL`)
- `--service-suffix <suffix>` — service naming suffix (default: `Api`)
- `--options-threshold <n>` — parameter count threshold for options object (default: `4`)
- `--documentation` — generate JSDoc comments
- `--strict-validation` — strict OpenAPI spec validation
- `--date-transformation` — convert `(string, date-time)` and `(string, date)` response fields to `Date` objects
- `--clean` — clean output directory (remove files no longer in generation)
- `--verbose` — verbose logging

#### Production essentials

- `--bearer-auth <env-var-name>` — Bearer token from env var. Adds `Authorization: Bearer <token>` to every request.

  ```bash
  npx @metaengine/openapi-fetch api.yaml ./src/api --bearer-auth API_TOKEN
  ```

- `--timeout <seconds>` — per-request timeout via `AbortSignal.timeout`.

  ```bash
  npx @metaengine/openapi-fetch api.yaml ./src/api --timeout 30
  ```

- `--retries <max-attempts>` — exponential backoff retries on `429` and `503` (base 0.5s, max delay 30s).

  ```bash
  npx @metaengine/openapi-fetch api.yaml ./src/api --retries 3
  ```

- `--custom-header <header=envVarName>` — static header from env var. Repeatable.

  ```bash
  npx @metaengine/openapi-fetch api.yaml ./src/api \
    --custom-header X-Tenant-ID=TENANT_ID \
    --custom-header X-App-Id=APP_ID
  ```

#### Fetch-distinctive

- `--import-meta-env` — use `import.meta.env` for env access (Vite, SvelteKit) instead of `process.env`
- `--result-pattern` — return `ApiResult<T>` instead of throwing, for structured error handling without exceptions
- `--middleware` — emit `Middleware` interface (`onRequest`, `onResponse`, `onError`) and accept a middleware array on `ClientConfig`
- `--error-handling` — smart error handling based on HTTP status semantics (404/403 → null, 400/422/409 → error body, 401/5xx → throw)

#### Type mapping

- `--type-mapping <slug=target>` — opt-in override for the TypeScript type emitted for a given OpenAPI `(type, format)` pair. Repeatable. Unknown slugs or targets are hard errors — no silent fallbacks.

  | Slug | OpenAPI `(type, format)` | Default | Override |
  |------|--------------------------|---------|----------|
  | `int64` | `(integer, int64)` | `number` | `int64=bigint` |
  | `decimal` | `(number, decimal)` | `number` | `decimal=string` |
  | `date-time` | `(string, date-time)` | `Date` | `date-time=string` |
  | `date` | `(string, date)` | `Date` | `date=string` |

  ```bash
  npx @metaengine/openapi-fetch api.yaml ./src/api \
    --type-mapping int64=bigint \
    --type-mapping date-time=string
  ```
