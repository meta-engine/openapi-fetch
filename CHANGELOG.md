# Changelog

All notable changes to `@metaengine/openapi-fetch` will be documented in this file.

## [1.1.2] - 2026-09-27

### Bug Fixes

- JSON error bodies using `application/problem+json` or another `+json` media type are now parsed into `HttpError.body`, including mixed-case media types and charset parameters.
- Models referenced only by error responses, together with their enum and base-model dependencies, are now retained when using `--include-tags`.
- Inline error-response bodies now generate named models, including when no tag filter is used.
- Nullable primitive aliases preserve `| null` in generated TypeScript.
- Models using `allOf` preserve required inherited properties and compatible property refinements.

### Dependencies

- Bundles `MetaEngine.TypeScript.OpenApi.Fetch` 1.1.3.
- Updated the bundled OpenAPI reader to Microsoft.OpenApi 3.5.4, which fixes circular-reference parsing failures ([CVE-2026-49451](https://github.com/advisories/GHSA-v5pm-xwqc-g5wc)).

## [1.1.1] - 2026-05-26

### Features

- **OpenAPI 3.1 support** — specs written against OpenAPI 3.1 / JSON Schema 2020-12 are now parsed and generated correctly. 3.0 specs are unaffected.
- **New: `--types-barrel` flag** — emits an `index.ts` barrel per folder (`models/index.ts`, `services/index.ts`) plus a root `index.ts`, so consumers can import everything from one entry point. Service files are re-exported under a namespace (`export * as UsersApi from './users.service'`) to avoid `TS2308` duplicate-identifier errors; models are re-exported flat.

  ```bash
  npx @metaengine/openapi-fetch api.yaml ./src/api --types-barrel
  ```

### OpenAPI 3.1 output

- `oneOf` members of the form `{ "type": "null" }` now contribute `| null` to the property type instead of generating a spurious `Null` interface.
- `const: <value>` on a schema generates a literal type (`channel: 'web'`) instead of falling back to `string`.
- `allOf` combining a `$ref` with an inline override no longer drops the inline branch.
- Array nullability is preserved: `type: ["array","null"]` and nullable items now generate `Array<string | null>` / `Array<string | null> | null` instead of collapsing to `Array<string>`.
- `$ref` JSON-Pointer escapes (`~1`, `~0`) resolve correctly instead of falling back to `unknown`.
- `$dynamicRef` / `$dynamicAnchor` recursive schemas resolve to proper self-referencing types.
- `contentEncoding` / `contentMediaType` binary schemas generate `Blob` output, matching 3.0 `format: binary`.
- Top-level `webhooks:` schemas generate typed payload interfaces.

### Output

- Array-typed query parameters with inline enum items render as `Array<'a' | 'b'>` instead of `Array<string>`.
- Backslash and control characters in inline-enum string literals are now escaped, producing valid TypeScript.
- With `--documentation`, auto-generated `@returns` text uses the resource noun instead of leaking the URL action segment.

### Bug Fixes

- Streaming requests no longer hardcode `Content-Type: application/json`; `FormData` and `Blob` bodies now let the platform set the boundary, fixing multipart uploads.
- Generated service files no longer drop `import` statements when an enum contains URL-like string values (e.g. `'http://...'`).
- `--strict-validation` no longer rejects valid 3.1 specs whose component keys contain `/` or `~`.

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
