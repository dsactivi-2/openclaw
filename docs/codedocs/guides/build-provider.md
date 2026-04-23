---
title: "Build A Provider Plugin"
description: "Create a provider plugin with API-key auth, a model catalog, and optional extra capabilities."
---

This guide shows the default pattern for a provider plugin: use `defineSingleProviderPluginEntry` when one plugin mostly maps to one provider, then add extra registrations only when you need them.

<Steps>
  <Step>
  ### Create the plugin entry

```ts
import { defineSingleProviderPluginEntry } from "openclaw/plugin-sdk/provider-entry";

const PROVIDER_ID = "acme";

export default defineSingleProviderPluginEntry({
  id: PROVIDER_ID,
  name: "Acme Provider",
  description: "Bundled Acme provider plugin",
  provider: {
    label: "Acme",
    docsPath: "/providers/acme",
    auth: [
      {
        methodId: "api-key",
        label: "Acme API key",
        hint: "API key",
        optionKey: "acmeApiKey",
        flagName: "--acme-api-key",
        envVar: "ACME_API_KEY",
        promptMessage: "Enter Acme API key",
      },
    ],
    catalog: {
      buildProvider: ({ apiKey }) => ({
        id: PROVIDER_ID,
        label: "Acme",
        apiKey,
        baseUrl: "https://api.acme.example/v1",
      }),
    },
  },
});
```

  </Step>
  <Step>
  ### Add onboarding defaults when the provider needs config mutations

```ts
import { applyProviderConfigWithDefaultModelPreset } from "openclaw/plugin-sdk/provider-onboard";

export function applyAcmeConfig(cfg: OpenClawConfig) {
  return applyProviderConfigWithDefaultModelPreset(cfg, {
    providerId: "acme",
    api: "openai-chat-completions",
    baseUrl: "https://api.acme.example/v1",
    defaultModel: { id: "acme/acme-chat-1", label: "Acme Chat 1" },
    primaryModelRef: "acme/acme-chat-1",
  });
}
```

  </Step>
  <Step>
  ### Register extra capabilities only when needed

```ts
import { defineSingleProviderPluginEntry } from "openclaw/plugin-sdk/provider-entry";
import { buildProviderToolCompatFamilyHooks } from "openclaw/plugin-sdk/provider-tools";

const toolCompat = buildProviderToolCompatFamilyHooks("openai");

export default defineSingleProviderPluginEntry({
  id: "acme",
  name: "Acme Provider",
  description: "Acme provider with image support",
  provider: {
    label: "Acme",
    docsPath: "/providers/acme",
    auth: [],
    catalog: { buildProvider: ({ apiKey }) => ({ id: "acme", apiKey, ...toolCompat }) },
  },
  register(api) {
    api.registerImageGenerationProvider({
      id: "acme-images",
      async generateImage() {
        return { images: [] };
      },
    });
  },
});
```

  </Step>
</Steps>

This pattern mirrors bundled extensions like `extensions/deepseek/index.ts` and `extensions/qwen/index.ts`: keep the entry file declarative, delegate provider-specific logic to catalog or onboarding helpers, and only drop down to raw `definePluginEntry` when one plugin needs to register several unrelated capability families.
