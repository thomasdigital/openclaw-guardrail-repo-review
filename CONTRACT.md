# PROMPT CONTRACT — External Guardrail/Gating Framework Adoption

**Source of truth.** Version 1.0-final · 2026-09-17 · Owner: Steve (CEO/Integrator) · **Next review: 2026-03-17 (6-month)** · Status: SHIP-READY (Codex red-teamed, r1 converged, 6 fixes folded in)

This contract governs how OpenClaw evaluates and adopts any third-party guardrail, gating,
or agent-safety framework. It is the authority future sessions consult BEFORE installing
external agent-control code. Where a session's judgment and this contract disagree, this file
wins until Victor amends it.

---

## 1. The decision rule (adopt / mine / reject)

For any external guardrail/gating/agent-safety repo, decide in this order and STOP at the first that fires:

1. **PRIOR-ART CHECK (mandatory first) — BEHAVIOR, not filename** *(Codex fix #1)*. Grep the live
   fabric (`~/.openclaw/bin`, guards, hooks) for the capability, THEN prove behavioral coverage with a
   test: feed the exact adversarial input the repo defends against (e.g. a base64-encoded secret in a
   tool result; 3 identical repeated calls) and confirm our guard actually catches it. A same-named
   file is NOT proof — "COVERED" may only be asserted after the behavioral test passes. If OpenClaw
   behaviorally covers it, **REJECT for adoption**; the only downstream question is rule 4.
   **Gap-override** *(Codex fix #2)*: if the behavioral test FAILS or the foreign tool is measurably
   stronger on a dimension that matters (see rule 4's thresholds), the "already have a naive version"
   rejection does NOT apply — escalate to rule 4 (MINE) or, only past rule 3, a scoped adoption.
2. **SUBTRACTION GATE.** Installing external code adds process count and blast radius. Per the
   Subtraction Doctrine, a new gate/hook/daemon is admissible ONLY with (a) a retirement criterion,
   (b) an SLO (default ≤2% false-block), and (c) proof that no existing gate can be EXTENDED to cover
   it (consolidation-first). Fail any → **REJECT for adoption**; consider MINE instead.
3. **BLAST-RADIUS GATE (security-critical)** *(Codex fix #4)*. A repo that intercepts tool calls,
   replaces native tools, patches config, or runs a privileged daemon is a supply-chain surface. It
   must clear ALL of: independent code audit · commit-history sanity · maintainer reputation ·
   **transitive-dependency scan (npm/pip audit, no unpinned or abandoned deps)** · **declared-permission
   review (what files/network/exec it can reach vs what it needs)** · **license compatibility (rule 6)**.
   A low-commit, low-star, unaudited high-privilege interceptor is **REJECT — do not install**,
   regardless of stated purpose. A security tool earns MORE scrutiny, not less. NOTE: this gate is
   NOT a blanket ban on all external code — a well-audited, well-maintained, appropriately-scoped tool
   that clears every check above and passes rules 1–2 may proceed. The bar is high, not infinite.
4. **CAPABILITY-DELTA CHECK — with objective thresholds** *(Codex fix #3)*. "Genuinely weaker on a
   dimension that matters" must be decided by a NAMED metric with a pass/fail line, not by feel.
   Examples: injection-catch rate on a fixed adversarial corpus (theirs vs ours, ≥X% delta to act);
   false-block rate; mean-time-to-detect. If our version loses on the named metric, the action is
   **MINE the pattern** — reimplement natively inside the existing fabric under a Birth Certificate
   (retirement criterion + SLO). Never install the foreign code to close a gap MINE can close.
5. **BENCHMARK CHECK.** Even a REJECT-for-adoption repo may be worth keeping as an external
   eval/benchmark target — a yardstick (e.g. the corpus in rule 4) that our fabric matches or beats.

Default posture: **ADOPT-NONE.** Adoption is the exception that must be argued past all five gates.

**Every run of this rule appends one line to the decision log** *(Codex fix #6)*:
`~/.openclaw/state/guardrail-adoption-log.ndjson` — {date, repo, verdict, gate that fired, evidence}.

---

## 2. Verdict on the three repos reviewed 2026-09-17

| Repo | Stars / maturity | Verdict | Basis |
|---|---|---|---|
| kraulerson/claude-dev-framework | 9★, v4.3.1, active | REJECT adoption · MINE nothing structural | 17 hooks / 14 rules all duplicate existing OpenClaw fabric |
| ilkerbbb/systematic-claw | 12★, 7.2k LoC plugin | REJECT adoption · MINE nothing | 15 gates already covered; doom-loop weaker than loop-circuit-breaker.py |
| Claw-Guard/ClawGuard | 27★, 5 commits | REJECT — DO NOT INSTALL (blast-radius) | privileged daemon, unaudited; L2/L3 regex-only, weaker than our 2-stage taint+LLM defense |

**Reconciled with Codex red-team (round 1, converged 2026-09-17).** "Install none" survives
unchanged; "mine almost nothing" is upgraded to "benchmark three candidate gaps, then mine 0–2
natively." Three candidates were behaviorally checked against the live fabric:

| Candidate gap (from the repos) | Verdict | Evidence file |
|---|---|---|
| Durable task/plan state + checkpoint/rollback (systematic-claw) | COVERED | orchestration-checkpoint.py, lane-claim.py, plan_approval_hash.py |
| Human approval queue for high-risk actions (ClawGuard supervised mode) | COVERED | approval-signer, andon-tick.py, Command Center "needs-you", hard-lock per-turn gates |
| Per-project-**type** enforcement profiles (claude-dev-framework's 5 profiles) | PARTIAL GAP | OpenClaw has project *identity* (project-ids.json) but not per-type gate profiles |

**Two defensible native actions (both MINE, gated, NOT adoptions):**
1. ≤20-min regex diff of ClawGuard's ~30 secret-detection patterns vs OpenClaw's taint/scrub set —
   harvest any pattern we lack, discard the rest. Gated on confirming ours is not already a superset.
2. Log per-project-type gate profiles to the frontier-gap ledger as a native pattern to build IF
   the project mix diversifies beyond web design/dev. Low urgency today (homogeneous mix).

**"Frontier-ahead" claim, corrected:** OpenClaw is ahead on prompt-injection defense specifically
(2-stage regex+local-LLM with homoglyph/base64 normalization vs ClawGuard's regex-only), PENDING a
behavioral test (inject plaintext + encoded secrets, repeated calls). Not a blanket superiority claim.

**Benchmark value retained:** the three repos' stated controls become an external coverage checklist
(the parity scoreboard in the deliverable) — a yardstick, not installed code.

---

## 3. Evidence anchors (files, not recall)
- Doom-loop coverage → `~/.openclaw/bin/loop-circuit-breaker.py` (per-loop budget breaker)
- Injection/sanitizer coverage → `~/.openclaw/bin/hook-posttooluse-injection-guard.py` + `taint-llm-check.py` (2-stage regex+LLM, homoglyph/base64 normalization)
- Secret handling breadth → predeploy-secret-gate.py, secret-scrub-guard.py, hook-stop-egress-secret-guard.py, taint-exfil-check.py, taint-publish-scan.py
- Governing doctrine → Subtraction Doctrine (`~/.claude/protocols/subtraction-doctrine.md`)

## 4. Revisit triggers (a REJECT is scoped, not permanent) *(Codex fix #6)*
A rejected repo is re-evaluated against §1 when ANY fires: (a) it crosses a maturity bar — e.g.
≥500 stars AND ≥6 months of active maintenance AND a real audit trail; (b) a behavioral test in §1.1
starts FAILING for OpenClaw (a real gap opened); (c) the 6-month review date arrives; (d) Victor asks.
A rejection recorded today never means "never look again."

## 5. Amendment protocol
Change this file only with Victor's sign-off OR a passed Codex red-team recorded in §2. Every
amendment stamps a new version line at the top and MUST update the Next-review date. License note:
all three repos reviewed here are MIT (permissive, compatible); any future candidate's license is
checked at gate 3. This file is the SSoT; artifacts/replies cite it, never restate it as firmer.
