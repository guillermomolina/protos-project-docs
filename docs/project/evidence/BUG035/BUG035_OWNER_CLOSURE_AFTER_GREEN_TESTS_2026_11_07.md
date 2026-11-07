# BUG035 — final owner-directed closure after green regression gates

**Date:** 2026-11-07. **Live issue:** [BUG035 #1081](https://github.com/guillermomolina/protos/issues/1081). **Consumer application:** [I153 #1068](https://github.com/guillermomolina/protos/issues/1068).

## Exact product publication and validation

- **PROTOS_REVISION:** [`bfb276f9b76c663d3ef1fb68823e13a22ede136c`](https://github.com/guillermomolina/protos/commit/bfb276f9b76c663d3ef1fb68823e13a22ede136c).
- **VERSION:** `0.3.452-SNAPSHOT`, verified in `pom.xml` and `CHANGELOG.md` at the product revision.
- **DELTA:** exact canonical frozen hidden Bytes prototype shared as a standard anchor; normal mutable Bytes instances copied with their octets, local slots, parent, alias/cycle relations and mutation state; non-transferable graphs still rejected. The published commit includes Java regressions and 12 Actor + 9 Group guest cases; no normative specification changed.
- **MAINTAINER REPORTED:** focal Java tests, `make test-protos`, `make test`, `make check` all PASS; `git diff --check` clean; commit pushed.
- **INDEPENDENT CHECK:** the GitHub product commit, its changed file list, version and matching changelog were read. No tests or RealWorld application were independently executed by the documentation publisher.

## Owner's closure instruction and correction

After the initial [BUG035 publication evidence snapshot](BUG035_CANONICAL_BYTES_ACTOR_TRANSFER_PUBLICATION_2026_11_07.md), the project owner explicitly rejected leaving this completed regression repair in `status:review` pending the RealWorld application run: a bug with the required green regression/integrated tests is to be **closed as completed**.

The original publication snapshot is retained unchanged to preserve its historical provenance, but **its claim that a RealWorld two-Worker smoke is a prerequisite to closing BUG035 is superseded by this owner instruction**. Its `BUG035_ISSUE_CLOSURE=PENDING_REALWORLD_ACCEPTANCE` marker is a historical pre-closure assessment, not the final status.

This bounded BUG035 repair has direct tests of the affected Actor/Group standard Bytes transfer and the full Protos suite. The separate RealWorld two-Worker HTTP loopback check is still unreported and stays an application-level check within [I153 #1068](https://github.com/guillermomolina/protos/issues/1068), **not a reason to hold or reopen BUG035**. Do not claim the RealWorld application smoke, HTTP endpoints or official RealWorld Hurl tests passed.

The current authoritative completed state is the live BUG035 GitHub issue. This record documents the owner's closure direction and the basis for that state, not a new semantic decision.

```text
WORK_ITEM=BUG035
ISSUE=guillermomolina/protos#1081
PROTOS_REVISION=bfb276f9b76c663d3ef1fb68823e13a22ede136c
VERSION=0.3.452-SNAPSHOT
HUMAN_FOCAL_TESTS=PASS_REPORTED
HUMAN_MAKE_TEST_PROTOS=PASS_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
HUMAN_DIFF_CHECK=CLEAN_REPORTED
PRODUCT_PUBLICATION=GITHUB_VERIFIED
OWNER_CLOSURE_DIRECTION=COMPLETED
REALWORLD_TWO_WORKER_SMOKE=UNREPORTED_OWNED_BY_I153
SPEC_CHANGE=NONE
```
