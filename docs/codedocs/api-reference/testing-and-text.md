---
title: "testing-and-text"
description: "API reference for the public testing harness, shared text helpers, and the zod compatibility subpath."
---

Import paths covered on this page:

- `openclaw/plugin-sdk/testing`
- `openclaw/plugin-sdk/text-runtime`
- `openclaw/plugin-sdk/zod`

These are support modules for plugin packages. They are not registration helpers; they exist so plugin authors can test, sanitize, chunk, and parse text without reaching into private internals.

## testing

`src/plugin-sdk/testing.ts` intentionally keeps the public test surface narrow but useful. Common exports include:

- Registration helpers: `createEmptyPluginRegistry`, `capturePluginRegistration`, `setDefaultChannelPluginRegistryForTests`
- Runtime capture: `createCliRuntimeCapture`, `spyRuntimeLogs`, `spyRuntimeJson`, `spyRuntimeErrors`
- Channel contract helpers: `expectChannelInboundContextContract`, `primeChannelOutboundSendMock`, `installChannelOutboundPayloadContractSuite`
- Gateway and ACP helpers: `callGateway`, `runAcpRuntimeAdapterContract`, `handleAcpCommand`
- Test environment helpers: `captureEnv`, `withEnv`, `withEnvAsync`, `withFetchPreconnect`, `createTempHomeEnv`

Because this module is public, it is a good signal for which testing hooks the OpenClaw maintainers are willing to support over time.

## text-runtime

`text-runtime` is a large re-export bundle. It exposes logging helpers, markdown IR and rendering utilities, string normalization, chunking helpers, reasoning-tag helpers, terminal-safe text helpers, and utility functions like `truncateUtf16Safe` and `safeParseJson`.

The module ends with two explicit export groups:

```ts
hasNonEmptyString
localeLowercasePreservingWhitespace
lowercasePreservingWhitespace
normalizeLowercaseStringOrEmpty
normalizeNullableString
normalizeOptionalLowercaseString
normalizeOptionalString
normalizeStringifiedOptionalString
readStringValue
```

```ts
CONFIG_DIR
clamp
clampInt
clampNumber
displayPath
displayString
ensureDir
escapeRegExp
isRecord
normalizeE164
pathExists
resolveConfigDir
resolveHomeDir
resolveUserPath
safeParseJson
shortenHomeInString
shortenHomePath
sleep
sliceUtf16Safe
truncateUtf16Safe
```

## zod

`openclaw/plugin-sdk/zod` is the simplest subpath in the SDK:

```ts
export * from "zod";
```

It exists so plugin packages can depend on the same import boundary style as the rest of the SDK while still using zod directly.

## Example

```ts
import { createCliRuntimeCapture } from "openclaw/plugin-sdk/testing";
import { normalizeOptionalString, truncateUtf16Safe } from "openclaw/plugin-sdk/text-runtime";
import { z } from "openclaw/plugin-sdk/zod";

const runtime = createCliRuntimeCapture();
const MessageSchema = z.object({
  text: z.string(),
});

const parsed = MessageSchema.parse({ text: "hello" });
runtime.info(parsed.text);

console.log(normalizeOptionalString("  hi  "));
console.log(truncateUtf16Safe("abcdef", 3));
```

Use `testing` for supported harness helpers, `text-runtime` for reusable text and logging utilities, and `zod` when you want package-boundary consistency in your extension imports.
