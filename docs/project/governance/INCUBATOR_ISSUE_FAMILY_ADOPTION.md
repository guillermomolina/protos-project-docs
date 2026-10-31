# Incubator issue family — adoption and validation evidence

This is a non-normative project-governance record. The authoritative instructions
are in [Protos `AGENTS.md`](https://github.com/guillermomolina/protos/blob/73bd9fe9/AGENTS.md),
[`AGENTS.work/INCUBATOR.md`](https://github.com/guillermomolina/protos/blob/73bd9fe9/AGENTS.work/INCUBATOR.md),
and [`AGENTS.work/COORDINATION.md`](https://github.com/guillermomolina/protos/blob/73bd9fe9/AGENTS.work/COORDINATION.md).
Live Issue identity and state remain authoritative in GitHub, not in this record.

## Adoption — 2026-10-31

The project owner explicitly approved a lightweight `INCxxx` formal work family
for speculative ideas worth tracking before they become design decisions or
implementable work.

- `INCxxx` is orthogonal to Issue Status, Priority, assignee, and hierarchy.
- The family is represented by one formal identifier and `family:INC`, under
  the existing collision-safe allocation and intake rules.
- Incubation does not ratify semantics, durable architecture, public APIs,
  or implementation; those authority gates remain with `Dxxx`/`PLATxxx`
  and the applicable implementation family.
- Community discussion remains in Discussions until a maintainer explicitly
  adopts an idea into the formal Issue system.
- Promotion creates a separately identified formal work item, linked to the
  incubated idea; it does not silently convert the idea into an approved design.
- `AGENTS.md` only selects the new work-type document. Incubator-specific
  rules stay in `AGENTS.work/INCUBATOR.md`; cross-family coordination stays in
  `AGENTS.work/COORDINATION.md`.

## Published product change

**Exact Protos revision:** [`73bd9fe9`](https://github.com/guillermomolina/protos/commit/73bd9fe9)
(`INC: introduce the INCxxx incubator issue family`), published on `main`.

The commit changed seven files:

- `AGENTS.md`
- `AGENTS.work/INCUBATOR.md` (new)
- `AGENTS.work/COORDINATION.md`
- `scripts/issue_intake.py`
- `.github/ISSUE_TEMPLATE/incubator.yml` (new)
- `.github/ISSUE_TEMPLATE/work.yml`
- `.github/ISSUE_TEMPLATE/community.yml`

No normative `spec/` files, runtime implementation, new Project fields,
or new workflow were part of the change.

## Human-executed validation evidence

The maintainer reported the following local results before publication:

| Check | Reported outcome |
| --- | --- |
| `git diff --cached --check` | PASS; no diagnostics |
| `python3 -m py_compile scripts/issue_intake.py` | PASS |
| `python3 scripts/issue_intake.py --self-test` | PASS; all existing and new checks |
| `INC_FAMILY_RECONCILIATION` | PASS |
| `UNTRUSTED_INC_CANDIDATE_NO_PROMOTION` | PASS |
| `INC_IDENTIFIER_UNIQUENESS_FAIL_CLOSED` | PASS |
| `git diff --cached --name-only` | Exactly the seven expected paths |
| `make test` | PASS, maintainer-reported |
| `make check` | PASS, maintainer-reported |

**Validation limitation:** The optional Python/YAML parsing check did **not**
run: the local Python environment raised `ModuleNotFoundError: No module named 'yaml'`
before parsing any issue template. This must not be represented as a YAML check
passing. The reported passing test/build results do not establish that the
unexecuted PyYAML check passed.

## First incubated idea: native Issue continuity

The existing [Issue #994](https://github.com/guillermomolina/protos/issues/994)
was reclassified from its historical identifier `D226` to:

**`INC001 — Universal UI widgets across TUI, WUI and GUI`**

Its live Issue title/body were updated to make exploratory status explicit,
preserve the original motivation and technical context, and remove premature
mandatory comparative research or architectural ratification requirements.
The Issue remains open with `status:inbox`, `family:INC`, no assignee, no
assigned Priority, and no milestone. The historical `D226` identifier remains
consumed and must not be reused. No separate Issue was created for the migration.

The issue itself is the live source of truth for any subsequent changes. This
record does not grant approval for a UI API, selected backend, architectural
decision, or implementation.

## Publication boundaries

This record captures a completed governance change and its reported validation
evidence, **not** a design ratification or a required evidence file for every
future `INCxxx` idea. Other incubator ideas should normally retain their
exploration and history in their GitHub Issues, as defined by the published
incubator policy.
