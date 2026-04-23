---
title: "provider-models"
description: "API reference for provider model helpers, onboarding config mutation helpers, HTTP helpers, usage fetchers, and env-var utilities."
---

Import paths covered on this page:

- `openclaw/plugin-sdk/provider-model-shared`
- `openclaw/plugin-sdk/provider-model-types`
- `openclaw/plugin-sdk/provider-onboard`
- `openclaw/plugin-sdk/provider-http`
- `openclaw/plugin-sdk/provider-usage`
- `openclaw/plugin-sdk/provider-env-vars`

These modules work together around provider configuration and capability shaping. `provider-model-shared` is where replay-policy and model-compat helpers live. `provider-onboard` mutates `OpenClawConfig` safely for setup flows. `provider-http`, `provider-usage`, and `provider-env-vars` handle the transport and account edges around those providers.

## provider-model-shared

Key functions from `src/plugin-sdk/provider-model-shared.ts`:

```ts
function getModelProviderHint(modelId: string): string | null
function isProxyReasoningUnsupportedModelHint(modelId: string): boolean
```

```ts
type ProviderReplayFamily =
  | "openai-compatible"
  | "anthropic-by-model"
  | "native-anthropic-by-model"
  | "google-gemini"
  | "passthrough-gemini"
  | "hybrid-anthropic-openai";

function buildProviderReplayFamilyHooks(
  options: BuildProviderReplayFamilyHooksOptions,
): Pick<ProviderPlugin, "buildReplayPolicy" | "sanitizeReplayHistory" | "resolveReasoningOutputMode">
```

It also exports ready-made constants such as `OPENAI_COMPATIBLE_REPLAY_HOOKS`, `ANTHROPIC_BY_MODEL_REPLAY_HOOKS`, and `PASSTHROUGH_GEMINI_REPLAY_HOOKS`.

## provider-onboard

Important types and functions from `src/plugin-sdk/provider-onboard.ts`:

```ts
type AgentModelAliasEntry =
  | string
  | {
      modelRef: string;
      alias?: string;
    };

type ProviderOnboardPresetAppliers<TArgs extends unknown[]> = {
  applyProviderConfig: (cfg: OpenClawConfig, ...args: TArgs) => OpenClawConfig;
  applyConfig: (cfg: OpenClawConfig, ...args: TArgs) => OpenClawConfig;
};
```

```ts
function withAgentModelAliases(...)
function applyOnboardAuthAgentModelsAndProviders(...)
function applyAgentDefaultModelPrimary(...)
function applyProviderConfigWithDefaultModelPreset(...)
function applyProviderConfigWithDefaultModelsPreset(...)
function applyProviderConfigWithModelCatalogPreset(...)
function createDefaultModelPresetAppliers(...)
function createDefaultModelsPresetAppliers(...)
function createModelCatalogPresetAppliers(...)
function ensureModelAllowlistEntry(params: {
  cfg: OpenClawConfig;
  modelRef: string;
  defaultProvider?: string;
}): OpenClawConfig
```

These helpers are designed for setup and onboarding code, not runtime provider execution. They mutate config predictably while preserving existing provider sections where possible.

## provider-http, provider-usage, and provider-env-vars

`provider-http` exports transport helpers such as `fetchWithTimeout`, `postJsonRequest`, `pollProviderOperationJson`, `resolveProviderEndpoint`, and `resolveProviderRequestPolicy`. `provider-usage` re-exports usage fetchers like `fetchClaudeUsage`, `fetchCodexUsage`, and `buildUsageHttpErrorSnapshot`. `provider-env-vars` is a smaller helper bundle for `getProviderEnvVars`, `listKnownProviderAuthEnvVarNames`, and `omitEnvKeysCaseInsensitive`.

## Example

```ts
import {
  OPENAI_COMPATIBLE_REPLAY_HOOKS,
  getModelProviderHint,
} from "openclaw/plugin-sdk/provider-model-shared";
import { applyProviderConfigWithDefaultModelPreset } from "openclaw/plugin-sdk/provider-onboard";

const hintedProvider = getModelProviderHint("acme/acme-chat-1");

const nextConfig = applyProviderConfigWithDefaultModelPreset(cfg, {
  providerId: hintedProvider ?? "acme",
  api: "openai-chat-completions",
  baseUrl: "https://api.acme.example/v1",
  defaultModel: { id: "acme/acme-chat-1", label: "Acme Chat 1" },
  primaryModelRef: "acme/acme-chat-1",
});

api.registerProvider({
  id: "acme",
  label: "Acme",
  docsPath: "/providers/acme",
  auth: [],
  catalog: { order: "simple", run: async () => null },
  ...OPENAI_COMPATIBLE_REPLAY_HOOKS,
});
```

If a provider needs only types, `openclaw/plugin-sdk/provider-model-types` is the narrowest import path. It re-exports `ModelApi`, `ModelCompatConfig`, `ModelDefinitionConfig`, `ModelProviderConfig`, and `BedrockDiscoveryConfig` without the extra helpers.
