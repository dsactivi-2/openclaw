---
title: "Types"
description: "Reference the key exported TypeScript types used by the OpenClaw plugin SDK."
---

The SDK exports a very large type surface, but most plugin authors touch the same small set repeatedly. This page collects the most important concrete definitions from the source plus the type-only bundles you can safely import from each subpath.

## Entry And Provider Types

From `src/plugin-sdk/provider-entry.ts`:

```ts
export type SingleProviderPluginOptions = {
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

This is the main authoring type for single-provider plugins. It is intentionally declarative: auth methods, env vars, aliases, and catalog behavior all live in one object.

From `src/plugin-sdk/plugin-entry.ts`, the most important type families are re-exported rather than defined locally:

- `OpenClawPluginApi`
- `OpenClawPluginDefinition`
- `OpenClawPluginConfigSchema`
- `ProviderCatalogContext`
- `ProviderAuthContext`
- `ProviderPrepareRuntimeAuthContext`
- `ProviderNormalizeToolSchemasContext`

Use `openclaw/plugin-sdk/plugin-entry` as the canonical import path for those types even when the underlying definition lives in `src/plugins/types.ts`.

## Onboarding And Model Types

From `src/plugin-sdk/provider-onboard.ts`:

```ts
export type AgentModelAliasEntry =
  | string
  | {
      modelRef: string;
      alias?: string;
    };
```

```ts
export type ProviderOnboardPresetAppliers<TArgs extends unknown[]> = {
  applyProviderConfig: (cfg: OpenClawConfig, ...args: TArgs) => OpenClawConfig;
  applyConfig: (cfg: OpenClawConfig, ...args: TArgs) => OpenClawConfig;
};
```

These types matter in onboarding flows because they describe how a setup wizard mutates provider config and model aliases without owning the whole config tree manually.

`openclaw/plugin-sdk/provider-model-types` is the narrow type-only import path for:

```ts
type ModelApi
type ModelCompatConfig
type ModelDefinitionConfig
type ModelProviderConfig
type BedrockDiscoveryConfig
```

## Search And Secret Types

From `src/plugin-sdk/provider-web-search-contract-fields.ts`:

```ts
export type WebSearchProviderContractCredential =
  | { type: "none" }
  | { type: "top-level" }
  | { type: "scoped"; scopeId: string };

export type WebSearchProviderConfiguredCredential = {
  pluginId: string;
  field?: string;
};

export type CreateWebSearchProviderContractFieldsOptions = {
  credentialPath: string;
  inactiveSecretPaths?: string[];
  searchCredential: WebSearchProviderContractCredential;
  configuredCredential?: WebSearchProviderConfiguredCredential;
};
```

From `src/plugin-sdk/secret-input-schema.ts`, the schema supports either a plain string or discriminated object forms with `source: "env"`, `source: "file"`, or `source: "exec"`. The exported `SecretInput`, `SecretInputStringResolution`, and `SecretInputStringResolutionMode` types from `openclaw/plugin-sdk/secret-input` and `openclaw/plugin-sdk/secret-ref-runtime` are the runtime-facing counterparts.

## Runtime Auth Types

From `src/plugin-sdk/provider-auth-runtime.ts`:

```ts
export type PreparedCodexAuthBridge = {
  codexHome: string;
  clearEnv: string[];
};

export type OAuthCallbackResult = { code: string; state: string };
```

These are the two concrete types to know if your provider bridges OAuth credentials into a local runtime or implements a local callback flow.

## Video Generation Types

From `src/plugin-sdk/video-generation.ts`:

```ts
export type VideoGenerationRequest = {
  provider: string;
  model: string;
  prompt: string;
  cfg: OpenClawConfig;
  agentDir?: string;
  authStore?: AuthProfileStore;
  timeoutMs?: number;
  size?: string;
  aspectRatio?: string;
  resolution?: VideoGenerationResolution;
  durationSeconds?: number;
  audio?: boolean;
  watermark?: boolean;
  inputImages?: VideoGenerationSourceAsset[];
  inputVideos?: VideoGenerationSourceAsset[];
  inputAudios?: VideoGenerationSourceAsset[];
  providerOptions?: Record<string, unknown>;
};
```

```ts
export type VideoGenerationProvider = {
  id: string;
  aliases?: string[];
  label?: string;
  defaultModel?: string;
  models?: string[];
  capabilities: VideoGenerationProviderCapabilities;
  isConfigured?: (ctx: VideoGenerationProviderConfiguredContext) => boolean;
  generateVideo: (req: VideoGenerationRequest) => Promise<VideoGenerationResult>;
};
```

This contract is the clearest example of the SDK keeping public capability types local to the boundary file so package consumers get stable emitted declarations.

## Type-Only Bundles By Import Path

| Import path | Primary type families |
|-------------|-----------------------|
| `openclaw/plugin-sdk/plugin-entry` | Plugin API, provider contexts, tool and service contracts |
| `openclaw/plugin-sdk/core` | Channel plugin types, channel metadata, outbound session route types |
| `openclaw/plugin-sdk/provider-model-types` | `ModelApi`, `ModelProviderConfig`, `ModelDefinitionConfig`, `ModelCompatConfig` |
| `openclaw/plugin-sdk/channel-streaming` | Channel streaming config and preview/block mode types |
| `openclaw/plugin-sdk/provider-web-search` | Search config and web-search provider contracts |
| `openclaw/plugin-sdk/testing` | Test harness capture and mock types |
| `openclaw/plugin-sdk/zod` | Full `zod` export surface |

For the full operational details behind these types, use the related API pages and concept pages. The import paths on this page are the stable ones to use in extension packages.
