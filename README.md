# QwenWork Codex Runtime Teardown

> Sanitized, evidence-first forensic analysis of a QwenWork cloud-agent runtime captured on 2026-09-11.

**Researcher:** [@sagarjaiswal3](https://github.com/sagarjaiswal3)  
**Status:** Independent research; not affiliated with Alibaba, Qwen, QwenWork, OpenAI, Google, or ByteDance.

## TL;DR

A researcher-controlled QwenWork cloud environment contained a **physical OpenAI Codex CLI 0.144.6 installation**, an ACP bridge that directly depends on `@openai/codex`, QwenWork-specific Codex configuration, and a compiled MuleRun agent that provides orchestration/proxy/bootstrap layers.

The same capture also contains a media-router implementation where QwenWork-facing model IDs map to vendor-prefixed internal model IDs, including:

```text
qwenwork/open-image-2       -> openai/gpt-image-2
qwenwork/banana-image-2     -> google/nano-banana-2
qwenwork/banana-image-2-lite-> google/nano-banana-2-lite
qwenwork/seedance-2.0       -> bytedance/seedance-2.0
```

These media mappings are **not evidence that QwenWork chat inference uses those same providers**, nor do they establish billing or contractual relationships. They do establish that the captured media-routing layer can expose QwenWork-branded aliases backed by vendor-prefixed internal identifiers.

## Strongest evidence

### 1. Physical OpenAI Codex package and executable

Captured path:

```text
/opt/codex-stage/node_modules/@openai/codex/
/opt/codex-stage/node_modules/@openai/codex-linux-x64/
```

Relevant package metadata:

```text
name:       @openai/codex
version:    0.144.6
repository: https://github.com/openai/codex
```

The bundled Linux executable was run locally after extraction and reported:

```text
codex-cli 0.144.6
```

Binary SHA-256:

```text
a31ae9450a26216eb1e7c53102fd42123dd675974310b0e2ca3aa4cb622a2c15
```

### 2. ACP bridge directly depends on Codex

Captured `/opt/acp-codex/package.json` identifies:

```text
@agentclientprotocol/codex-acp 1.1.5
```

and declares:

```text
@openai/codex: ^0.144.6
```

### 3. QwenWork-specific Codex provider template

Captured `/opt/agent-config/codex/config.toml` contains:

```toml
model = "__CODEX_MODEL__"
model_provider = "qwenwork"

[model_providers.qwenwork]
name = "QwenWork"
base_url = "__QW_API_BASE_URL__/v1"
wire_api = "responses"
supports_websockets = false

[model_providers.qwenwork.auth]
command = "/usr/local/bin/fetch-qwenwork-token"
args = ["--audience", "codex"]
```

A separate live-session capture showed the deployed form using `model_provider = "qwenwork"`, a QwenWork API base URL, and `gpt-5.6-sol` as the requested model.

### 4. Static image and live-session prompt are cryptographically identical

The baked image contains:

```text
/opt/agent-config/codex/system_prompt.md
```

The earlier live session contained:

```text
~/.codex/system_prompt.md
```

Both hash to:

```text
b54847fe8f4e3df02f8a6ba37ceabeaea8a465b2c6b189858741d73f610ed25d
```

That links the independently captured static `/opt` image to the live Codex runtime state without publishing the full internal prompt.

### 5. Compiled MuleRun agent

Captured binary:

```text
/opt/mule-sdk-go/mule-agent
```

Observed version:

```text
v0.0.7-phaseC+5bc5ec12
```

Go build metadata identifies:

```text
module:   github.com/mulerun/mule-run/agent-sdk
command:  github.com/mulerun/mule-run/agent-sdk/cmd/mule-agent
Go:       1.25.5
revision: 5bc5ec12eda7091dfb952b34c00517148c53c63c
```

Embedded package paths include `internal/acp`, `internal/engineadapter`, `internal/maasproxy`, `internal/resourcebootstrap`, `internal/filesync`, and related orchestration components.

### 6. QwenWork media aliases map to vendor-prefixed models

Captured source:

```text
/opt/agent-skills/media-generate-sg/bin/cli.js
```

Relevant mappings:

```text
qwenwork/open-image-2        -> openai/gpt-image-2
qwenwork/banana-image-2      -> google/nano-banana-2
qwenwork/banana-image-2-lite -> google/nano-banana-2-lite
qwenwork/seedance-2.0        -> bytedance/seedance-2.0
```

The CLI's offline provider listing returned:

```json
{
  "providers": ["alibaba", "klingai", "minimax", "qwenwork"]
}
```

See [`EVIDENCE.md`](EVIDENCE.md) and [`evidence/media-routing-aliases.json`](evidence/media-routing-aliases.json).

## Reconstructed architecture

```text
QwenWork product / cloud UI
          |
          v
   MuleRun agent harness
          |
          +---- resource bootstrap / file sync / hooks
          |
          v
         ACP
          |
          v
@agentclientprotocol/codex-acp 1.1.5
          |
          v
   @openai/codex 0.144.6
          |
          +---- tools / MCP / sessions
          |
          v
 custom provider: qwenwork
 wire protocol: Responses
          |
          v
   QwenWork MaaS/API layer
          |
          v
 server-side routing not established by this capture
```

## What this proves

The captured QwenWork cloud image physically bundles OpenAI Codex CLI 0.144.6 and connects it to a QwenWork-specific agent/configuration stack. This is stronger than inferring Codex from environment-variable names alone: the Codex package, native executable, ACP dependency, configuration template, runtime metadata, and matching prompt hash are all independently observable artifacts.

The media router additionally contains explicit QwenWork-to-vendor model alias mappings.

## What this does **not** prove

This repository does not claim:

- every QwenWork deployment uses this exact version or stack;
- every QwenWork model request goes directly to OpenAI;
- the `gpt-5.6-sol` model label proves a specific upstream billing/account relationship;
- the media-model aliases prove the provider used by QwenWork chat models;
- use of Codex or third-party model infrastructure is inherently improper;
- QwenWork is merely a reskin of Codex.

Those would require additional server-side evidence.

## Source preservation

The raw archives are withheld because session material and internal files may contain credentials, identifiers, or unnecessary proprietary material. Instead this repository publishes narrowly scoped facts, hashes, sanitized excerpts, and reproducible metadata.

Primary archive fingerprints are in [`evidence/SHA256SUMS.txt`](evidence/SHA256SUMS.txt).

## Public references

- OpenAI Codex: https://github.com/openai/codex
- Codex configuration reference: https://developers.openai.com/codex/config-reference/
- Agent Client Protocol Codex adapter: https://github.com/agentclientprotocol/codex-acp
- QwenWork documentation: https://docs.qwenwork.ai/

## Responsible-use note

This repository documents software architecture and provenance. It is not a guide to replay credentials, bypass quotas, access other users' data, or obtain unauthorized service access.
