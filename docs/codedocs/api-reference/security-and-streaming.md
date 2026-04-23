---
title: "security-and-streaming"
description: "API reference for secret schemas, secret references, SSRF-aware networking, security helpers, and channel streaming compatibility."
---

Import paths covered on this page:

- `openclaw/plugin-sdk/secret-input`
- `openclaw/plugin-sdk/secret-ref-runtime`
- `openclaw/plugin-sdk/security-runtime`
- `openclaw/plugin-sdk/ssrf-runtime`
- `openclaw/plugin-sdk/channel-secret-runtime`
- `openclaw/plugin-sdk/channel-streaming`

These modules define the public security-facing boundary for plugins. They cover how plugins represent secrets, how they resolve and validate secret references, and how they normalize streaming behavior across legacy and current channel config shapes.

## secret-input and secret-ref-runtime

From `src/plugin-sdk/secret-input.ts`:

```ts
function buildOptionalSecretInputSchema(): ReturnType<typeof buildSecretInputSchema>["optional"]
function buildSecretInputArraySchema(): z.ZodArray<ReturnType<typeof buildSecretInputSchema>>
```

Important re-exported helpers include `buildSecretInputSchema`, `coerceSecretRef`, `hasConfiguredSecretInput`, `isSecretRef`, `resolveSecretInputString`, `normalizeResolvedSecretInputString`, and `normalizeSecretInputString`.

`src/plugin-sdk/secret-input-schema.ts` defines the actual public schema. It accepts plain strings or discriminated objects with `source: "env" | "file" | "exec"` and validates `provider` plus the specific secret reference ID format.

`secret-ref-runtime` is narrower and only exports:

```ts
coerceSecretRef
type SecretInput
type SecretRef
```

Use it when runtime code only needs normalized secret refs, not the full zod schema bundle.

## security-runtime and ssrf-runtime

`security-runtime` re-exports shared channel security, context visibility, external content wrapping, and safe regex helpers. `ssrf-runtime` adds the network guard layer, including `fetchWithSsrFGuard`, `isPrivateOrLoopbackHost`, policy helpers, and `formatErrorMessage`.

`channel-secret-runtime` complements those modules with secret-target registry and runtime secret-collector helpers that channel plugins can use when surfacing account-scoped secret assignments.

## channel-streaming

Custom functions from `src/plugin-sdk/channel-streaming.ts`:

```ts
function getChannelStreamingConfigObject(...)
function resolveChannelStreamingChunkMode(...)
function resolveChannelStreamingBlockEnabled(...)
function resolveChannelStreamingBlockCoalesce(...)
function resolveChannelStreamingPreviewChunk(...)
function resolveChannelStreamingPreviewToolProgress(...)
function resolveChannelStreamingNativeTransport(...)
function resolveChannelPreviewStreamMode(
  entry: StreamingCompatEntry | null | undefined,
  defaultMode: "off" | "partial",
): "off" | "partial" | "block"
```

These functions exist because channel config still has to tolerate legacy alias shapes. The implementation explicitly checks both nested `streaming` objects and older fields like `streamMode`, `blockStreaming`, `draftChunk`, and `nativeStreaming`.

## Example

```ts
import {
  buildOptionalSecretInputSchema,
  hasConfiguredSecretInput,
} from "openclaw/plugin-sdk/secret-input";
import { resolveChannelPreviewStreamMode } from "openclaw/plugin-sdk/channel-streaming";

const SecretSchema = buildOptionalSecretInputSchema();
const parsed = SecretSchema.parse({ source: "env", provider: "default", id: "OPENAI_API_KEY" });

const previewMode = resolveChannelPreviewStreamMode(
  { streaming: { mode: "block" } },
  "partial",
);

console.log(hasConfiguredSecretInput(parsed), previewMode);
```

Use this page's modules whenever a plugin crosses a trust boundary: user-provided secrets, external content, or network destinations. They exist specifically to keep those operations inside the published SDK contract.
