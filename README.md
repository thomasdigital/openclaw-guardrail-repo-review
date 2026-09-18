# OpenClaw Guardrail / Gating Repo Review

A frontier-level review of three third-party agent-safety repos against OpenClaw's
existing guard/hook fabric, plus the decision contract and evidence that came out of it.

**Bottom line: adopt none. Two standard secret-detection patterns were mined natively; no outside code was installed.**

## The three repos reviewed (2026-09-17)

| Repo | Maturity | Verdict |
|---|---|---|
| `kraulerson/claude-dev-framework` | 9★, active | REJECT adoption — 17 hooks / 14 rules all duplicate existing fabric |
| `ilkerbbb/systematic-claw` | 12★, 7.2k LoC | REJECT adoption — 15 gates already covered; doom-loop weaker than ours |
| `Claw-Guard/ClawGuard` | 27★, 5 commits | REJECT — DO NOT INSTALL — unaudited privileged daemon, regex-only defense |

OpenClaw already runs a more mature version of everything the three do: a guard/hook
fabric under a Subtraction Doctrine, a two-stage (regex + local-LLM) injection defense,
and comprehensive secret handling.

## What's in this repo

| File | What it is |
|---|---|
| [`CONTRACT.md`](CONTRACT.md) | The source-of-truth decision rule for adopting any external guardrail/gating framework (v1.0-final, Codex red-teamed). Future sessions consult this before installing outside agent-control code. |
| [`REGEX-DIFF.md`](REGEX-DIFF.md) | The secret-pattern diff: ClawGuard's 15 patterns vs OpenClaw's scrubber + gitleaks. Ours is a strict superset. Two gitleaks-class patterns (`glpat-` GitLab tokens, PEM private-key blocks) were mined natively. |
| [`deploy/index.html`](deploy/index.html) | The visual deliverable (Victor Artifact Kit): TL;DR "install none," a coverage map, and the reconciliation with the Codex red-team. |

## How the decision was made

1. **Prior-art check (behavior, not filename)** — grep the live fabric, then prove coverage with an adversarial test.
2. **Subtraction gate** — a new gate/hook is admissible only with a retirement criterion + SLO, and only if no existing gate can be extended.
3. **Blast-radius gate** — anything that intercepts tool calls or runs a privileged daemon must clear a full supply-chain audit.
4. **Capability-delta check** — if ours loses on a named metric, mine the pattern natively; never install foreign code to close a gap.
5. **Benchmark check** — a rejected repo may still be kept as an external yardstick.

Default posture: **ADOPT-NONE.** Every run of the rule appends a line to an adoption decision log.

## Outcome

- No third-party code adopted or installed.
- Two secret-detection patterns (`glpat-`, PEM key blocks) reimplemented natively in OpenClaw's own scrubber, tested and corpus-checked for false positives.
- The three repos retained only as an external coverage checklist.
