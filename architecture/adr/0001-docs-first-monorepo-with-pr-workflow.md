# ADR-0001: Docs-first monorepo with a pull-request workflow

- **Status:** Accepted
- **Date:** 2026-09-19
- **Decision area:** Repository structure and change management

## Context

This repository will hold documentation, experiments, eight progressive projects and a flagship platform over roughly 6–8 months. The early phase is documentation only: roadmap, principles, requirements and technical notes. Code starts with P1 around Day 011.

The work needs one place where decisions, requirements, code and evidence stay linked. It also needs a review point: early documentation changes contained broken links and malformed tables that were only caught in review, and some changes are drafted with AI assistance.

## Decision

1. I will keep everything in a single repository (monorepo): docs, projects, experiments and infrastructure, laid out as in the repository map in the [README](../../README.md).
2. I will work docs-first. Structure, roadmap, principles and requirements are merged before the code they govern. Code lands under `projects/` and the other planned directories as each phase starts.
3. Every change reaches `main` through a pull request. Branches are named `docs/...`, `feat/...`, `fix/...` or `chore/...`, and commit messages use the same prefixes.
4. Every pull request uses the [pull request template](../../.github/pull_request_template.md). A pull request that changes an architectural decision adds or supersedes an ADR.

## Alternatives considered

| Option | Why it was not chosen |
|---|---|
| One repository per project | Clean boundaries and independent CI, but it fragments evidence and decisions. Shared contracts would need multi-repo changes, which is heavy for a solo learner. |
| Code first, documentation later | Faster start, but requirements drift and the rationale is lost. It conflicts with [Principles](../../PRINCIPLES.md) 1, 2 and 12. |
| Direct commits to `main` | Least friction, but no review point and no visible diff. Defects go unnoticed. |
| Notes in an external tool or blog | Easy to edit, but not versioned with the code, no review, and links rot. |

## Trade-offs

Gained: one history, traceability from decision to code to evidence, a review point on every change, and an easy place to add CI later.

Given up: pull-request ceremony for a single maintainer, a repository that mixes docs and code as it grows, and unrelated projects sharing one release cadence.

## Consequences

- The pull request template is added with this ADR.
- Branch protection on `main` should be enabled in GitHub settings: require a pull request, block force pushes, block deletion. This is outside the files and is tracked as an action item.
- Path-scoped CI (link check, table check, per-project tests) should be added as code arrives.
- Revisit this decision if the flagship platform needs its own release lifecycle, if repository size or CI time becomes a problem, or if other contributors join with different permissions. The likely change is to move the flagship into its own repository and keep this one as the learning journal.

## Related

- [Principles](../../PRINCIPLES.md)
- [Roadmap](../../ROADMAP.md)
