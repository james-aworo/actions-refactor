# Actions Refactor Skill

## Short description
This repository contains an agent skill definition in `SKILL.md` for refactoring Actions controller flows into a dedicated `domains/actions` structure with strict safety gates.

## Purpose
- Provide a single, explicit refactor workflow for Actions-related code moves.
- Keep refactors behavior-preserving, reviewable, and reversible.
- Enforce safer execution paths through stop gates, ownership rules, and verification steps.
- Centralize maintenance in one skill file to reduce drift.

## Repository structure
```text
.
├── SKILL.md      # Primary skill definition (behavior, workflow, guardrails, output format)
└── README.md     # Repository overview and contributor guidance
```

## How the `SKILL.md` file works
`SKILL.md` is the source of truth for skill behavior. It defines:
- Skill metadata (name, version, triggers, allowed tools)
- Scope and non-goals
- Controller-first execution workflow
- Ownership and dependency classification rules
- Adapter/use-case boundary rules
- Audit and patch report formats
- Stop conditions and final response format

In practice, the skill is designed to process one selected controller flow per run and stop when risk or scope conditions are not met.

## Action/refactor design notes
- **Narrow scope by design:** refactor one Actions controller flow at a time.
- **Structure over cleanup:** package relocation and boundary clarity take priority over architectural redesign.
- **Safe boundaries:** controllers call Actions services; cross-domain calls go through thin adapters/use-cases.
- **No hidden logic in wrappers:** adapters and use-cases should be forwarding-only.
- **Guardrails first:** explicit stop gates prevent risky or ambiguous refactors.
- **Auditability:** JSON/JSONL outputs are specified for run state, audits, review tasks, and patch reporting.

## Development workflow
1. Read `SKILL.md` fully before making changes.
2. Update only the section(s) needed for your change.
3. Keep behavior explicit; avoid broad rewrites.
4. Preserve or strengthen stop gates and safety checks.
5. Update version and behavior notes when scope changes.
6. Keep diffs small and focused for review.

## Testing or validation guidance
For this repository, validation is content-focused:
- Check markdown clarity and consistency.
- Ensure section references and paths are accurate.
- Verify workflow ordering and rule consistency after edits.
- Confirm no contradictory instructions were introduced.

Skill-level validation guidance (for downstream execution) is documented in `SKILL.md` under verification and diff-safety sections.

## Contribution guidelines
- Keep changes practical and developer-focused.
- Do not weaken safety guardrails to satisfy edge cases.
- Do not invent behavior not documented in `SKILL.md`.
- Prefer adding explicit TODO notes where requirements are unclear.
- Keep instructions deterministic and review-friendly.

TODO: Add contribution process details (branching, review approvals, release/versioning) if the team standardizes them.

## License

Copyright (c) 2026 Prewave. All rights reserved.

This repository is proprietary and confidential. It is intended only for authorized Prewave employees, contractors, or agents, and only for approved company purposes.

Do not share, publish, copy to external repositories, or use outside Prewave without prior written approval.
