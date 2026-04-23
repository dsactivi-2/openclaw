---
title: "Getting Started"
description: "Build OpenClaw plugins with the public SDK for providers, channels, runtime helpers, and setup flows."
---

OpenClaw's plugin SDK gives extension authors a stable way to register providers, channels, setup flows, auth, and runtime helpers without importing private internals.

## The Problem

- OpenClaw extensions need to register many different capabilities, but raw runtime internals are large and easy to couple to by accident.
- Provider plugins usually repeat the same boilerplate for API-key auth, model catalog wiring, onboarding defaults, and runtime auth resolution.
- Channel plugins need consistent behavior for pairing, DM policy, reply threading, outbound routing, and status reporting across many messaging systems.
- Plugin authors still need low-level helpers for config I/O, secrets, SSRF-safe network access, and streaming compatibility without crossing unstable boundaries.

## The Solution

The SDK under `openclaw/plugin-sdk/*` wraps those patterns into focused entrypoints. `definePluginEntry` and `defineSingleProviderPluginEntry` create stable plugin definitions, `createChatChannelPlugin` composes common chat-channel behavior, and runtime boundary modules expose config, auth, search, streaming, and browser-safe helpers with explicit public contracts.

```ts
import { defineSingleProviderPluginEntry } from "openclaw/plugin-sdk/provider-entry";

export default defineSingleProviderPluginEntry({
  id: "acme",
  name: "Acme Provider",
  description: "Example OpenClaw provider plugin",
  provider: {
    label: "Acme",
    docsPath: "/providers/acme",
    auth: [
      {
        methodId: "api-key",
        label: "Acme API key",
        hint: "API key",
        envVar: "ACME_API_KEY",
        promptMessage: "Enter your Acme API key",
      },
    ],
    catalog: {
      buildProvider: ({ apiKey }) => ({
        id: "acme",
        apiKey,
        label: "Acme",
      }),
    },
  },
});
```

## Installation

" "bun"]}>
  <Tab value="npm">

```bash
npm install openclaw
```

  </Tab>
  <Tab value="pnpm">

```bash
pnpm add openclaw
```

  </Tab>
  <Tab value="yarn">

```bash
yarn add openclaw
```

  </Tab>
  <Tab value="bun">

```bash
bun add openclaw
```

  </Tab>
</Tabs>

## Quick Start

The smallest useful SDK example is a plugin definition module. The object returned by `defineSingleProviderPluginEntry` is what OpenClaw loads at runtime.

```ts
import { defineSingleProviderPluginEntry } from "openclaw/plugin-sdk/provider-entry";

const plugin = defineSingleProviderPluginEntry({
  id: "hello-provider",
  name: "Hello Provider",
  description: "Minimal provider example",
  provider: {
    label: "Hello Provider",
    docsPath: "/providers/hello-provider",
    auth: [
      {
        methodId: "api-key",
        label: "Hello API key",
        hint: "API key",
        envVar: "HELLO_API_KEY",
        promptMessage: "Enter the Hello API key",
      },
    ],
    catalog: {
      buildProvider: ({ apiKey }) => ({
        id: "hello-provider",
        label: "Hello Provider",
        apiKey,
      }),
    },
  },
});

console.log(plugin.id);
console.log(plugin.name);
```

Expected output:

```text
hello-provider
Hello Provider
```

## Key Features

- Stable entry helpers for generic plugins, single-provider plugins, and channel plugins.
- Contract-safe runtime boundaries for config, auth, search, browser tooling, secrets, and SSRF-aware networking.
- Reusable channel composition for DM policy, pairing, threading, and outbound routing.
- Provider-specific helpers for onboarding defaults, model replay policies, tool-schema compatibility, and usage/auth flows.
- Narrow testing helpers so extension packages can validate registration and gateway behavior without private imports.

<Cards>
  <Card title="Architecture" href="/docs/architecture">See how plugin entrypoints connect to OpenClaw's registries and runtime boundaries.</Card>
  <Card title="Core Concepts" href="/docs/plugin-entry">Start with plugin entries, provider composition, channels, and boundary modules.</Card>
  <Card title="API Reference" href="/docs/api-reference/plugin-entry">Jump to grouped API docs for every public SDK subpath covered in this site.</Card>
</Cards>
