---
title: "provider-auth"
description: "API reference for provider auth helpers, auth-profile storage, runtime auth resolution, and OAuth bridging."
---

Import paths:

- `openclaw/plugin-sdk/provider-auth`
- `openclaw/plugin-sdk/provider-auth-runtime`

These two modules form the public auth surface for provider plugins. `provider-auth` focuses on setup, env vars, profile storage, and input normalization. `provider-auth-runtime` focuses on execution-time auth resolution, OAuth callback handling, and the Codex auth bridge.

## provider-auth

Primary exports from `src/plugin-sdk/provider-auth.ts`:

- `ensureAuthProfileStore`, `ensureAuthProfileStoreForLocalUpdate`
- `listProfilesForProvider`, `upsertAuthProfile`, `removeProviderAuthProfilesWithLock`
- `resolveEnvApiKey`, `readClaudeCliCredentialsCached`
- `createProviderApiKeyAuthMethod`
- `applyAuthProfileConfig`, `buildApiKeyCredential`, `upsertApiKeyProfile`, `writeOAuthCredentials`
- `normalizeApiKeyInput`, `validateApiKeyInput`, `promptSecretRefForSetup`

Custom function:

```ts
function isProviderApiKeyConfigured(params: {
  provider: string;
  agentDir?: string;
}): boolean
```

`isProviderApiKeyConfigured` first checks `resolveEnvApiKey(provider)?.apiKey`, then consults the auth-profile store only if an `agentDir` is available. That makes it useful in setup wizards and doctor checks where you want a cheap "configured or not" test without fully resolving runtime auth.

## provider-auth-runtime

Important custom exports from `src/plugin-sdk/provider-auth-runtime.ts`:

```ts
const CODEX_AUTH_ENV_CLEAR_KEYS = ["OPENAI_API_KEY"] as const;

type PreparedCodexAuthBridge = {
  codexHome: string;
  clearEnv: string[];
};

type OAuthCallbackResult = { code: string; state: string };
```

```ts
function generateOAuthState(): string
function parseOAuthCallbackInput(
  input: string,
  messages?: { missingState?: string; invalidInput?: string },
): OAuthCallbackResult | { error: string }
```

```ts
async function waitForLocalOAuthCallback(params: {
  expectedState: string;
  timeoutMs: number;
  port: number;
  callbackPath: string;
  redirectUri: string;
  successTitle: string;
  progressMessage?: string;
  hostname?: string;
  onProgress?: (message: string) => void;
}): Promise<OAuthCallbackResult>
```

```ts
function isCodexBridgeableOAuthCredential(value: unknown): value is OAuthCredential
function resolveCodexAuthBridgeHome(params: {
  agentDir: string;
  bridgeDir: string;
  profileId: string;
}): string
function buildCodexAuthBridgeFile(credential: OAuthCredential): string
async function prepareCodexAuthBridge(params: {
  agentDir: string;
  bridgeDir: string;
  profileId: string;
}): Promise<PreparedCodexAuthBridge | undefined>
async function resolveApiKeyForProvider(...)
async function getRuntimeAuthForModel(...)
```

## Example

```ts
import {
  generateOAuthState,
  parseOAuthCallbackInput,
  waitForLocalOAuthCallback,
} from "openclaw/plugin-sdk/provider-auth-runtime";

const state = generateOAuthState();
const callback = await waitForLocalOAuthCallback({
  expectedState: state,
  timeoutMs: 120000,
  port: 38123,
  callbackPath: "/oauth/callback",
  redirectUri: "http://localhost:38123/oauth/callback",
  successTitle: "Acme connected",
});

const parsed = parseOAuthCallbackInput(
  `http://localhost:38123/oauth/callback?code=${callback.code}&state=${callback.state}`,
);
```

Use `provider-auth` for setup-time flows and `provider-auth-runtime` when the plugin needs runtime auth resolution or an OAuth callback server. The split keeps setup code lighter and avoids pulling runtime-only dependencies into every auth prompt.
