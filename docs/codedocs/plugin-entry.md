---
title: "Plugin Entry"
description: "Learn how generic plugin entry modules become stable, lazily configured OpenClaw extensions."
---

`definePluginEntry` is the lowest-level authoring primitive in the public SDK. It exists for plugins that are not only a provider or only a channel: tools, services, memory adapters, browser helpers, setup APIs, and multi-capability bundles such as the OpenAI extension in `extensions/openai/index.ts` all build on this helper.

## What It Solves

Without a common entry helper, every plugin would have to recreate the same shape: `id`, `name`, `description`, optional config schema, and a `register(api)` function. `src/plugin-sdk/plugin-entry.ts` turns that into one canonical structure and makes `configSchema` lazy through `createCachedLazyValueGetter` from `src/plugin-sdk/lazy-value.ts`. That means startup paths such as metadata scans do not need to build heavy schemas up front.

## How It Relates To Other Concepts

- `provider-entry.ts` is a specialized wrapper that ultimately calls `definePluginEntry`.
- `core.ts` uses a similar pattern for channel plugins through `defineChannelPluginEntry`.
- `plugin-runtime.ts` exposes runtime-facing helpers that your registered plugin logic can use after activation.

## Internal Flow

```mermaid
flowchart TD
  A[definePluginEntry options] --> B[createCachedLazyValueGetter]
  B --> C[get configSchema getter]
  A --> D[register function]
  C --> E[plugin definition object]
  D --> E
  E --> F[OpenClaw imports plugin]
  F --> G[register(api)]
```

The implementation in `src/plugin-sdk/plugin-entry.ts` is deliberately small. It accepts a `configSchema` value or factory, wraps it in a cached getter, and returns a plain object with optional fields only when they exist. That pattern matters because some plugins have security audit collectors or node-host commands while others do not, and the SDK does not want authors to emit noisy `undefined` fields.

Basic usage:

```ts
import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry";

export default definePluginEntry({
  id: "hello-tools",
  name: "Hello Tools",
  description: "Registers a single debugging tool",
  register(api) {
    api.registerTool({
      name: "hello_world",
      description: "Return a fixed string",
      parameters: { type: "object", properties: {} },
      execute: async () => ({ ok: true, text: "hello" }),
    });
  },
});
```

Advanced usage with multiple capabilities:

```ts
import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry";
import { buildProviderToolCompatFamilyHooks } from "openclaw/plugin-sdk/provider-tools";

const openAIToolCompatHooks = buildProviderToolCompatFamilyHooks("openai");

export default definePluginEntry({
  id: "acme-suite",
  name: "Acme Suite",
  description: "Provider plus extra runtime helpers",
  register(api) {
    api.registerProvider({
      id: "acme",
      label: "Acme",
      docsPath: "/providers/acme",
      ...openAIToolCompatHooks,
      catalog: { order: "simple", run: async () => null },
      auth: [],
    });

    api.registerCliBackend({
      id: "acme-cli",
      label: "Acme CLI",
      resolve: async () => null,
    });
  },
});
```

<Callout type="warn">
Do not put side effects inside a `configSchema` factory. `definePluginEntry` caches the result, but the getter still runs on demand during metadata or registration flows. Keep schema factories pure and move runtime initialization into `register(api)`.
</Callout>

<Accordions>
  <Accordion title="Why use definePluginEntry instead of returning a plain object?">
    The helper buys you more than syntax reduction. It guarantees the returned object matches the shape OpenClaw expects, and it makes lazy config-schema evaluation the default instead of something every plugin has to remember to implement. That is especially important for extension packages that may be loaded in metadata-only modes where heavy schema work is wasted. It also keeps future compatibility changes in one public function instead of pushing them into every plugin repository.
  </Accordion>
  <Accordion title="When should you skip straight to provider-entry or core instead?">
    Use `provider-entry` when your plugin is primarily one provider with API-key auth and a catalog, because that helper generates a lot of repetitive provider boilerplate for you. Use `core` when your plugin is a channel, because channels need registration-mode handling, outbound routing, pairing, and security adapters that `definePluginEntry` intentionally does not know about. Reach for raw `definePluginEntry` when your extension registers multiple capability families or has no channel/provider identity at all. The OpenAI bundled plugin is a good model: it registers providers, CLI backends, image generation, speech, media understanding, and video generation from one entry.
  </Accordion>
</Accordions>

Next: [Provider Plugins](/docs/provider-plugins), [Channel Plugins](/docs/channel-plugins), and the [plugin-entry API page](/docs/api-reference/plugin-entry).
