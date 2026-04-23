---
title: "video-generation"
description: "API reference for the typed video-generation provider contract and the Dashscope-compatible helper exports."
---

Import path: `openclaw/plugin-sdk/video-generation`

Unlike many SDK modules, `src/plugin-sdk/video-generation.ts` defines most of its public types locally so the generated declaration surface stays stable for external package consumers. It is the contract to implement when a plugin registers `api.registerVideoGenerationProvider(...)`.

## Core Types

```ts
type GeneratedVideoAsset = {
  buffer?: Buffer;
  url?: string;
  mimeType: string;
  fileName?: string;
  metadata?: Record<string, unknown>;
};

type VideoGenerationResolution = "480P" | "720P" | "768P" | "1080P";

type VideoGenerationAssetRole =
  | "first_frame"
  | "last_frame"
  | "reference_image"
  | "reference_video"
  | "reference_audio";
```

```ts
type VideoGenerationRequest = {
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
type VideoGenerationProvider = {
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

## Capabilities

The contract makes capability negotiation explicit. `VideoGenerationModeCapabilities` can declare max counts, supported durations, size or aspect-ratio support, audio or watermark support, and a typed `providerOptions` map. That means setup and runtime code can reject unsupported options before they silently reach the wrong provider.

## Dashscope-compatible Helpers

This module also re-exports a ready-made set of helpers for Dashscope-style task APIs:

- `buildDashscopeVideoGenerationInput`
- `buildDashscopeVideoGenerationParameters`
- `runDashscopeVideoGenerationTask`
- `pollDashscopeVideoTaskUntilComplete`
- `downloadDashscopeGeneratedVideos`
- `extractDashscopeVideoUrls`
- `resolveVideoGenerationReferenceUrls`
- `DASHSCOPE_WAN_VIDEO_CAPABILITIES`
- `DEFAULT_DASHSCOPE_WAN_VIDEO_MODEL`

## Example

```ts
import type {
  VideoGenerationProvider,
  VideoGenerationRequest,
  VideoGenerationResult,
} from "openclaw/plugin-sdk/video-generation";

async function generateVideo(req: VideoGenerationRequest): Promise<VideoGenerationResult> {
  return {
    videos: [
      {
        url: "https://cdn.acme.example/video.mp4",
        mimeType: "video/mp4",
      },
    ],
    model: req.model,
  };
}

export const acmeVideoProvider: VideoGenerationProvider = {
  id: "acme-video",
  label: "Acme Video",
  defaultModel: "acme-video/v1",
  capabilities: {
    resolutions: ["720P"],
    supportsAspectRatio: true,
    providerOptions: {
      seed: "number",
    },
  },
  generateVideo,
};
```

If your provider's task lifecycle already looks like Dashscope's, reuse the exported helpers. If not, implement only the contract types and keep provider-specific polling or download behavior in your own extension code.
