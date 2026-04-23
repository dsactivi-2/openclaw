---
title: "browser-and-acp"
description: "API reference for browser-safe runtime helpers, ACP runtime integration, and error/account utility subpaths."
---

Import paths covered on this page:

- `openclaw/plugin-sdk/browser-config-runtime`
- `openclaw/plugin-sdk/browser-node-runtime`
- `openclaw/plugin-sdk/browser-setup-tools`
- `openclaw/plugin-sdk/browser-security-runtime`
- `openclaw/plugin-sdk/acp-runtime`
- `openclaw/plugin-sdk/error-runtime`
- `openclaw/plugin-sdk/account-id`

These modules are the public surface for plugins that interact with browser or node-host tooling, ACP session management, and a few small utility subpaths.

## Browser-oriented modules

`browser-config-runtime` focuses on config and setup concerns such as `resolveConfigPath`, `resolveGatewayPort`, `normalizePluginsConfig`, and `resolveEffectiveEnableState`.

`browser-node-runtime` exposes gateway- and node-facing helpers:

```ts
addGatewayClientOptions
callGatewayFromCli
runCommandWithRuntime
resolveGatewayAuth
isNodeCommandAllowed
resolveNodeCommandAllowlist
ensureGatewayStartupAuth
startLazyPluginServiceModule
runExec
withTimeout
```

`browser-setup-tools` is a toolkit for setup surfaces. It re-exports parameter readers, node list helpers, image helpers, media storage, CLI formatting, and a small test harness surface such as `createTempHomeEnv`.

`browser-security-runtime` covers filesystem and network safety with exports like `openFileWithinRoot`, `writeFileFromPathWithinRoot`, `isBlockedHostnameOrIp`, `resolvePinnedHostnameWithPolicy`, `ensurePortAvailable`, `wrapExternalContent`, and `safeEqualSecret`.

## acp-runtime

`src/plugin-sdk/acp-runtime.ts` is the public boundary for ACP integration:

```ts
getAcpSessionManager
getAcpRuntimeBackend
registerAcpRuntimeBackend
requireAcpRuntimeBackend
unregisterAcpRuntimeBackend
readAcpSessionEntry
```

Custom function:

```ts
async function tryDispatchAcpReplyHook(
  event: PluginHookReplyDispatchEvent,
  ctx: PluginHookReplyDispatchContext,
): Promise<PluginHookReplyDispatchResult | void>
```

The implementation is careful about `sendPolicy === "deny"` cases so ACP-bound sessions still keep internal state consistent without leaking user-facing delivery when delivery should be suppressed.

## error-runtime and account-id

From `src/plugin-sdk/error-runtime.ts`:

```ts
const SUBAGENT_RUNTIME_REQUEST_SCOPE_ERROR_CODE =
  "OPENCLAW_SUBAGENT_RUNTIME_REQUEST_SCOPE";

class RequestScopedSubagentRuntimeError extends Error {
  code = SUBAGENT_RUNTIME_REQUEST_SCOPE_ERROR_CODE;
}
```

That module also re-exports `collectErrorGraphCandidates`, `extractErrorCode`, `formatErrorMessage`, and `isApprovalNotFoundError`.

`account-id` is a small utility bundle:

```ts
DEFAULT_ACCOUNT_ID
normalizeAccountId
normalizeOptionalAccountId
```

## Example

```ts
import { resolveGatewayAuth } from "openclaw/plugin-sdk/browser-node-runtime";
import { generateSecureToken } from "openclaw/plugin-sdk/browser-security-runtime";
import { DEFAULT_ACCOUNT_ID } from "openclaw/plugin-sdk/account-id";

const auth = await resolveGatewayAuth({ host: "127.0.0.1", port: 18789 });
const token = generateSecureToken();

console.log(auth != null, token.length > 0, DEFAULT_ACCOUNT_ID);
```

Use these subpaths when your plugin interacts with browser tooling, local gateway calls, ACP, or request-scoped subagent surfaces. They keep those high-risk operations inside explicit, supportable boundaries.
