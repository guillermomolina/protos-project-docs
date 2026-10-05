# TEST002 legacy Java/JUnit inventory baseline

Date: 2026-10-05

Owner: `TEST002` / `guillermomolina/protos#538`

This is snapshot evidence, not a replacement for live GitHub state and not a
test-ownership registry.

## Exact identities

- Audited Protos HEAD:
  `53c54bb952354cae61e72db68c4ff8f509c827ed`
  (`TEST009-O: bound shared host leaves exposed by M5 native-body PIC`).
- Current test-placement policy boundary:
  `e99d0baba547ac41b3894f32ddca450172ee1f8b`
  (`TEST001-I: retire superseded test ownership infrastructure`, 2026-09-16).

The boundary commit introduced the current placement rule: prefer ordinary
Protos/Test Tool coverage for faithfully expressible observable behavior; retain
Java/JUnit for materially distinct Java, Truffle, compiler/backend, runtime,
scheduler, native/host or independent bootstrap evidence. It also explicitly
routes reconciliation of pre-policy legacy Java/JUnit tests to TEST002.

## Reproducible legacy universe

The legacy universe is the path intersection of
`src/test/java/**/*.java` at the policy boundary and the audited current HEAD.

| Population | Files |
| --- | ---: |
| Java tests at policy boundary | 430 |
| Java tests at current HEAD | 527 |
| Legacy survivors | 393 |
| Legacy files removed since boundary | 37 |
| Java tests added after boundary | 134 |

Surviving legacy files by top-level package:

| Package | Files |
| --- | ---: |
| `execution` | 259 |
| `runtime` | 59 |
| `parser` | 18 |
| `semantic` | 17 |
| `cli` | 14 |
| `conformance` | 7 |
| `lsp` | 7 |
| `analysis` | 5 |
| `lexer` | 4 |
| `documentation` | 3 |
| **Total** | **393** |

The 37 removed legacy files were: execution 29, semantic 4, parser 2, analysis 1,
conformance 1.

The 134 post-policy Java files intentionally excluded from the initial legacy
target were: execution 111, runtime 11, cli 5, embedding 2, and one each in
analysis, documentation, lsp, parser and semantic.

## Existing local reconciliation evidence

Several survivors have acquired explicit Java-side rationale during later work.
Examples include:

- `ProtosNonLocalReturnTest`: exact host ReturnHome/control transfer;
- `ProtosRepresentedValueLookupTest`: exact lookup-home/physical prototype identity;
- `ProtosStandardStringProtocolTest`: frozen Prelude/bootstrap topology;
- `ProtosIdentityMapConformanceTest`: representation and exact bootstrap binding;
- `ProtosStandardFutureProtocolTest`: evaluator suspension/scheduler state;
- `ProtosStandardEnvironmentProtocolTest`: native representability/invalid Unicode;
- `ProtosArrayConformanceCompletionTest`: represented lifecycle state and
  host-visible callback transfer while ordinary Array semantics live in Protos.

These are evidence for the audit, not automatic classification of whole packages.

## First bounded classification

Four legacy Java classes consume fifteen real negative `.protos` fixtures under
`protos/tests/parser/`:

- `ProtosGrammarSurfaceNegativeFixtureTest` (LM008-B1);
- `ProtosBindingSurfaceNegativeFixtureTest` (LM008-B2);
- `ProtosCallableSurfaceNegativeFixtureTest` (LM008-B3);
- `ProtosReceiverSuperSurfaceNegativeFixtureTest` (LM008-B4).

Their contract is to prove that normatively invalid source is rejected by the
lexer/parser before a guest Process can execute. The Java boundary is therefore
necessary observation/bootstrap evidence; the `.protos` files are already the
source fixtures.

Classification: **`KEEP_JAVA_BOOTSTRAP`**.

Moving these assertions into an ordinary guest Test would be circular or
impossible: successful guest execution is precisely what must not occur.

## Conclusion and next slice

TEST002 is **not closed**. No product/test change is claimed by this baseline.

The 259 surviving legacy files in `execution` are a mixed cohort containing
source-visible behavior, representation/bootstrap checks, scheduler/runtime
evidence, Test Tool infrastructure, bytecode/compiler coverage and host
integration. The next bounded slice is a read-only contract audit:

**TEST002-A2 — classify the surviving legacy execution cohort by real contract**

For each coherent family, choose exactly one of
`MIGRATE_TO_PROTOS`, `KEEP_JAVA_HOST_RUNTIME`,
`KEEP_JAVA_BOOTSTRAP`, or `SPLIT`. Do not infer equivalence from filenames,
do not revive the retired ownership registry, and identify the first safe
implementation batch rather than designing a repository-wide mega-patch.
