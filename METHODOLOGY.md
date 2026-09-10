# Methodology

## Acquisition context

Two artifacts from a researcher-controlled QwenWork cloud environment were examined:

1. a live Codex home/session snapshot; and
2. a later archive of the environment's `/opt`-side runtime files.

The raw artifacts are not redistributed.

## Procedure

1. Extracted the `/opt` archive using UNRAR 7.23.
2. Enumerated the tree and measured file/directory counts and filesystem size.
3. Located executable/package evidence for Codex, ACP, MuleRun, skills, MCPs, and media-routing components.
4. Executed only local/offline identification commands such as `codex --version`, `mule-agent --version`, `go version -m`, and the media router's local provider listing.
5. Did **not** replay captured credentials or use extracted authentication material against remote services.
6. Computed SHA-256 hashes for source archives and key artifacts.
7. Cross-checked the static image against the independent live-session capture.
8. Verified that the static and live QwenWork system-prompt files have identical SHA-256 hashes.
9. Reduced potentially proprietary files to minimal factual excerpts or structured metadata necessary to substantiate the findings.
10. Scanned the publication set for JWT-like values, email addresses, bearer credentials, and obvious session identifiers before publication.

## Claim standard

The report distinguishes three levels:

- **Observed:** directly present in captured files or local executable output.
- **Inferred:** architecture supported by multiple observed artifacts.
- **Not established:** facts that would require server-side or contractual evidence.

This distinction is especially important for model routing: a client-side model identifier or alias is not by itself proof of the ultimate inference provider or payer.
