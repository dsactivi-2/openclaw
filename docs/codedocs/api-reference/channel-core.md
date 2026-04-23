---
title: "channel-core"
description: "API reference for the channel helpers exported from openclaw/plugin-sdk/core."
---

Import path: `openclaw/plugin-sdk/core`

This module is the public center of gravity for channel authors. It re-exports many supporting types, but the most important functions are the entry helpers and the route-building utilities implemented in `src/plugin-sdk/core.ts`.

## Channel Registration Helpers

```ts
function defineChannelPluginEntry<TPlugin>(options: {
  id: string;
  name: string;
  description: string;
  plugin: TPlugin;
  configSchema?: ChannelEntryConfigSchema<TPlugin> | (() => ChannelEntryConfigSchema<TPlugin>);
  setRuntime?: (runtime: PluginRuntime) => void;
  registerCliMetadata?: (api: OpenClawPluginApi) => void;
  registerFull?: (api: OpenClawPluginApi) => void;
}): DefinedChannelPluginEntry<TPlugin>
```

```ts
function defineSetupPluginEntry<TPlugin>(plugin: TPlugin): { plugin: TPlugin }
```

`defineChannelPluginEntry` handles registration-mode differences. If OpenClaw is only collecting CLI metadata, it can skip full runtime registration. If the registration mode is `full`, it registers the channel and then runs any optional extra hooks.

## Channel Composition Helpers

```ts
function createChatChannelPlugin<
  TResolvedAccount extends { accountId?: string | null },
  Probe = unknown,
  Audit = unknown,
>(params: {
  base: ChatChannelPluginBase<TResolvedAccount, Probe, Audit>;
  security?: ChannelSecurityAdapter<TResolvedAccount> | ChatChannelSecurityOptions<TResolvedAccount>;
  pairing?: ChannelPairingAdapter | ChatChannelPairingOptions;
  threading?: ChannelThreadingAdapter | ChatChannelThreadingOptions<TResolvedAccount>;
  outbound?: ChannelOutboundAdapter | ChatChannelAttachedOutboundOptions;
}): ChannelPlugin<TResolvedAccount, Probe, Audit>
```

```ts
function createChannelPluginBase<TResolvedAccount>(
  params: CreateChannelPluginBaseOptions<TResolvedAccount>,
): CreatedChannelPluginBase<TResolvedAccount>
```

`createChatChannelPlugin` is the higher-level helper. It normalizes chat-channel concerns such as DM security, pairing approval text, threading reply modes, and attached outbound results. `createChannelPluginBase` is lower level and mostly fills optional fields while merging channel metadata from `getChatChannelMeta(...)`.

## Routing And Target Helpers

```ts
function buildChannelOutboundSessionRoute(params: {
  cfg: OpenClawConfig;
  agentId: string;
  channel: string;
  accountId?: string | null;
  peer: { kind: "direct" | "group" | "channel"; id: string };
  chatType: "direct" | "group" | "channel";
  from: string;
  to: string;
  threadId?: string | number;
}): ChannelOutboundSessionRoute
```

```ts
function getChatChannelMeta(id: ChatChannelId): ChannelMeta
function stripChannelTargetPrefix(raw: string, ...providers: string[]): string
function stripTargetKindPrefix(raw: string): string
async function ensureConfiguredAcpBindingReady(params: {
  cfg: OpenClawConfig;
  configuredBinding: ResolvedConfiguredAcpBinding | null;
}): Promise<{ ok: true } | { ok: false; error: string }>
```

| Function | Purpose |
|----------|---------|
| `buildChannelOutboundSessionRoute` | Create the canonical routing payload returned by `resolveOutboundSessionRoute`. |
| `getChatChannelMeta` | Resolve built-in channel metadata by channel ID. |
| `stripChannelTargetPrefix` | Remove provider prefixes like `slack:` or `telegram:` from user input. |
| `stripTargetKindPrefix` | Remove generic target prefixes like `user:` or `group:`. |
| `ensureConfiguredAcpBindingReady` | Validate that a configured ACP binding is ready before runtime use. |

## Example

```ts
import {
  buildChannelOutboundSessionRoute,
  createChatChannelPlugin,
  defineChannelPluginEntry,
} from "openclaw/plugin-sdk/core";

const plugin = createChatChannelPlugin({
  base: {
    id: "qa-like",
    meta: { id: "qa-like", label: "QA Like", docsPath: "/channels/qa-like" },
    setup: { applyAccountConfig: ({ cfg }) => cfg },
    messaging: {
      normalizeTarget: (raw) => raw.trim(),
      resolveOutboundSessionRoute: ({ cfg, agentId, target }) =>
        buildChannelOutboundSessionRoute({
          cfg,
          agentId,
          channel: "qa-like",
          peer: { kind: "direct", id: target },
          chatType: "direct",
          from: "qa-like:default",
          to: target,
        }),
    },
  },
});

export default defineChannelPluginEntry({
  id: "qa-like",
  name: "QA Like",
  description: "Example channel plugin",
  plugin,
});
```

Related pages: [Channel Plugins](/docs/channel-plugins) and [security-and-streaming](/docs/api-reference/security-and-streaming).
