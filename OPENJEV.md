# OpenJEV Support

This fork adds optional support for [OpenJEV](https://openjev.sh), a free community gateway to the same Jev model
built by [TypeSafe](https://typesafe.ai). TypeSafe remains the default; anyone with a `TYPESAFE_API_KEY` sees zero
behaviour change.

## What was added

| File | Change |
|------|--------|
| `src/commonMain/kotlin/com/pambrose/jev4k/JevConfig.kt` | Added `JevProvider` enum (`TYPESAFE`, `OPENJEV`), `OPENJEV_API_KEY_ENV`/`OPENJEV_BASE_URL`/`OPENJEV_MODEL`/`PROVIDER_ENV` constants in `JevDefaults`, `provider` property in `JevConfigBuilder`, provider-aware key/baseUrl/model resolution in `build()`, `provider` field in `JevConfig` |
| `.env.example` | Documented `OPENJEV_API_KEY` and `JEV_PROVIDER` env vars |
| `README.md` | Added OpenJEV note after the intro and a "Running with OpenJEV" section |

No TypeSafe code paths were removed, renamed, or re-defaulted. The existing `TYPESAFE_API_KEY`, `TYPESAFE_BASE_URL`,
and `TYPESAFE_DEFAULT_MODEL` env vars work exactly as before when TypeSafe is the selected provider.

## Provider selection rule

1. **Explicit choice wins** — `provider = JevProvider.OPENJEV` in the builder, or `JEV_PROVIDER=openjev` env var.
2. **Otherwise, if `TYPESAFE_API_KEY` is set** → TypeSafe (unchanged default).
3. **Otherwise, if only `OPENJEV_API_KEY` is set** → OpenJEV.

When OpenJEV is selected, the defaults become endpoint `https://api.openjev.sh`, model `openjev`, and the key is read
from `OPENJEV_API_KEY`. Explicit `apiKey`, `baseUrl`, and `defaultModel` always override any provider default. The
`TYPESAFE_BASE_URL` and `TYPESAFE_DEFAULT_MODEL` env vars remain generic overrides regardless of provider (they are
already used this way for Ollaya).

OpenJEV returns HTTP 503 on overload; the existing `RetryPolicy.retryStatuses` (408, 429, 500–599) already covers it.

## Configuration

```bash
# Option 1: auto-select (TypeSafe key unset, OpenJEV key set)
export OPENJEV_API_KEY=oj-...

# Option 2: explicit, when both keys are present
export TYPESAFE_API_KEY=ts-...
export OPENJEV_API_KEY=oj-...
export JEV_PROVIDER=openjev
```

```kotlin
// In code
val jev = JevClient {
    provider = JevProvider.OPENJEV
    apiKey = "oj-..."           // or let it read OPENJEV_API_KEY
}
```

## Verification

A live `POST https://api.openjev.sh/v1/systemone` request with model `openjev`, state `ping`, and one noul question
returned HTTP 200. No repo code was executed during verification.

## Upstream

Original project: https://github.com/pambrose/jev4k by @pambrose (Apache License 2.0).
