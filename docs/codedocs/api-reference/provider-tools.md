---
title: "provider-tools"
description: "API reference for provider-side tool schema normalization and compatibility hooks."
---

Import path: `openclaw/plugin-sdk/provider-tools`

`src/plugin-sdk/provider-tools.ts` is where provider plugins normalize JSON Schema for model families that do not accept the full tool schema surface OpenClaw can generate. The bundled OpenAI extension uses `buildProviderToolCompatFamilyHooks("openai")`, and xAI-specific compatibility helpers also live here.

## Constants

```ts
const XAI_TOOL_SCHEMA_PROFILE = "xai";
const HTML_ENTITY_TOOL_CALL_ARGUMENTS_ENCODING = "html-entities";
const XAI_UNSUPPORTED_SCHEMA_KEYWORDS: Set<string>
```

The xAI constants encode the exact compatibility patch that `resolveXaiModelCompatPatch()` applies: a schema profile name, a list of unsupported keywords, and the fact that tool-call arguments may need HTML entity decoding.

## Main Functions

```ts
function stripUnsupportedSchemaKeywords(
  schema: unknown,
  unsupportedKeywords: ReadonlySet<string>,
): unknown
function stripXaiUnsupportedKeywords(schema: unknown): unknown
function resolveXaiModelCompatPatch(): ModelCompatConfig
function applyXaiModelCompat<T extends { compat?: unknown }>(model: T): T
```

```ts
function findUnsupportedSchemaKeywords(
  schema: unknown,
  path: string,
  unsupportedKeywords: ReadonlySet<string>,
): string[]
function normalizeGeminiToolSchemas(
  ctx: ProviderNormalizeToolSchemasContext,
): AnyAgentTool[]
function inspectGeminiToolSchemas(
  ctx: ProviderNormalizeToolSchemasContext,
): ProviderToolSchemaDiagnostic[]
```

```ts
function normalizeOpenAIToolSchemas(
  ctx: ProviderNormalizeToolSchemasContext,
): AnyAgentTool[]
function findOpenAIStrictSchemaViolations(
  schema: unknown,
  path: string,
  options?: { requireObjectRoot?: boolean },
): string[]
function inspectOpenAIToolSchemas(
  ctx: ProviderNormalizeToolSchemasContext,
): ProviderToolSchemaDiagnostic[]
```

```ts
type ProviderToolCompatFamily = "gemini" | "openai";

function buildProviderToolCompatFamilyHooks(family: ProviderToolCompatFamily): {
  normalizeToolSchemas: (ctx: ProviderNormalizeToolSchemasContext) => AnyAgentTool[];
  inspectToolSchemas: (ctx: ProviderNormalizeToolSchemasContext) => ProviderToolSchemaDiagnostic[];
}
```

## Example

```ts
import { buildProviderToolCompatFamilyHooks } from "openclaw/plugin-sdk/provider-tools";

const hooks = buildProviderToolCompatFamilyHooks("openai");

api.registerProvider({
  id: "acme-openai",
  label: "Acme OpenAI",
  docsPath: "/providers/acme-openai",
  auth: [],
  catalog: { order: "simple", run: async () => null },
  ...hooks,
});
```

Use `buildProviderToolCompatFamilyHooks("gemini")` when the provider needs the Gemini schema cleaner, and use the xAI helpers when the provider wants to patch model compatibility metadata directly. Because these helpers are pure transforms, they are safe to reuse in tests and setup code.
