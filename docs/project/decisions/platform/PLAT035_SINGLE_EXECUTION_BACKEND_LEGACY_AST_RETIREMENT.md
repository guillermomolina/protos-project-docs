# PLAT035 — Single execution backend and legacy Truffle AST retirement

Status: `RATIFIED`

Selected architecture: Candidate C — one target executable backend, with the
legacy executable Truffle AST retained only as bounded migration scaffolding
until its audited dependency inventory reaches zero.

Approval: explicit project-owner approval on 2026-09-17 after the PLAT035
comparative decision packet and invariant/delta consistency review.

Decision Issue: `guillermomolina/protos#551`

Triggered by: `AUD012` / `guillermomolina/protos#541`

Research / ratification source baseline:
`5529fa515016f6a44fb15934c4691e162c417d73`

Primary prior authority: `PLAT014` — C-prime (`C′`) stackful continuation
composition over Truffle Bytecode DSL continuations.

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

## Boundary

PLAT035 is a durable JVM/Truffle implementation architecture decision. It does
not change observable Protos syntax, values, object semantics, Task/Future
semantics, Actor/Process semantics, error/control behavior, ordering, Standard
Library behavior, or public compatibility promises.

Another conforming implementation of Protos does not need to use Truffle or the
Truffle Bytecode DSL. Bytecode DSL remains backend machinery, not Protos
semantic authority.

Surface AST and Canonical AST are not retirement targets of PLAT035. The
retirement target is specifically the duplicate executable backend reached after
canonicalization through `CanonicalToTruffleLowerer` and the executable
`ProtosExpressionNode` tree.

## Ratified invariant

The project owner explicitly selected the following end state:

```text
TARGET_EXECUTION_BACKEND = TRUFFLE_BYTECODE_DSL_ONLY

source
  -> Surface AST
  -> Canonical AST
  -> Truffle Bytecode DSL
  -> execution

CanonicalToTruffleLowerer = LEGACY_TRANSITIONAL
ProtosExpressionNode execution tree = LEGACY_TRANSITIONAL
NEW_DEPENDENCIES_ON_LEGACY_BACKEND = FORBIDDEN
END_STATE = LEGACY_EXECUTION_BACKEND_REMOVED
```

This is an architecture invariant, not permission for unsafe mechanical
deletion. Existing legacy-backend consumers remain migration blockers until an
equivalent Bytecode-backed route, retained test evidence, or justified removal
has been established.

## Current implementation evidence

At the ratification baseline, `ProtosSourceCompiler` still contains two
post-canonical execution routes:

```text
Surface AST
    -> Canonical AST
       |-> CanonicalToTruffleLowerer -> ProtosExpressionNode -> CallTarget
       `-> CanonicalToBytecodeLowerer -> Bytecode DSL -> CallTarget
```

The public Truffle language entry `ProtosLanguage.parse(...)` already calls
`ProtosSourceCompiler.compileBytecode(...)`. Normal hosted language execution is
therefore already centered on the Bytecode DSL path.

AUD012 established that remaining direct Java guest-entry uses are heterogeneous:
ordinary semantic harnesses, Java/runtime/bootstrap tests, compiler/backend tests,
compile-only assertions, deliberately unhosted staging paths, and historical
helpers whose test ownership still requires TEST002 classification.

That heterogeneity justifies an ordered migration. It does not justify two
permanent executable semantic backends.

## Relationship to PLAT014

PLAT014 already ratified C′ continuation composition over the Truffle Bytecode
DSL and explicitly rejected a permanent AST continuation framework as duplicate
infrastructure. It also rejected a temporary AST continuation hybrid after the
Bytecode feasibility spike proved direct C′ viability.

PLAT014 deliberately deferred exact migration slices from the previous AST/replay
backend. PLAT035 resolves one of those deferred architecture questions by making
the retirement direction explicit:

- Bytecode DSL is the single target executable backend;
- the legacy executable AST is not a permanent alternative backend;
- no second continuation or execution authority may be reintroduced under a
  test/bootstrap exception.

PLAT035 does not reopen PLAT014's suspension, continuation, ownership, cleanup,
control-transfer, instrumentation, location-identity, or carrier semantics.

## Comparative evidence

The decision packet reviewed the relevant architecture space rather than treating
the preferred end state as self-proving.

### GraalPy

GraalPy is the closest Truffle migration precedent. It introduced a bytecode
interpreter alongside the previous AST interpreter during migration, then made
the bytecode interpreter the default. The coexistence phase provided validation
and migration safety; it was not evidence for preserving two permanent semantic
execution authorities.

### GraalJS, TruffleRuby and Apple Pkl

These provide the necessary counterexample to an incorrect argument: AST
interpreters are not intrinsically inferior. A Truffle AST can be a sound,
optimizing canonical backend.

Their relevance to PLAT035 is therefore corrective. The issue is not
"AST bad / bytecode good"; it is whether Protos should permanently maintain two
independent executable realizations of the same Canonical AST semantics after
Bytecode DSL has already become the selected production substrate.

### Espresso and SimpleLanguage

These reinforce that a bytecode-oriented or Bytecode-DSL-backed execution
architecture may still use specialized Truffle nodes internally. "Single
backend" does not mean "no Node classes"; it means one authoritative executable
pipeline rather than two complete post-canonical lowering universes.

### V8

V8 provides a strong non-Truffle retirement precedent. During migration, old and
new pipelines coexisted. Once Ignition + TurboFan became the universal execution
pipeline, Full-codegen/Crankshaft were retired, simplifying architecture and
removing coordination machinery whose only purpose was serving parallel compiler
pipelines.

### Frontend/backend separation

Compiler pipelines routinely retain syntax and semantic trees while converging on
one executable backend representation. PLAT035 follows the same separation:

```text
Surface AST -> Canonical AST -> one executable backend
```

The value of Surface and Canonical ASTs is therefore unaffected by this decision.

## Candidate comparison

The decision packet compared four materially distinct architectures:

- **A — permanent dual executable backends**;
- **B — Bytecode production plus permanent legacy Java/testing backend**;
- **C — Bytecode DSL as sole target backend with bounded migration scaffolding**;
- **D — immediate hard deletion and repair fallout afterward**.

Candidate C was selected.

Candidate A was rejected because it permanently duplicates semantic lowering,
tooling adaptation, test obligations and future runtime changes without a current
requirement for two executable authorities.

Candidate B was rejected because "tests need it" would become a permanent
architecture institution. Java ownership can be preserved without preserving a
second executable backend.

Candidate D was rejected because AUD012 has already demonstrated heterogeneous
legacy consumers. Immediate deletion would risk discarding legitimate bootstrap,
runtime and component evidence before an equivalent Bytecode-backed route exists.

Candidate C is the smallest safe architecture: one intended backend, plus only
the temporary compatibility required by identified migration blockers.

## Ratified retirement contract

### 1. Canonical AST is the backend-independent boundary

Canonical AST remains the last backend-independent executable semantic
representation for this architecture. Surface and Canonical ASTs remain available
for parsing, canonicalization, static tooling, inspection and other already-valid
purposes unless separately changed by future decisions.

### 2. All source-backed executable paths converge on Bytecode DSL

Production, bootstrap, Java-owned tests and deliberately unhosted execution may
have different hosting APIs and lifecycle boundaries, but executable Protos source
must ultimately lower through the Bytecode DSL backend.

A different host boundary is not permission for a different semantic backend.

### 3. The legacy executable AST has migration status only

Existing `CanonicalToTruffleLowerer` / executable `ProtosExpressionNode`
dependencies may remain temporarily only while they correspond to concrete,
audited migration blockers.

Every retained blocker must have:

- an identified occurrence or bounded family;
- a remediation owner;
- a replacement/removal route; and
- evidence before deletion.

No new ordinary dependency on the legacy backend may be introduced.

### 4. Java-owned does not mean legacy-backend-owned

A Java/JUnit test may remain Java-owned when it tests Java, Truffle, runtime,
bootstrap, host integration, lifecycle, internal representation or another
legitimate Java-side concern.

If that test executes guest Protos source, its guest execution must migrate to an
explicit Bytecode-backed Java execution boundary rather than preserving the old
backend for convenience.

TEST002 remains authoritative for whether observable semantic tests should move
from Java/JUnit to ordinary Protos/TOOL002 tests. PLAT035 does not pre-decide
TEST002 ownership.

### 5. `ProtosSourceCompiler.compile(...)` cannot remain a permanent second-backend API

During migration the API may be adapted, split, redirected or replaced as
mechanically necessary.

At the end state it must not provide access to an independent executable
`CanonicalToTruffleLowerer` backend. It either routes through the canonical
Bytecode backend under an appropriate hosting boundary or ceases to exist in its
current role.

Exact API spelling and local class decomposition are implementation details unless
they expose a new durable architecture choice.

### 6. Retain frontend/compiler tests only where the subject remains real

Parser, Surface AST, canonicalization and Bytecode-lowering tests remain valid
where those are their actual subjects.

Tests whose only durable purpose is to preserve behavior of the obsolete
executable AST backend are retirement candidates after equivalent semantic or
backend evidence exists where required.

Deleting an obsolete-backend test must not be used to hide missing behavior
coverage.

### 7. Fail closed against regression

Repository/CI guidance must prevent new raw legacy executable-backend dependencies
from silently appearing.

The guard must target dependency on the retiring backend, not ban all Java tests,
all compiler construction, all Truffle nodes, or all direct internal execution
mechanics.

Bounded retirement implementation may temporarily touch legacy components while
removing or routing them. Such work must not normalize new consumers.

### 8. Tooling contracts survive backend retirement

PLAT005, PLAT013, PLAT015, PLAT026 and PLAT034 remain authoritative for
instrumentation, source, debugger, root/tag and generic-tool behavior.

The retirement is invalid if it achieves one backend by weakening already-ratified
tooling behavior.

### 9. Bytecode DSL stays behind an implementation boundary

The Truffle Bytecode DSL is platform-specific and its APIs may evolve. Protos
semantic/frontend authority must therefore remain independent of Bytecode DSL
Java types.

This preserves the ability of another Protos implementation, or a future JVM
backend redesign, to realize the same Protos semantics without adopting current
Truffle implementation types as language concepts.

### 10. Removal requires zero audited dependencies and replacement evidence

Physical removal of the legacy executable backend is authorized only after the
migration inventory demonstrates that no legitimate consumer still requires it
and retained behavior is covered on the target architecture.

The final retirement evidence must include a repository-wide rescan, focal
validation for migrated surfaces, authoritative full validation, and explicit
confirmation that no unclassified legacy executable dependency remains.

## Invariant / delta consistency review

The mandatory pre-ratification consistency review produced the following result.

### Owner-approved PLAT035 invariant

```text
TARGET_EXECUTION_BACKEND = TRUFFLE_BYTECODE_DSL_ONLY
LEGACY_AST = TRANSITIONAL
NEW_LEGACY_DEPENDENCIES = FORBIDDEN
END_STATE = LEGACY_EXECUTION_BACKEND_REMOVED
```

Candidate C preserves this invariant exactly.

### PLAT014 continuation authority

**Preserved.** Candidate C strengthens convergence on the already-ratified C′
Bytecode continuation substrate. It does not alter suspension, resume, Task
ownership, carrier reuse, cleanup, Error, cancellation or non-local-return
semantics.

### Surface / Canonical AST authority

**Preserved.** Candidate C removes only the duplicate executable backend. It does
not classify frontend or Canonical AST representations as obsolete.

### Java test ownership

**Preserved.** Candidate C does not force every retained test into Protos. Java
ownership and guest execution backend are treated as separate dimensions.

### TEST002 authority

**Preserved.** PLAT035 does not classify semantic tests as migration candidates or
retained Java tests beyond the backend requirement. TEST002 keeps that ownership.

### Tooling architecture

**Preserved.** Existing tooling decisions remain required invariants during
migration.

### New consequence introduced by PLAT035

The only newly ratified durable constraint is that an intentional direct/unhosted
Java execution path is no longer sufficient justification for preserving the
legacy executable AST. Such paths must eventually receive a Bytecode-backed
execution boundary.

This consequence was explicitly surfaced in the decision packet and explicitly
approved by the project owner on 2026-09-17.

### Consistency result

```text
OWNER_INVARIANT=PASS
PLAT014=PASS
SURFACE_AST=UNCHANGED
CANONICAL_AST=UNCHANGED
TEST002_AUTHORITY=UNCHANGED
JAVA_TEST_OWNERSHIP=UNCHANGED
TOOLING_CONTRACTS=UNCHANGED
PROTOS_VISIBLE_SEMANTICS=UNCHANGED
NEW_DURABLE_DELTA=SINGLE_EXECUTION_BACKEND_ONLY
INVARIANT_DELTA_CONSISTENCY=PASS
```

## Consequences for AUD012

PLAT035 changes AUD012 from an open-ended legitimacy audit into a retirement
routing audit.

An AUD012 occurrence may still be classified as intentionally Java-owned,
compile-only, runtime-internal or deliberately unhosted, but no classification
may imply permanent entitlement to the legacy executable AST backend.

For every executable legacy dependency AUD012 must now determine:

```text
legacy occurrence
    -> migrate to Bytecode-backed Java boundary
    -> migrate semantic evidence under TEST002
    -> retain only non-executable/frontend evidence
    -> remove with obsolete backend
    -> route a newly exposed semantic/platform blocker through its proper gate
```

The remaining count of legacy executable consumers is therefore a retirement
backlog, not a permanent exception list.

## Implementation sequencing

Ratification authorizes bounded implementation work consistent with this
contract. A sensible sequence is:

1. define the explicit Bytecode-backed Java/unhosted execution boundary;
2. establish the fail-closed new-dependency guard;
3. migrate Java-owned source-executing harnesses;
4. route ordinary semantic tests through TEST002/canonical test execution as
   appropriate;
5. migrate bootstrap/runtime fallback execution;
6. remove tests/helpers whose only subject was the obsolete executable backend;
7. remove `CanonicalToTruffleLowerer` and executable `ProtosExpressionNode`
   machinery once the dependency inventory reaches zero;
8. run final repository-wide rescan and authoritative full validation.

The exact slice decomposition is implementation work. If migration exposes a new
observable semantic choice or a materially different platform architecture, that
question crosses the normal Dxxx/PLATxxx approval gate before dependent work
continues.

## Closure / retirement evidence

PLAT035's architectural decision is ratified by this record. Physical legacy
backend retirement is complete only when downstream implementation evidence can
truthfully establish at least:

```text
TARGET_EXECUTION_BACKEND=TRUFFLE_BYTECODE_DSL_ONLY
NEW_LEGACY_BACKEND_DEPENDENCY_GUARD=ENFORCED
LEGACY_EXECUTABLE_DEPENDENCY_INVENTORY=ZERO
JAVA_OWNED_GUEST_EXECUTION=BYTECODE_BACKED
BOOTSTRAP_RUNTIME_GUEST_EXECUTION=BYTECODE_BACKED
LEGACY_BACKEND_ONLY_TESTS=REMOVED_OR_REPLACED
CANONICAL_TO_TRUFFLE_LOWERER=REMOVED
EXECUTABLE_PROTOS_EXPRESSION_NODE_BACKEND=REMOVED
TOOLING_CONFORMANCE=PASS
FINAL_RESCAN=PASS
AUTHORITATIVE_FULL_VALIDATION=PASS
PROTOS_VISIBLE_SEMANTICS_CHANGED=NO
```

Until those implementation facts are published, the backend is architecturally
retired as a target but may still physically exist as bounded migration debt.
