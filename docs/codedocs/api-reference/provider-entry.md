---
title: "provider-entry"
description: "API reference for the single-provider plugin helper and its onboarding-oriented option types."
---

Import path: `openclaw/plugin-sdk/provider-entry`

`src/plugin-sdk/provider-entry.ts` is a convenience layer on top of `definePluginEntry`. It is designed for provider plugins that mostly register one provider and want first-class API-key auth, env-var wiring, wizard choices, and a catalog builder without repeating boilerplate.

## Exported Types

```ts
type SingleProviderPluginApiKeyAuthOptions = Omit<
  ApiKeyAuthMethodOptions,
  "providerId" | "expectedProviders" | "wizard"
> & {
  expectedProviders?: string[];
  wizard?: false | ProviderPluginWizardSetup;
};
```

```ts
type SingleProviderPluginCatalogOptions =
  | {
      buildProvider: Parameters<typeof buildSingleProviderApiKeyCatalog>[0]["buildProvider"];
      allowExplicitBaseUrl?: boolean;
    }
  | {
      run: ProviderPluginCatalog["run"];
      order?: ProviderPluginCatalog["order"];
    };
```

```ts
type SingleProviderPluginOptions = {
  id: string;
  name: string;
  description: string;
  kind?: OpenClawPluginDefinition["kind"];
  configSchema?: OpenClawPluginConfigSchema | (() => OpenClawPluginConfigSchema);
  provider?: {
    id?: string;
    label: string;
    docsPath: string;
    aliases?: string[];
    envVars?: string[];
    auth?: SingleProviderPluginApiKeyAuthOptions[];
    catalog: SingleProviderPluginCatalogOptions;
  };
  register?: (api: OpenClawPluginApi) => void;
};
```

## Main Function

```ts
function defineSingleProviderPluginEntry(
  options: SingleProviderPluginOptions,
): ReturnType<typeof definePluginEntry>
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `options.id` | `string` | — | Plugin ID and default provider ID. |
| `options.name` | `string` | — | Human-readable plugin name. |
| `options.description` | `string` | — | Plugin description. |
| `options.provider.id` | `string` | `options.id` | Explicit provider ID when it should differ from the plugin ID. |
| `options.provider.label` | `string` | — | Provider label shown to users. |
| `options.provider.docsPath` | `string` | — | Docs path registered with the provider. |
| `options.provider.aliases` | `string[]` | — | Extra provider aliases. |
| `options.provider.envVars` | `string[]` | — | Additional auth-related env vars. |
| `options.provider.auth` | `SingleProviderPluginApiKeyAuthOptions[]` | `[]` | API-key auth methods and setup prompts. |
| `options.provider.catalog` | `SingleProviderPluginCatalogOptions` | — | Provider catalog strategy. |
| `options.register` | `(api: OpenClawPluginApi) => void` | — | Optional extra registrations after provider registration. |

Internally, the helper does three things worth knowing:

- It merges provider-level env vars with any env vars declared on auth methods.
- It auto-builds wizard choice metadata unless you disable or override it.
- It turns the `buildProvider` shortcut into a `catalog.run(...)` function that calls `buildSingleProviderApiKeyCatalog(...)`.

## Example

```ts
import { defineSingleProviderPluginEntry } from "openclaw/plugin-sdk/provider-entry";

export default defineSingleProviderPluginEntry({
  id: "acme",
  name: "Acme Provider",
  description: "Example provider plugin",
  provider: {
    label: "Acme",
    docsPath: "/providers/acme",
    auth: [
      {
        methodId: "api-key",
        label: "Acme API key",
        hint: "API key",
        envVar: "ACME_API_KEY",
        promptMessage: "Enter the Acme API key",
      },
    ],
    catalog: {
      buildProvider: ({ apiKey }) => ({
        id: "acme",
        label: "Acme",
        apiKey,
      }),
    },
  },
});
```

Use `catalog.run` instead of `buildProvider` when the provider must inspect config or resolve credentials dynamically, as the bundled Qwen plugin does in `extensions/qwen/index.ts`.
