# Architecture Decision Records

ADRs capture decisions that are **expensive to reverse** — data ownership, settlement
flow, key custody, network selection, and anything that changes the documented API
contract in [../BACKEND-CLIENT-CONTRACT-MAP.md](../BACKEND-CLIENT-CONTRACT-MAP.md).

An ADR is not a design doc and not a meeting note. It records *one* decision, the
alternatives that were rejected, and what the decision costs us. If a PR can be
understood from its description and a code comment, it does not need an ADR. If
reversing it later would require touching every surface or migrating user data, it
does.

## When to write one

Write an ADR in the same PR as the decision when the change:

- introduces or removes a service, datastore, or queue;
- changes who holds keys or funds;
- changes the settlement or refund flow;
- adds, removes, or breaks a public endpoint family;
- introduces a network other than testnet-first;
- replaces a **Mocked** or **Partial** path in [../CAPABILITY-MAP.md](../CAPABILITY-MAP.md)
  with something a user can rely on.

Pure UI changes, bug fixes that restore documented behaviour, and CI changes do not
need one. Label the PR `needs:design` when you are unsure.

## Format

Files are named `NNNN-kebab-case-title.md` and numbered sequentially. Start from the
issue template at `.github/ISSUE_TEMPLATE/adr.yml`.

```markdown
# NNNN. Decision stated as a sentence

- Status: Proposed | Accepted | Superseded by ADR-NNNN | Deprecated
- Date: YYYY-MM-DD
- Surfaces: app/backend, app/contract

## Context
## Decision
## Alternatives considered
## Consequences
```

## Status lifecycle

| Status | Meaning |
|---|---|
| `Proposed` | PR is open; not yet binding. |
| `Accepted` | Merged. Binding — later PRs must follow it or supersede it. |
| `Superseded by ADR-NNNN` | Replaced. Keep the file for history; link the successor. |
| `Deprecated` | The decision no longer applies, but nothing replaced it. |

Never edit an Accepted ADR to change what it says. Supersede it with a new file and
point the old one at the new one, so the reasoning stays readable.

## Index

| # | Title | Status | Surfaces |
|---|---|---|---|
| [0001](./0001-self-custody-no-intermediary-custody.md) | Keep QuickEx self-custodial; no intermediary holds funds | Accepted | all |
| [0002](./0002-testnet-first-mainnet-feature-gated.md) | Testnet-first, with mainnet behaviour behind disabled-by-default feature flags | Accepted | all |
| [0003](./0003-supabase-as-system-of-record.md) | Supabase is the system of record; no additional relational database | Accepted | app/backend |
| [0004](./0004-canonical-status-vocabulary.md) | Live / Partial / Mocked / Experimental is the only status vocabulary | Accepted | docs |
