---
title: "Build A Channel Plugin"
description: "Create a chat-style channel plugin with routing, setup, and outbound delivery."
---

This guide follows the same structure used by the bundled QA and Google Chat plugins: create the channel with `createChatChannelPlugin`, then export it through `defineChannelPluginEntry`.

<Steps>
  <Step>
  ### Define the core channel behavior

```ts
import {
  buildChannelOutboundSessionRoute,
  createChatChannelPlugin,
} from "openclaw/plugin-sdk/core";

export const demoChannel = createChatChannelPlugin({
  base: {
    id: "demo-channel",
    meta: { id: "demo-channel", label: "Demo Channel", docsPath: "/channels/demo-channel" },
    capabilities: { chatTypes: ["direct", "group"] },
    setup: {
      applyAccountConfig: ({ cfg, accountId, input }) => ({
        ...cfg,
        channels: {
          ...cfg.channels,
          demo: {
            ...(cfg.channels?.demo ?? {}),
            [accountId ?? "default"]: input,
          },
        },
      }),
    },
    messaging: {
      normalizeTarget: (raw) => raw.trim(),
      resolveOutboundSessionRoute: ({ cfg, agentId, accountId, target }) =>
        buildChannelOutboundSessionRoute({
          cfg,
          agentId,
          channel: "demo-channel",
          accountId,
          peer: { kind: "direct", id: target },
          chatType: "direct",
          from: `demo-channel:${accountId ?? "default"}`,
          to: target,
        }),
    },
  },
});
```

  </Step>
  <Step>
  ### Add outbound delivery and pairing

```ts
export const demoChannel = createChatChannelPlugin({
  base: { /* ...same as above... */ },
  pairing: {
    text: {
      idLabel: "demo user",
      message: "Demo Channel pairing approved.",
      notify: async ({ message, send }) => {
        await send(message);
      },
    },
  },
  outbound: {
    base: { deliveryMode: "direct" },
    attachedResults: {
      channel: "demo-channel",
      sendText: async ({ to, text }) => ({
        messageId: `${to}:${text.length}`,
      }),
    },
  },
});
```

  </Step>
  <Step>
  ### Export the channel entry

```ts
import { defineChannelPluginEntry } from "openclaw/plugin-sdk/core";

export default defineChannelPluginEntry({
  id: "demo-channel",
  name: "Demo Channel",
  description: "Example channel plugin",
  plugin: demoChannel,
});
```

  </Step>
</Steps>

If the channel also ships a setup-only module, export `defineSetupPluginEntry(demoChannel)` from `setup-entry.ts`. That is the pattern the SDK expects when a plugin wants separate setup metadata and full runtime activation.
