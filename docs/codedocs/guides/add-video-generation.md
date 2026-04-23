---
title: "Add Video Generation"
description: "Register a video-generation provider with typed capability metadata and provider-specific request handling."
---

Video generation uses a dedicated public contract in `openclaw/plugin-sdk/video-generation`. The main implementation unit is a `VideoGenerationProvider` with a `capabilities` block and a `generateVideo(req)` method.

<Steps>
  <Step>
  ### Declare capabilities up front

```ts
import type { VideoGenerationProvider } from "openclaw/plugin-sdk/video-generation";

export const acmeVideoProvider: VideoGenerationProvider = {
  id: "acme-video",
  label: "Acme Video",
  defaultModel: "acme-video/v1",
  capabilities: {
    resolutions: ["720P", "1080P"],
    supportsAspectRatio: true,
    supportsAudio: true,
    providerOptions: {
      seed: "number",
      style: "string",
    },
  },
  async generateVideo(req) {
    return {
      videos: [
        {
          url: "https://cdn.acme.example/video.mp4",
          mimeType: "video/mp4",
          fileName: "video.mp4",
        },
      ],
      model: req.model,
    };
  },
};
```

  </Step>
  <Step>
  ### Validate provider options and source assets in your implementation

```ts
async function generateVideo(req: VideoGenerationRequest): Promise<VideoGenerationResult> {
  if (req.providerOptions?.seed && typeof req.providerOptions.seed !== "number") {
    throw new Error("seed must be a number");
  }

  if ((req.inputImages?.length ?? 0) > 1) {
    throw new Error("Acme accepts at most one input image");
  }

  return {
    videos: [{ url: "https://cdn.acme.example/out.mp4", mimeType: "video/mp4" }],
    metadata: { aspectRatio: req.aspectRatio ?? "16:9" },
  };
}
```

  </Step>
  <Step>
  ### Register the provider from your plugin entry

```ts
import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry";

export default definePluginEntry({
  id: "acme-video",
  name: "Acme Video",
  description: "Adds Acme video generation support",
  register(api) {
    api.registerVideoGenerationProvider(acmeVideoProvider);
  },
});
```

  </Step>
</Steps>

If your provider matches Dashscope's task model, the SDK also exposes helpers such as `runDashscopeVideoGenerationTask` and `pollDashscopeVideoTaskUntilComplete` so you can keep polling and download logic out of the plugin entry file.
