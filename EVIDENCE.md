# Evidence

## Evidence set A — physical Codex installation

### A1. npm package

Captured path:

```text
optqwen/codex-stage/node_modules/@openai/codex/package.json
```

Sanitized metadata:

```json
{
  "name": "@openai/codex",
  "version": "0.144.6",
  "description": "Codex CLI is a coding agent from OpenAI that runs locally on your computer.",
  "license": "Apache-2.0",
  "repository": "git+https://github.com/openai/codex.git"
}
```

SHA-256 of the captured package manifest:

```text
b701b7d7b7683263e5612e612c468c526d78c3deb1360741e976dc40e0456919
```

### A2. native Codex executable

Captured path:

```text
optqwen/codex-stage/node_modules/@openai/codex-linux-x64/vendor/x86_64-unknown-linux-musl/bin/codex
```

Observed file metadata:

```text
ELF 64-bit LSB pie executable, x86-64, static-pie linked, stripped
size: 298,516,528 bytes
```

Running the extracted executable with `--version` returned:

```text
codex-cli 0.144.6
```

SHA-256:

```text
a31ae9450a26216eb1e7c53102fd42123dd675974310b0e2ca3aa4cb622a2c15
```

This is direct physical evidence, not an inference from naming alone.

## Evidence set B — ACP bridge

Captured path:

```text
optqwen/acp-codex/package.json
```

Metadata:

```text
name:    @agentclientprotocol/codex-acp
version: 1.1.5
```

Relevant dependency:

```text
@openai/codex: ^0.144.6
```

Manifest SHA-256:

```text
4333e2ea31306ae95fd6e115d90130f926f576cc54d6243fd3c2a44a9e550770
```

## Evidence set C — QwenWork Codex configuration

Static image template:

```text
optqwen/agent-config/codex/config.toml
```

Relevant excerpt:

```toml
model = "__CODEX_MODEL__"
model_provider = "qwenwork"
model_reasoning_effort = "high"
approval_policy = "never"
sandbox_mode = "danger-full-access"

[model_providers.qwenwork]
name = "QwenWork"
base_url = "__QW_API_BASE_URL__/v1"
wire_api = "responses"
supports_websockets = false

[model_providers.qwenwork.auth]
command = "/usr/local/bin/fetch-qwenwork-token"
args = ["--audience", "codex"]
timeout_ms = 5000
```

Template SHA-256:

```text
1e8f3b73f5dffc1a38c60c06c4a70fe390cd1d4006f5bec0e8071e3c5d26f569
```

A separate live-session capture contained the deployed provider configuration and reported `model_provider=qwenwork`, `cli_version=0.144.6`, and `originator=mule-agent-go`.

## Evidence set D — cross-capture prompt hash

Static image:

```text
optqwen/agent-config/codex/system_prompt.md
```

Earlier live runtime:

```text
codex/system_prompt.md
```

Both SHA-256 values are:

```text
b54847fe8f4e3df02f8a6ba37ceabeaea8a465b2c6b189858741d73f610ed25d
```

The static file is 21,412 bytes. The complete prompt is intentionally withheld. The hash match demonstrates that the static image and live runtime share the same QwenWork instruction artifact.

## Evidence set E — MuleRun agent

Captured path:

```text
optqwen/mule-sdk-go/mule-agent
```

Observed:

```text
version: v0.0.7-phaseC+5bc5ec12
size: 19,148,984 bytes
SHA-256: a2200cf8f83522d5857619405f2acf0fb3e2117dda5906f0ece6a35150edc5d6
```

`go version -m` reports:

```text
Go version: go1.25.5
command: github.com/mulerun/mule-run/agent-sdk/cmd/mule-agent
module: github.com/mulerun/mule-run/agent-sdk
VCS revision: 5bc5ec12eda7091dfb952b34c00517148c53c63c
VCS time: 2026-09-10T12:03:21Z
```

Embedded package paths include:

```text
internal/acp
internal/engineadapter
internal/maasproxy
internal/resourcebootstrap
internal/filesync
internal/runner
internal/supervisor
internal/engineadapter/pluginhooks
```

## Evidence set F — QwenWork media routing aliases

Captured file:

```text
optqwen/agent-skills/media-generate-sg/bin/cli.js
```

File SHA-256:

```text
e4279b7d3ef88fd410443ab009f8597d1fdc4d56974b62d0c5dcce56647eafe7
```

The router registry contains QwenWork-facing IDs with `real.modelId` values pointing to vendor-prefixed model IDs:

```text
qwenwork/open-image-2        -> openai/gpt-image-2
qwenwork/banana-image-2      -> google/nano-banana-2
qwenwork/banana-image-2-lite -> google/nano-banana-2-lite
qwenwork/seedance-2.0        -> bytedance/seedance-2.0
```

The same captured code contains vendor API-path strings including:

```text
/vendors/openai/v1/gpt-image-2/generation
/vendors/openai/v1/gpt-image-2/edit
/vendors/google/v1/nano-banana-2/generation
/vendors/bytedance/v1/seedance-2.0/text-to-video/generation
```

An offline invocation of the captured router CLI:

```text
node cli.js list --providers
```

returned:

```json
{
  "providers": [
    "alibaba",
    "klingai",
    "minimax",
    "qwenwork"
  ]
}
```

This supports the narrower conclusion that the media layer exposes a QwenWork abstraction while internally retaining vendor-prefixed routing identifiers. It does not establish a direct commercial relationship or the provider used for chat inference.

## Evidence set G — source archive measurements

The extracted `optqwen` tree contained:

```text
19,018 files
2,416 directories
1,195,952,987 bytes (filesystem tree size via du -sb)
```

Raw RAR size:

```text
294,608,577 bytes
```

Raw RAR SHA-256:

```text
6a1a139bef2670a5acd0cff41362f1495538f45aaa482f72748364b32e89e8ab
```

The archive itself is not redistributed.

## Confidence

### High confidence

1. The captured image physically contains OpenAI Codex CLI 0.144.6.
2. The included ACP bridge directly depends on `@openai/codex`.
3. QwenWork supplies its own Codex configuration/provider template.
4. The static image and live runtime share the exact same system-prompt artifact by SHA-256.
5. A compiled MuleRun agent provides orchestration/proxy/bootstrap components around the engine.
6. The captured media router explicitly maps several `qwenwork/*` aliases to vendor-prefixed model IDs.

### Not established

1. The complete server-side routing path after a request reaches QwenWork infrastructure.
2. Any specific upstream billing, partnership, or commercial arrangement.
3. Whether every QwenWork session uses this same stack.
4. Whether chat-model labels are always forwarded unchanged to an upstream vendor.
5. Whether the media aliases imply anything about QwenWork's chat-model provider.
