# Governance

This file controls the playbook itself — how rules are maintained, changed, and communicated.

## Rule levels

- **MUST**: mandatory; a deviation requires an ADR with reason, owner, and review trigger.
- **SHOULD**: default practice; document why it is not used.
- **MAY**: optional, requirement-driven.

Every rule in the playbook uses exactly one of these levels. Ambiguous phrasing such as "consider" or "try to" is not permitted in a rule statement.

## Playbook ownership

| Role | Responsibility |
|---|---|
| Playbook owner | Approves changes to MUST-level rules and breaking structural changes |
| Section owner | Reviews and approves changes to their section; named per section README |
| Contributor | Any team member may propose a change via a pull request with rationale |

When no section owner is named, the playbook owner acts as reviewer.

## How to change a rule

1. Open a pull request with the proposed change and a written rationale.
2. For a MUST-level rule change, tag the playbook owner for approval.
3. For a SHOULD or MAY change, the section owner approves.
4. Record the change in the changelog below with version, date, section, change type, and summary.
5. Breaking changes to a MUST rule trigger a project-impact sweep (see below).

A "breaking change" is any edit that removes or weakens a mandatory obligation, renames a file or section in a way that breaks existing cross-links, or changes how a template field is interpreted.

## Changelog

| Version | Date | Section | Change type | Summary |
|---|---|---|---|---|
| 1.0 | 2026-11-01 | All | Initial release | First complete version of the playbook |

Add a row for every change after v1.0. Use change types: `added`, `changed`, `deprecated`, `removed`, `fixed`.

## Communication to in-flight projects

When a MUST-level rule changes:

1. Publish a notice to all active project technical owners within 5 business days.
2. State: what changed, why, whether existing implementations are compliant, and the deadline to align.
3. Projects that cannot align by the deadline record an ADR exception with owner and expiry.
4. Playbook owner reviews outstanding exceptions quarterly.

SHOULD and MAY changes are communicated in the next regular team sync; no deadline is imposed on in-flight projects.

## Deprecated rules

A deprecated rule is not deleted immediately. It is:

1. Marked `[DEPRECATED as of vX.Y — use <replacement> instead]` in its file.
2. Listed in the changelog as `deprecated`.
3. Retained for a documented migration window (default: two release cycles or 90 days, whichever is longer).
4. Removed only after the migration window and a final owner confirmation.

## Exception process

A MUST-rule exception must include:

- the exact rule being excepted;
- the business or technical reason;
- compensating controls in place;
- named approver (technical owner or above);
- expiry date and review trigger; and
- a link from the ADR to this governance file.

Expired exceptions automatically re-open the obligation. No individual developer may silently override a mandatory rule.
