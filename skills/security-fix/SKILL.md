---
name: security-fix
description: Fix and verify one validated or plausible security vulnerability with the smallest repo-native patch, regression test, and bypass review; reports fixed, no_change, or blocked.
---

# Security Fix

Turn one security finding into a minimal, verified code change — or prove the code is already safe. Use only when the user asks to fix a security vulnerability (e.g. a finding from the `security-audit` report, a scanner alert, a bug-bounty report). Not for ordinary bugs or style issues.

Method adapted from OpenAI's codex-security `fix-finding` skill (Apache-2.0), rewritten for Claude Code.

## Verification mode

- **standard** (default): you may run builds, tests, and a focused reproducer locally.
- **static**: when the user asks for it, or the project can't be built/run here. Assess source and diff only; say runtime checks were not performed and never count them as passed.

Never contact external services, use discovered credentials, or exploit anything outside a local test.

## 1. Establish the patch contract (before editing)

Read the affected code, its direct callers, nearby helpers, and existing tests. Write down, from repository evidence:

- the attacker-controlled input and concrete source→sink path (or broken control)
- the **security invariant** to enforce, and the **narrowest shared enforcement boundary** (the helper/middleware/validator every path goes through — not just the one reported line)
- legitimate behavior, APIs, error semantics, and compatibility constraints that must survive
- the closest existing precedent in the repo for validation/escaping/authz/error handling

Treat it as a data-flow and boundary problem, not the specific example payload. Consider equivalent encodings, parser forms, aliases, case/Unicode normalization, other callers and sinks, and every copy of security-sensitive state that could bypass the fix. Handle unsafe input explicitly — reject it; don't silently truncate or reinterpret it into another reachable form.

If evidence shows the path is already safe, stop with outcome `no_change` and cite the control. Don't make speculative changes.

## 2. Reproduce (standard mode, when feasible)

Write the smallest high-signal test that drives the malicious input through the real boundary, plus one legitimate control case through the same path. Confirm the malicious case currently fails the invariant.

## 3. Implement the smallest repo-native fix

- Fix at the shared boundary; prefer existing helpers and established library APIs (parameterized queries, framework escaping, `realpath`+prefix check, allowlists, existing authz decorators) over hand-rolled logic.
- Don't broaden into redesign, cleanup, or sibling findings — note those for the user instead.
- If the finding came with a proposed patch, use it as a starting candidate but keep only hunks that actually enforce the invariant without breaking legitimate behavior.
- Add or update the regression test.

## 4. Challenge your own patch

Before verifying, try to break it:
- inspect every direct caller of each changed helper and both outcomes of each changed condition
- look for **one sibling path, representation, or copy** that still reaches the vulnerable sink
- look for **one ordinary/default input** the patch now wrongly rejects or reinterprets

Revise if either exists. For High/Critical findings or tricky boundaries, launch a fresh general-purpose subagent as a read-only reviewer: give it the finding, repo root, verification mode, and the diff — but not your rationale or test results — and ask for surviving bypasses and regressions. Confirm its concerns against source before acting on them.

## 5. Verify

**Standard:** inspect the final diff; run the narrowest relevant syntax/type/build check; rerun the exploit test and at least one alternate malicious input class; rerun the legitimate control, nearby tests, and the package's required checks.

**Static:** inspect the final diff; trace the original exploit path and alternate inputs through the changed control; trace callers and existing tests to show legitimate behavior is intact. Cite source for each gate and state what it can't establish.

Remove any temporary files or debug changes.

## 6. Outcome

Report in chat:

- **Outcome:** `fixed`, `no_change`, or `blocked` (vulnerability may be real but a needed fact, tool, access, or product/compat decision is missing — name it).
- The vulnerable path, the invariant, and the legitimate behavior preserved.
- Patch strategy and why it's the narrowest complete option.
- Files changed; tests added.
- Verification performed (commands and results) and anything not verified.
- Residual risk or related sibling issues noticed but not fixed.

If the finding came from a `security-audit` report, append a short "Fix status" note under that finding in its `report.md`. Do not commit unless the user asks.
