# Fail-Closed Review

Claude Code skill: `fail-closed-review`

## What

PR review rubric for governance/authz products. Default: anything not proven deny is fail-open. Produces structured findings only — never exploit PoCs.

## When to use

Reviewing authz, policy, RLS, MCP/API auth, or admin surfaces; Rigor role in Faber↔Rigor handoffs.

## Install

Copy this folder into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/fail-closed-review
cp SKILL.md ~/.claude/skills/fail-closed-review/
```

Claude Code loads `SKILL.md` from `~/.claude/skills/<name>/`.

## Sibling skills

`adversarial-qa`, `verified-delivery`, `contract-and-compat`, `migration-and-data-safety`, `reachability-audit`, `guards-that-scan`, `handoff-faber-rigor`, `code-that-holds`

Proposed repos: see [Vorxeo](https://github.com/Vorxeo) `skill-*` packs.

## License

MIT — Copyright (c) 2026 Vorxeo. See [LICENSE](./LICENSE).
