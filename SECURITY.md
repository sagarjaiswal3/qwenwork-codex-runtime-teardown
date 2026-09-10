# Security and Redaction Policy

The source material included runtime/session artifacts that may contain sensitive values. The following are intentionally excluded:

- bearer/JWT tokens and API keys;
- cookies and authorization headers;
- VNC/remote-session tickets;
- session, account, and user identifiers;
- personal email addresses;
- complete shell environment dumps;
- complete session/rollout transcripts;
- the complete QwenWork system prompt;
- raw archives and executable binaries;
- unnecessary proprietary/internal content.

## If a secret is found

Do not quote, mirror, or test it. Open a minimal issue naming only the affected repository path so the publication can be corrected.

## Scope

The evidence here is intended for provenance and architecture verification only. Do not use it to replay credentials, bypass access controls, evade quotas, or access infrastructure without authorization.
