# Secret-pattern diff — ClawGuard vs OpenClaw (2026-09-17)

**Verdict: OpenClaw is a strict SUPERSET. Zero patterns to harvest. One 3-line verification worth doing.**

## What ClawGuard actually has
README claims "30+ patterns"; the real file (`clawguard/sanitizer.py`) has **15**:
AWS access key, AWS secret key, GitHub token, GitLab PAT (glpat-), JWT, Bearer token,
SSH/PEM private key, DB connection string, generic api-key, Slack (xox), Stripe (sk_live),
SendGrid (SG.), generic private key, password, secret/token. (README overclaim noted.)

## What OpenClaw has — two layers
1. **Native scrubber** (`lib/secret_scrub.py`, ~49 pattern lines, loaded by secret-scrub-guard.py
   + predeploy-secret-gate.py): AWS, GCP, GitHub, JWT, Bearer, api-key, Slack, Stripe, SendGrid,
   password, secret, token — **plus npm and Twilio, which ClawGuard does NOT have.**
2. **gitleaks 8.30.1** (~165 default rules) runs as a cross-check; secret-scrub-guard.py even has a
   function to "run gitleaks over the corpus and report what our scrubber cannot see." gitleaks'
   ruleset is a strict superset of ClawGuard's 15 (GitLab PAT, PEM keys, DB URIs all included).

## Coverage map
| ClawGuard pattern | OpenClaw native scrubber | gitleaks (cross-check) |
|---|---|---|
| AWS access/secret, GitHub, JWT, Bearer, Slack, Stripe, SendGrid, api-key, password, secret/token | ✅ | ✅ |
| GitLab PAT (glpat-) | ? (not confirmed in native) | ✅ |
| SSH/PEM private-key block | ? (not confirmed in native) | ✅ |
| DB connection string (user:pass@) | ? (not confirmed in native) | ✅ |
| **npm token, Twilio** | ✅ (ClawGuard lacks these) | ✅ |

## RESOLUTION (2026-09-17) — live module checked, 2 of 3 added
Checked the LIVE module `~/.openclaw/bin/lib/secret_scrub.py` (not the backup):
- **DB connection string** — ALREADY covered by `URL_CRED_RE` (line 167: `://user:secret@host`). No change.
- **GitLab PAT (glpat-)** — was a REAL gap. ADDED `glpat-[A-Za-z0-9_\-]{20,}` to `SECRET_VALUE_RE`.
- **PEM/SSH private-key block** — was a REAL gap (`private_key` was only caught as a JSON KEY NAME,
  not as a raw `-----BEGIN … PRIVATE KEY-----` body). ADDED `PEM_KEY_RE`, wired into `redact_text`
  (redacts the whole block) and `scan_text` (new kind `private-key`).

Verification:
- Module selftest PASSES with new assertions (detection + redaction + idempotence + public-key
  non-false-positive all green).
- Corpus measurement: **0 files** in the scrubber's live operating scope match either new pattern —
  no mass-rewrite risk (same discipline every prior addition in this file follows).
- Both patterns are distinctive literal markers (near-zero false-positive by construction), the same
  class as the existing ghp_ / sk_live_ / SG. prefixes.

**Rollback:** `cp ~/.openclaw/backups/secret_scrub-20260917-180421.py.bak ~/.openclaw/bin/lib/secret_scrub.py`

**Nothing from ClawGuard's repo was imported.** Two standard gitleaks-class patterns mined natively.
Diff closed.
