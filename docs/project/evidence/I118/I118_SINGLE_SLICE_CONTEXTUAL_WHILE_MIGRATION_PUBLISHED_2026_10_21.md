# I118 — Published single-slice contextual while migration

**Date:** 2026-10-21  
**Formal work:** [I118 / #943](https://github.com/guillermomolina/protos/issues/943), family I, **one implementation slice**  
**Product repository:** `guillermomolina/protos`  
**Exact published product revision:** [`2a86c48c403b7992f424a42e6f53123898de6cb8`](https://github.com/guillermomolina/protos/commit/2a86c48c403b7992f424a42e6f53123898de6cb8)  
**Commit message:** `I118: adopt contextual while in first-party Protos sources`  
**Implementation version:** `0.3.395-SNAPSHOT`  
**Normative specification revision:** `0.1.465` (unchanged; source-style migration only)  
**Prerequisite:** [I117-B product `7a122cdeeed91ca31690e2d39412dadad3518f72`](https://github.com/guillermomolina/protos/commit/7a122cdeeed91ca31690e2d39412dadad3518f72) implementing D210-Q2.  
**Plan/handoff:** [single-slice I118 work record](../../work/I118/I118_A_EDITOR_AND_HUMAN_EXECUTOR_HANDOFF_2026_10_21.md) (historical filename retains `_A`; content supersedes phased plan).

## Delivered

I118 applies D210-Q2's approved ordinary-source idiom `while(CONDITION) { BODY }` to equivalent previously explicit `(() => CONDITION).whileTrue() { BODY }` and `.whileTrue(() => { BODY })` spellings in first-party source. No new `while` keyword, parser, canonicalizer, runtime, message protocol, Truffle Bytecode DSL or other executable-language semantics are introduced. Normal result remains governed by the ordinary `whileTrue` selector. The source migration was implemented in **one substantive patch and one product publication** as requested by the owner.

**Reported source migration:** **311 loops across 53 Protos source files**:

| Protos source tree | Converted loops | Changed source files in published commit |
|---|---:|---:|
| `protos/lib/` | 198 | 17 |
| `protos/tools/` | 58 | 13 |
| `protos/tests/` | 54 | 22 |
| `protos/examples/` | 1 | 1 |
| `protos/benchmarks/` | 0 | 0 |
| `protos/tutorials/` | 0 | 0 |
| **Total** | **311** | **53** |

The published commit changes **56 paths**: 53 Protos sources, one Java test fixture (`src/test/java/com/guillermomolina/protos/execution/tools/test/discovery/ProtosTestToolFileSelectionMainAdoptionTest.java`) to match the new source spelling in the Test Tool, plus `pom.xml` and `CHANGELOG.md`. This is reconciled with the actual published commit file list; the additional Java source-shape assertion is necessary because `protos/tools/test/Main.protos` now uses `while(suiteIndex < executionSuites.size()) {`. Only syntax forms and their closing `})` / `}` were edited in Protos source.

**Deliberate exclusions:**

1. Protocol conformance under `protos/tests/conformance/control/while-*.protos` retains explicit `.whileTrue`, including `while-contextual-syntax.protos` which compares both forms; published unchanged.
2. A pre-existing condition Closure or body passed by reference remains an ordinary `.whileTrue` invocation and is not converted by name.
3. Multi-expression condition Closures in `protos/lib/network/IpAddresses.protos` and `protos/lib/network/IpEndpoints.protos` retain their explicit form.
4. All **six** canonical `protos/benchmarks/concurrency/*.protos` PERF001-F workload sources remain byte-for-byte equal to the pre-I118 HEAD, preserving the measurement corpus; **9 otherwise plausible conversions** were explicitly reverted before publication.
5. Java/Python/shell embedded `whileTrue` protocol/fixture expressions were not globally migrated; the one Java test which asserted the changed first-party Test Tool source spelling was adjusted.
6. Normative specification and historical changelog passages that teach/record the ordinary `whileTrue` protocol were preserved.

The adopted spelling is a source-style choice established by `protos/AGENTS.md` and `docs/guide/SOURCE_STYLE.md`, not a new semantic decision. Any performance change would require separate measurement and is **not** claimed here.

## Human validation and published evidence

The owner explicitly reported, after all final edits and the single product push:

- `git diff --check`: **clean**;
- local tests: **all PASS**;
- `I118`: **pushed**.

The published `CHANGELOG.md` also records **`make check` PASS** and **`make test` PASS**. These are **maintainer-reported validation results**, not tests rerun by the documentation author or independent CI verification. No raw logs were reproduced here.

Read-only GitHub verification established that commit `2a86c48c403b7992f424a42e6f53123898de6cb8` is published, `pom.xml` has version `0.3.395-SNAPSHOT`, `CHANGELOG.md` has an I118 entry with the preserved exceptions and reported gates, and the global specification changelog still leads with `0.1.465`.

## Closure assessment

- **Product publication:** PASS — exact stable commit and implementation version verified.
- **Source migration:** PASS as published — 311 equivalent cases across 53 first-party Protos files; deliberate protocol/benchmark/non-equivalent exceptions documented.
- **Semantic authority:** unchanged D210-Q2 and standard `whileTrue` protocol.
- **Validation:** maintainer-reported PASS; no independently executed tests.
- **Durable publication:** this record is the revision-coupled evidence for closing [I118 / #943](https://github.com/guillermomolina/protos/issues/943).

No follow-up slice is required to complete I118. Any future unrelated style issues or actual semantic regressions are separate work.
