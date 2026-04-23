---
title: "provider-web-search"
description: "API reference for web-search provider contracts, credential wiring, caching helpers, and provider-owned tool definitions."
---

Import paths:

- `openclaw/plugin-sdk/provider-web-search`
- `openclaw/plugin-sdk/provider-web-search-contract`
- `openclaw/plugin-sdk/provider-web-search-config-contract`

These modules provide two layers of support. The `provider-web-search` subpath is the full runtime-oriented helper set, while the `*-contract` subpaths focus on stable credential wiring and config mutation behavior.

## Main Contract Function

From `src/plugin-sdk/provider-web-search-contract.ts`:

```ts
function createWebSearchProviderContractFields(
  options: CreateWebSearchProviderContractFieldsOptions & {
    selectionPluginId?: string;
  },
): Pick<
  WebSearchProviderPlugin,
  "inactiveSecretPaths" | "getCredentialValue" | "setCredentialValue"
> &
  Partial<
    Pick<
      WebSearchProviderPlugin,
      "applySelectionConfig" | "getConfiguredCredentialValue" | "setConfiguredCredentialValue"
    >
  >
```

The `provider-web-search-config-contract` variant exports the same base behavior without `applySelectionConfig`.

Important types from `src/plugin-sdk/provider-web-search-contract-fields.ts`:

```ts
type WebSearchProviderContractCredential =
  | { type: "none" }
  | { type: "top-level" }
  | { type: "scoped"; scopeId: string };

type WebSearchProviderConfiguredCredential = {
  pluginId: string;
  field?: string;
};

type CreateWebSearchProviderContractFieldsOptions = {
  credentialPath: string;
  inactiveSecretPaths?: string[];
  searchCredential: WebSearchProviderContractCredential;
  configuredCredential?: WebSearchProviderConfiguredCredential;
};
```

## Runtime Helper Surface

`provider-web-search.ts` re-exports the pieces most providers need once they own `createTool(...)` themselves:

- Request helpers: `resolveSearchCount`, `resolveSearchTimeoutSeconds`, `normalizeFreshness`, `parseIsoDateRange`
- Cache helpers: `buildSearchCacheKey`, `readCachedSearchPayload`, `writeCachedSearchPayload`
- Trusted endpoint helpers: `postTrustedWebToolsJson`, `withTrustedWebSearchEndpoint`, `withTrustedWebToolsEndpoint`
- Credential/config helpers: `resolveProviderWebSearchPluginConfig`, `getScopedCredentialValue`, `setTopLevelCredentialValue`

Deprecated helper:

```ts
function createPluginBackedWebSearchProvider(
  provider: WebSearchProviderPlugin,
): WebSearchProviderPlugin
```

This function now throws from `createTool()`. The source comment in `src/plugin-sdk/provider-web-search.ts` is explicit: implement provider-owned `createTool(...)` directly.

## Example

```ts
import { createWebSearchProviderContractFields } from "openclaw/plugin-sdk/provider-web-search-contract";
import type { WebSearchProviderPlugin } from "openclaw/plugin-sdk/provider-web-search";

const contract = createWebSearchProviderContractFields({
  credentialPath: "plugins.entries.acme.config.webSearch.apiKey",
  searchCredential: { type: "top-level" },
  configuredCredential: { pluginId: "acme" },
  selectionPluginId: "acme",
});

export function createAcmeSearchProvider(): WebSearchProviderPlugin {
  return {
    id: "acme-search",
    label: "Acme Search",
    ...contract,
    createTool: () => ({
      description: "Search Acme",
      parameters: { type: "object", properties: { query: { type: "string" } }, required: ["query"] },
      execute: async ({ query }) => ({ ok: true, query, results: [] }),
    }),
  };
}
```

Use the contract subpaths for stable credential semantics, and only pull in the broader runtime helpers when the provider also needs caching, freshness parsing, or trusted endpoint posting.
