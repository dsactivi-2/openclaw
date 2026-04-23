---
title: "plugin-entry"
description: "API reference for the generic plugin entry helper and its exported type families."
---

Import path: `openclaw/plugin-sdk/plugin-entry`

This module is the generic foundation for non-channel plugins. The implementation lives in `src/plugin-sdk/plugin-entry.ts` and returns a plain plugin definition object with lazy `configSchema` evaluation.

## Main Function

```ts
function definePluginEntry(options: {
  id: string;
  name: string;
  description: string;
  kind?: OpenClawPluginDefinition["kind"];
  configSchema?: OpenClawPluginConfigSchema | (() => OpenClawPluginConfigSchema);
  reload?: OpenClawPluginDefinition["reload"];
  nodeHostCommands?: OpenClawPluginDefinition["nodeHostCommands"];
  securityAuditCollectors?: OpenClawPluginDefinition["securityAuditCollectors"];
  register: (api: OpenClawPluginApi) => void;
}): {
  id: string;
  name: string;
  description: string;
  configSchema: OpenClawPluginConfigSchema;
  register: NonNullable<OpenClawPluginDefinition["register"]>;
}
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `id` | `string` | — | Stable plugin ID used in plugin registration and config keys. |
| `name` | `string` | — | Human-readable plugin name. |
| `description` | `string` | — | User-facing description for metadata and docs. |
| `kind` | `OpenClawPluginDefinition["kind"]` | — | Optional plugin kind classification. |
| `configSchema` | `OpenClawPluginConfigSchema \| (() => OpenClawPluginConfigSchema)` | `emptyPluginConfigSchema` | Schema or lazy schema factory. |
| `reload` | `OpenClawPluginDefinition["reload"]` | — | Reload triggers for runtime restarts or config changes. |
| `nodeHostCommands` | `OpenClawPluginDefinition["nodeHostCommands"]` | — | Optional node-host command definitions. |
| `securityAuditCollectors` | `OpenClawPluginDefinition["securityAuditCollectors"]` | — | Optional security audit collectors. |
| `register` | `(api: OpenClawPluginApi) => void` | — | Function that registers providers, tools, services, or channels. |

The return value is a normalized plugin definition object. `configSchema` is backed by the cached getter created in `src/plugin-sdk/lazy-value.ts`, so expensive schema construction runs at most once.

## Re-exported Types

This subpath is also the primary public home for plugin-facing type families used throughout the rest of the SDK:

- `OpenClawPluginApi`, `OpenClawPluginDefinition`, `OpenClawPluginConfigSchema`
- `OpenClawPluginToolContext`, `OpenClawPluginToolFactory`, `OpenClawPluginService`
- Provider contexts such as `ProviderCatalogContext`, `ProviderAuthContext`, `ProviderPrepareRuntimeAuthContext`, and `ProviderNormalizeToolSchemasContext`
- Capability interfaces such as `SpeechProviderPlugin`, `RealtimeTranscriptionProviderPlugin`, and `MediaUnderstandingProviderPlugin`

Those names are re-exported directly from `src/plugins/types.ts` and `src/plugins/provider-runtime-model.types.ts`, so this module is the canonical import path even when the types originate elsewhere.

## Example

```ts
import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry";

export default definePluginEntry({
  id: "debug-tools",
  name: "Debug Tools",
  description: "Example helper plugin",
  register(api) {
    api.registerTool({
      name: "echo_debug",
      description: "Echo a string for debugging",
      parameters: {
        type: "object",
        properties: {
          text: { type: "string" },
        },
        required: ["text"],
      },
      execute: async ({ text }) => ({ ok: true, text }),
    });
  },
});
```

Related modules: [provider-entry](/docs/api-reference/provider-entry), [runtime-and-config](/docs/api-reference/runtime-and-config), and [testing-and-text](/docs/api-reference/testing-and-text).
