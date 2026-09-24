---
name: fail-closed-review
description: >-
  Use when reviewing PRs for governance/authz products (G-MAR style).
  Fail-closed rubric — find fail-open excepts, body-vs-credential authority,
  optional-field skips, second doors, comment-claimed auth, gate default-deny
  gaps. Findings only; do not invent exploit PoCs.
---
# Fail-Closed Review

PR review rubric for governance and authorization products (G-MAR / Vorxeo style). Default stance: **anything not proven deny is fail-open until shown otherwise.**

You produce findings. You do **not** write exploit PoCs, attack scripts, or offensive cyber procedures. If a hole is clear from the diff and tests, state the hole with evidence paths and the required fix.

## When this skill applies

- Reviewing PRs that touch authz gates, policy engines, RLS, MCP/API auth, admin surfaces, feature flags that gate enforcement.
- Deep review role in `handoff-faber-rigor` (Rigor).
- After `adversarial-qa` finds a hole — convert to structured review findings.
- Cross-check with **guards-that-scan** and **reachability-audit** when those skills apply to the same change.

## Checklist (run every PR)

### 1. Fail-open except

- Search for `except`, `catch`, bare `pass`/`continue` around auth/policy decisions.
- Failure mode **except-as-allow**: error in the gate → request proceeds.
- Required: on gate error → deny (or hard fail the request), never continue as authenticated/authorized.

### 2. Authority from body vs credential

- Who is the subject? Must come from verified credential/session/signature — **not** from request body, query, or spoofable header alone.
- Failure mode **body-as-principal**: `user_id` in JSON trusted over token subject.
- Required: body may propose; credential decides. Reject mismatches.

### 3. Optional field skipping control

- Optional auth/policy fields that, when omitted, skip the check.
- Failure mode **omit-to-bypass**: client drops `org_id` / `scope` / `audience` → gate no-ops.
- Required: omit → deny or explicit safe default; never "no field = no check".

### 4. Second door / inverse route

- Same capability reachable via admin API, MCP tool, CLI, webhook, internal job, GraphQL mutation, or legacy path.
- Failure mode **second-door**: front door hardened; back door open.
- Required: front-door parity (`contract-and-compat`); list all surfaces; same gate or documented exception with default deny.

### 5. Comment claiming auth happened above

- Comments like "auth checked in middleware" without a call site that still runs for this path.
- Failure mode **comment-auth**: comment ≠ enforcement.
- Required: reachable call to the gate; prefer types/guards that won't compile/run without it. See **guards-that-scan**.

### 6. Gate default deny

- New routes, tools, flags: default must be deny until allowlist/policy says allow.
- Failure mode **default-allow**: missing policy → open.
- Required: explicit allow; missing config → refuse. Irreversible sweeps use off/warn/enforced with default **warn** until verified (`migration-and-data-safety`).

### 7. Client-visible lies

- Response fields that imply enforcement when mode is `not_enforced` / shadow / warn-only.
- Failure mode **enforcement-lie**: P0 under `contract-and-compat`.
- Required: truth in the contract; no "blocked" when only logged.

### 8. Findings only — no exploit PoCs

- Do not attach payloads, exploit scripts, or step-by-step attack recipes.
- State: what fails closed incorrectly, where, why it is fail-open, what fix is required.
- Product adversary tests belong in `adversarial-qa` (matrix), still without offensive PoCs.

## Output format (required)

For each finding, use this block. Severity is mandatory.

```text
### Finding
- Severity: P0 | P1 | P2 | P3
- Title: <short>
- Evidence path: <file:line or route/tool name>
- Why fail-open: <one concrete sentence>
- Required fix: <enforceable change; no PoC>
- Surfaces also at risk: <list or "unknown — needs reachability-audit">
- Verified? : cite verified-delivery receipt if re-tested, else "review-only"
```

### Severity guide

| Sev | Use when |
|-----|----------|
| **P0** | Authz bypass, client-visible enforcement lie, default-allow on new public surface |
| **P1** | Second door without gate, omit-to-bypass on optional control, body-as-principal |
| **P2** | Comment-auth, missing tests for deny path, warn-mode without plan to enforce |
| **P3** | Docs drift, naming confusion, non-security hygiene tied to the gate |

## Review verdict

Align with `handoff-faber-rigor`:

- **Approve** — checklist clean; deny paths tested (Verified).
- **Approve-with-follow-ups** — no P0/P1 open; follow-ups listed with owners.
- **Request-changes** — any P0/P1, or Verified evidence missing for claimed fixes.

Rigor does not Approve on "looks fine". Faber does not self-certify.

## Grep / search prompts (defensive)

Use these as review search themes (adapt to stack); they are not attack recipes:

- bare except around `authorize` / `require_` / `check_policy` / `enforce`
- request body fields named like `user_id`, `role`, `tenant_id` used in allow decisions
- optional schema fields tied to auth middleware
- duplicate route registrations / MCP tool names / CLI subcommands for same mutation
- strings `not_enforced`, `shadow`, `dry_run`, `warn_only` near user-facing responses
- `FORCE ROW LEVEL SECURITY` paired with superuser connection assumptions

## Failure modes (named)

| Name | Symptom |
|------|---------|
| **except-as-allow** | Exception → continue |
| **body-as-principal** | Body subject beats credential |
| **omit-to-bypass** | Missing field skips gate |
| **second-door** | Alternate surface ungated |
| **comment-auth** | Comment without reachable gate |
| **default-allow** | Missing policy → open |
| **enforcement-lie** | API says blocked, mode is warn |
| **review-as-PoC** | Reviewer invents exploit steps — stop; finding only |

## Interaction with other skills

- **verified-delivery**: claimed "fixed" must show deny-path Verified.
- **adversarial-qa**: converts holes into attack×surface matrix; stop-on-hole then fix.
- **contract-and-compat**: front-door parity and not_enforced honesty.
- **migration-and-data-safety**: RLS / role / expand-migrate-contract.
- **reachability-audit**: map second doors before Approve.
- **code-that-holds**: prefer unforgeable gates over comments.

## Anti-patterns

- Approving because "tests exist" without a deny-case assertion.
- Asking the author for an exploit PoC "to prove it".
- Treating admin-only as safe without checking MCP/CLI/job twins.
- Rubber-stamping warn-mode as "secure enough" forever.
