---
title: "runtime-and-config"
description: "API reference for runtime bundles such as config-runtime, plugin-runtime, runtime-env, runtime-doctor, and cli-runtime."
---

Import paths covered on this page:

- `openclaw/plugin-sdk/plugin-runtime`
- `openclaw/plugin-sdk/config-runtime`
- `openclaw/plugin-sdk/runtime-env`
- `openclaw/plugin-sdk/runtime-doctor`
- `openclaw/plugin-sdk/cli-runtime`

These modules are mostly curated re-export bundles. Their job is to provide a stable import path for common runtime behavior without asking plugin authors to know OpenClaw's private folder layout.

## plugin-runtime

`src/plugin-sdk/plugin-runtime.ts` re-exports command, hook, HTTP, interactive, and lazy-service helpers from the runtime. It also re-exports the key types:

```ts
export type { PluginRuntime, RuntimeLogger } from "../plugins/runtime/types.js";
```

Use this subpath when a plugin needs request-scoped runtime types or helper surfaces for commands, HTTP path registration, interactive bindings, or lazy service modules.

## config-runtime

`src/plugin-sdk/config-runtime.ts` is the main supported config boundary. High-value exports include:

- `loadConfig`, `writeConfigFile`, `getRuntimeConfigSnapshot`, `setRuntimeConfigSnapshot`
- `resolveDefaultAgentId`, `resolveChannelModelOverride`, `resolveMarkdownTableMode`
- `resolveChannelGroupPolicy`, `resolveDefaultGroupPolicy`, `resolveOpenProviderRuntimeGroupPolicy`
- `loadSessionStore`, `saveSessionStore`, `resolveSessionStoreEntry`, `resolveSessionKey`
- `resolveConfiguredSecretInputString`, `resolveRequiredConfiguredSecretRefInputString`

This module also re-exports many config types such as `OpenClawConfig`, `DmPolicy`, `GroupPolicy`, `MarkdownTableMode`, and channel-specific config types.

## runtime-env and cli-runtime

`runtime-env` exposes process helpers:

```ts
createNonExitingRuntime
defaultRuntime
retryAsync
computeBackoff
sleepWithAbort
waitForAbortSignal
ensureGlobalUndiciEnvProxyDispatcher
registerUnhandledRejectionHandler
```

`cli-runtime` is smaller and focused on command formatting, duration parsing, waiting helpers, prompt styling, and version exports. It is the right import path for setup commands or browser-related CLI tooling that needs formatting without the full command graph.

## runtime-doctor

This module covers config compatibility and installation checks. Common exports include `normalizeLegacyChannelAliases`, `normalizeLegacyStreamingAliases`, `detectPluginInstallPathIssue`, `formatPluginInstallPathIssue`, and `removePluginFromConfig`.

## Example

```ts
import {
  loadConfig,
  resolveSessionKey,
  writeConfigFile,
} from "openclaw/plugin-sdk/config-runtime";
import { retryAsync } from "openclaw/plugin-sdk/runtime-env";

const cfg = await loadConfig();
const sessionKey = resolveSessionKey({
  channel: "demo",
  agentId: "main",
  to: "user:123",
});

await retryAsync(async () => {
  cfg.plugins ??= {};
  await writeConfigFile(cfg);
});
```

Reach for these bundles when you want stable runtime helpers, not when you need provider- or channel-specific contracts. Those are intentionally split into other subpaths.
