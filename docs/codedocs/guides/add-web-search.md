---
title: "Add Web Search"
description: "Expose a web-search provider with stable credential wiring and a provider-owned tool implementation."
---

The recommended pattern for web-search plugins is to keep credential wiring in the contract-safe helper and keep the actual search tool in the provider implementation.

<Steps>
  <Step>
  ### Define the provider contract fields

```ts
import { createWebSearchProviderContractFields } from "openclaw/plugin-sdk/provider-web-search-contract";

const SEARCH_KEY_PATH = "plugins.entries.acme.config.webSearch.apiKey";

const contractFields = createWebSearchProviderContractFields({
  credentialPath: SEARCH_KEY_PATH,
  searchCredential: { type: "top-level" },
  configuredCredential: { pluginId: "acme" },
  selectionPluginId: "acme",
});
```

  </Step>
  <Step>
  ### Implement the tool directly on the provider

```ts
import type {
  WebSearchProviderPlugin,
  WebSearchProviderToolDefinition,
} from "openclaw/plugin-sdk/provider-web-search";

function createAcmeSearchTool(): WebSearchProviderToolDefinition {
  return {
    description: "Search Acme's indexed web corpus.",
    parameters: {
      type: "object",
      properties: {
        query: { type: "string" },
        count: { type: "number", minimum: 1, maximum: 10 },
      },
      required: ["query"],
    },
    execute: async ({ query, count = 5 }) => {
      return { ok: true, query, count, results: [] };
    },
  };
}

export function createAcmeWebSearchProvider(): WebSearchProviderPlugin {
  return {
    id: "acme-search",
    label: "Acme Search",
    credentialLabel: "Acme Search API key",
    envVars: ["ACME_SEARCH_API_KEY"],
    ...contractFields,
    createTool: () => createAcmeSearchTool(),
  };
}
```

  </Step>
  <Step>
  ### Add caching or timeout helpers only when you need them

```ts
import {
  buildSearchCacheKey,
  readCachedSearchPayload,
  writeCachedSearchPayload,
} from "openclaw/plugin-sdk/provider-web-search";

async function cachedExecute(query: string) {
  const cacheKey = buildSearchCacheKey({ provider: "acme-search", query });
  const cached = await readCachedSearchPayload(cacheKey);
  if (cached) {
    return cached;
  }

  const response = { ok: true, query, results: [] };
  await writeCachedSearchPayload(cacheKey, response);
  return response;
}
```

  </Step>
</Steps>

This matches the direction in `src/plugin-sdk/provider-web-search.ts`: `createPluginBackedWebSearchProvider` is deprecated, and the provider should own `createTool(...)` directly.
