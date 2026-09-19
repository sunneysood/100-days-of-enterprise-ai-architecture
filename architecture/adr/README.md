# Architecture Decision Records

ADRs capture consequential architecture decisions: what was chosen, what was rejected, and why.

Use an ADR for decisions involving architecture, security, reliability, data ownership, technology selection, deployment, performance or cost.

## How to write one

1. Copy [_TEMPLATE.md](./_TEMPLATE.md) to `NNNN-short-kebab-title.md`, using the next unused number.
2. Fill in every section. Keep it to one or two pages.
3. Set the status to `Proposed`, then `Accepted` when the decision is merged.
4. Add a row to the index below.

## Rules

- Do not rewrite an accepted ADR. If the decision changes, write a new ADR and mark the old one `Superseded by ADR-NNNN`.
- Link ADRs from the project requirements, the day entry, or the pull request that acts on them.
- Record rejected alternatives. They are the most useful part in six months.

## Statuses

`Proposed` · `Accepted` · `Deprecated` · `Superseded by ADR-NNNN`

## Index

| ADR | Title | Status | Date |
|---|---|---|---|
| [0001](./0001-docs-first-monorepo-with-pr-workflow.md) | Docs-first monorepo with a pull-request workflow | Accepted | 2026-09-19 |
