# MetaEngine OpenAPI Fetch

[![npm version](https://img.shields.io/npm/v/@metaengine/openapi-fetch.svg)](https://www.npmjs.com/package/@metaengine/openapi-fetch)
[![npm downloads](https://img.shields.io/npm/dm/@metaengine/openapi-fetch.svg)](https://www.npmjs.com/package/@metaengine/openapi-fetch)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Generate framework-agnostic TypeScript services and models from OpenAPI/Swagger specifications using the native Fetch API.**

Runs on Node, browsers, Vite, SvelteKit, Next.js, Bun, Deno, and any TS runtime that ships `fetch`. No runtime dependencies in the generated code.

---

## Quick Links

- **NPM Package**: [@metaengine/openapi-fetch](https://www.npmjs.com/package/@metaengine/openapi-fetch)
- **NuGet Package**: [MetaEngine.TypeScript.OpenApi.Fetch](https://www.nuget.org/packages/MetaEngine.TypeScript.OpenApi.Fetch)
- **Website**: [metaengine.eu/packages/openapi-fetch](https://www.metaengine.eu/packages/openapi-fetch)

---

## Features

- ✅ **Framework-agnostic** — works in any TS runtime with `fetch`
- ✅ **Native Fetch API** — zero runtime dependencies in generated code
- ✅ **Production-ready** — Bearer auth, timeouts, retries, custom headers, middleware, all via CLI flags
- ✅ **TypeScript** — fully typed clients and models
- ✅ **Result pattern** — opt into `ApiResult<T>` for structured error handling without exceptions
- ✅ **Middleware hooks** — `onRequest` / `onResponse` / `onError` composable around the request pipeline
- ✅ **Vite + SvelteKit** — `--import-meta-env` for `import.meta.env` access
- ✅ **Tree-shakeable** — separate file per model and per service

---

## Installation

```bash
npm install --save-dev @metaengine/openapi-fetch
```

Or use directly with npx:

```bash
npx @metaengine/openapi-fetch <input> <output>
```

---

## Requirements

- Node.js 18.0 or later
- .NET 8.0 or later runtime ([Download](https://dotnet.microsoft.com/download))
- A TS runtime that supports `fetch` (Node 18+, all modern browsers, Bun, Deno)

---

## Quick Start

### Basic

```bash
npx @metaengine/openapi-fetch api.yaml ./src/api \
  --documentation \
  --result-pattern
```

### Production setup

```bash
npx @metaengine/openapi-fetch api.yaml ./src/api \
  --bearer-auth API_TOKEN \
  --timeout 30 \
  --retries 3 \
  --custom-header X-Tenant-ID=TENANT_ID \
  --error-handling
```

### With npm scripts

Add to your `package.json`:

```json
{
  "scripts": {
    "generate:api": "metaengine-openapi-fetch api.yaml ./src/api --bearer-auth API_TOKEN --timeout 30 --retries 3"
  }
}
```

Then run:
```bash
npm run generate:api
```

### Runtime-specific examples

**Vite / SvelteKit** (uses `import.meta.env`):
```bash
npx @metaengine/openapi-fetch api.yaml ./src/api \
  --import-meta-env \
  --base-url-env VITE_API_URL
```

**Next.js**:
```bash
npx @metaengine/openapi-fetch api.yaml ./src/api \
  --base-url-env NEXT_PUBLIC_API_URL
```

**Plain Node / Bun / Deno**:
```bash
npx @metaengine/openapi-fetch api.yaml ./src/api \
  --base-url-env API_BASE_URL
```

### More examples

```bash
# From URL
npx @metaengine/openapi-fetch https://api.example.com/openapi.json ./src/api

# Filter by OpenAPI tags
npx @metaengine/openapi-fetch api.yaml ./src/api --include-tags users,auth

# Multiple custom headers from env vars
npx @metaengine/openapi-fetch api.yaml ./src/api \
  --custom-header X-Tenant-ID=TENANT_ID \
  --custom-header X-App-Id=APP_ID
```

---

## CLI Options

| Option | Description | Default |
|--------|-------------|---------|
| `--include-tags <tags>` | Filter by OpenAPI tags (comma-separated, case-insensitive) | - |
| `--base-url-env <name>` | Environment variable name for base URL | `API_BASE_URL` |
| `--service-suffix <suffix>` | Service naming suffix | `Api` |
| `--options-threshold <n>` | Parameter count for options object | `4` |
| `--documentation` | Generate JSDoc comments | `false` |
| `--import-meta-env` | Use `import.meta.env` for env access (Vite, SvelteKit) | `false` |
| `--result-pattern` | Return `ApiResult<T>` instead of `T` for structured error handling | `false` |
| `--middleware` | Emit middleware hooks (`onRequest`, `onResponse`, `onError`) in client | `false` |
| `--error-handling` | Smart error handling based on HTTP status semantics | `false` |
| `--bearer-auth <env-var-name>` | Bearer token from env var (adds `Authorization: Bearer <token>`) | - |
| `--timeout <seconds>` | Request timeout in seconds for all operations | - |
| `--retries <max-attempts>` | Enable retries with exponential backoff (status codes 429, 503) | - |
| `--custom-header <header=envVarName>` | Static header from env var. Repeatable. | - |
| `--strict-validation` | Strict OpenAPI validation | `false` |
| `--date-transformation` | Convert date strings in responses to `Date` objects | `false` |
| `--clean` | Clean output directory (remove files not in generation) | `false` |
| `--verbose` | Enable verbose logging | `false` |
| `--type-mapping <slug=target>` | Override TS type for an OpenAPI format. Repeatable. See [Type mapping overrides](#type-mapping-overrides) | - |
| `--help, -h` | Show help message | - |

---

## Error response models

Schemas used by declared error responses are generated alongside request and success-response models. Their referenced enums and base models are included when filtering operations with `--include-tags`; no additional flag is needed.

```bash
npx @metaengine/openapi-fetch api.json ./src/api --include-tags Things --types-barrel
```

For a `422` response referencing `ThingsProblemDetails`, whose `code` property references `ThingsErrorCode`, the output includes `models/things-problem-details.ts` and `models/things-error-code.ts`. Inline error bodies also receive named models.

Generated clients parse `application/json` and structured `+json` error responses, including `application/problem+json`, into `HttpError.body`. Media types are matched without regard to case or charset parameters.

These types can be imported when handling validated error payloads. Generating them does not perform runtime validation of the server response.

## Generated Code Structure

```
output/
  ├── models/                    # One file per model
  │   ├── user.ts               # export interface User { ... }
  │   ├── product.ts
  │   └── ...
  ├── api/                       # One file per service/tag
  │   ├── users.api.ts          # All user operations
  │   ├── products.api.ts
  │   └── ...
  ├── client.ts                  # ClientConfig, createClient, getDefaultClient
  └── errors.ts                  # ApiError, error helpers
```

---

## Production-ready features

### Bearer authentication

```bash
npx @metaengine/openapi-fetch api.yaml ./src/api --bearer-auth API_TOKEN
```

Generated `client.ts` reads `process.env.API_TOKEN` (or `import.meta.env.API_TOKEN` with `--import-meta-env`) and adds `Authorization: Bearer <token>` to every request. Missing env vars produce a clear error message at startup, not a silent 401.

### Timeout

```bash
npx @metaengine/openapi-fetch api.yaml ./src/api --timeout 30
```

Uses `AbortSignal.timeout(seconds * 1000)` and composes correctly with consumer-supplied `AbortSignal` via `AbortSignal.any`.

### Retries with exponential backoff

```bash
npx @metaengine/openapi-fetch api.yaml ./src/api --retries 5
```

Retries on `429` and `503` only (configurable defaults from the underlying engine). Exponential backoff: base 0.5s, max delay 30s.

### Custom headers from env vars

```bash
npx @metaengine/openapi-fetch api.yaml ./src/api \
  --custom-header X-Tenant-ID=TENANT_ID \
  --custom-header X-App-Id=APP_ID
```

Repeatable. Each header value comes from a separate env var, validated at startup.

### Result pattern

```bash
npx @metaengine/openapi-fetch api.yaml ./src/api --result-pattern
```

Operations return `ApiResult<T>` instead of throwing — useful when you want to handle errors as values rather than exceptions:

```typescript
const result = await usersApi.getUser('123');
if (result.ok) {
  console.log(result.data.name);
} else {
  console.error(result.error.code, result.error.message);
}
```

### Middleware

```bash
npx @metaengine/openapi-fetch api.yaml ./src/api --middleware
```

Generated client emits a `Middleware` interface and accepts an optional middleware array on `ClientConfig`:

```typescript
const client = createClient({
  baseUrl: 'https://api.example.com',
  middleware: [
    {
      onRequest: (req) => { console.log('→', req.url); return req; },
      onResponse: (res) => { console.log('←', res.status); return res; },
      onError: (err) => { console.error('✗', err); throw err; },
    },
  ],
});
```

Middleware composes with `--bearer-auth`, `--timeout`, and `--error-handling`.

---

## Type mapping overrides

Use `--type-mapping` to override the TS type emitted for a given OpenAPI `(type, format)` pair. Repeatable. Unknown slugs and unknown targets are hard errors.

| Slug | OpenAPI `(type, format)` | Default | `--type-mapping` value |
|------|--------------------------|---------|------------------------|
| `int64` | `(integer, int64)` | `number` | `int64=bigint` |
| `decimal` | `(number, decimal)` | `number` | `decimal=string` |
| `date-time` | `(string, date-time)` | `Date` | `date-time=string` |
| `date` | `(string, date)` | `Date` | `date=string` |

```bash
npx @metaengine/openapi-fetch api.yaml ./src/api \
  --type-mapping int64=bigint \
  --type-mapping date-time=string
```

---

## See it live

Try the generator with your own spec at <https://www.metaengine.eu/converters>.

---

## Programmatic Usage

The NuGet package allows programmatic use in .NET projects. See the [website documentation](https://www.metaengine.eu/packages/openapi-fetch) for full C# API reference.

---

## Support

- **Issues**: [GitHub Issues](https://github.com/meta-engine/openapi-fetch/issues)
- **Email**: info@metaengine.eu
- **Website**: [metaengine.eu](https://www.metaengine.eu)

---

## License

MIT License - see [LICENSE](./LICENSE) file for details.

---

## About This Repository

This is the **documentation and issue tracking repository** for MetaEngine OpenAPI Fetch. The compiled NPM package is available at [@metaengine/openapi-fetch](https://www.npmjs.com/package/@metaengine/openapi-fetch).

Source code is proprietary, but the package is free to use under MIT license.
