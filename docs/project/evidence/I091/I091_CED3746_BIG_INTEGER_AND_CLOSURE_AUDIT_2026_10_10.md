# I091 — BigInteger containment and closure audit at 0.3.331-SNAPSHOT

**Audit date:** 2026-10-10
**Issue:** [I091 / #869](https://github.com/guillermomolina/protos/issues/869), **OPEN / NOT READY FOR CLOSURE**.
**Product revision:** `PROTOS_REVISION=ced3746f9664ec5ceb562461c077bdf0627e5663` ([immutable commit](https://github.com/guillermomolina/protos/commit/ced3746f9664ec5ceb562461c077bdf0627e5663)).
**Published product version:** `0.3.331-SNAPSHOT`.
**Parent product revision:** `70ee9504224c59c033c647f9185bf8fb9b266b93` (I092).
**Previous I091 post-publication audit:** [I091_POST_PUBLICATION_NUMERIC_CONTAINMENT_AND_CANDIDATE_C_AUDIT_2026_10_10.md](I091_POST_PUBLICATION_NUMERIC_CONTAINMENT_AND_CANDIDATE_C_AUDIT_2026_10_10.md).
**Architectural authority:** ratified [PLAT056 Candidate C](../../decisions/platform/PLAT056_PRIMITIVE_FIRST_NUMERIC_GUEST_OBJECT_ARCHITECTURE.md), [D197](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md) as future numeric-tower semantic authority; current product normative `spec/semantics/VALUES_AND_COLLECTIONS.md` still governs observed behavior.
**Validation provenance:** the maintainer reports the published commit has `git diff --check` clean and all local tests PASS. Exact commands, test counts, Native Image execution and fresh benchmarking were not provided. This auditor **did not execute builds, tests, validators or benchmarks**. It does not infer their outcomes.

## 1. What the new publication actually establishes

GitHub `main` was inspected at the exact product commit. The commit changes 43 production Java paths (including the removed old and added new rounding locations), plus test and Protos conformance paths, `pom.xml` and `CHANGELOG.md`.

The material source improvements inspected in this revision are:

- `ProtosIntegerValue` is signed-64; `ProtosLargeIntegerValue` has a narrow `BigInteger` payload and ordinary frozen-object delegation; semantic normalization remains in `ProtosNumericValueSupport`.
- `ProtosBinary64Rounding` moved from `execution` to `runtime`. **Two signed-64 exact operands take the primitive-only division algorithm**; exact large-operand quotients still use legitimate unbounded arithmetic. Correct rounding and mixed large/small boundaries are exercised by `ProtosI091Binary64RoundingTest`, including differential/oracle cases (the maintainer reports local tests PASS).
- `ProtosNumericHashKey` stores small hashes as `long` and genuinely large hashes as exact `BigInteger`, comparing full values rather than truncated lower words. `ProtosStandardMapProtocolTest` covers large hash equality, distinct values with equal low bits, collisions, recorded-hash changes, and comparison mutation.
- `ProtosStandardObjectProtocol.call` now includes `ordinaryReceiver(receiver)`; large semantic Integers are excluded as a receiver to the standard object-constructor protocol. Guest conformance tests distinguish small and large Integer receiver/prototype behavior.
- File positions and host limits go through exact semantic numeric acceptance and range checking; Polyglot array indices outside the signed-64 nonnegative domain are rejected **before** foreign access.
- `ProtosNumericValueSupport.Octets` now lazily initializes the 256 cached guest octets only on byte-I/O use, rather than imposing eager wrappers on unrelated arithmetic.
- `ProtosSemanticTransferPayload` centralizes the PLAT051 host `BigInteger` leaf protocol; regex families no longer need to manipulate that host class directly.
- `ProtosHostJavaProvider`, the foreign value adapter and literal lowering now use normalized host numeric scalars, preserving explicit Java/SPI contracts.

This is meaningful progress against earlier audit findings; **neither lexical-count reduction nor existence of tests establishes I091 closure by itself**.

## 2. Verified surviving `BigInteger` production uses

The **checked subset** is 16 production Java files with 73 code occurrences of the identifier `BigInteger` (imports, type names, operations; comments and string literals excluded by a read-only source-level lexical inspection). Every listed file was fetched from the pinned product revision. The historical I091-A baseline was **447 `BigInteger` occurrences**, 187 `ProtosIntegerValue`, 49 `ProtosFloatValue`, 683 total numeric-symbol occurrences in 77 of 431 production Java files, at historical `76de43651079172d90be5bc6b1f535336ae47cfb`. The **current tree has 436 production Java files**; this review does not assert that the indexed search covers every one.

| Inspected file (relative to `src/main/java/com/guillermomolina/protos/`) | Occurrences | Classification and bounded disposition |
| --- | ---: | --- |
| `runtime/ProtosFloatValue.java` | 2 | Truffle interop `asBigInteger` projection: explicit ABI; retain |
| `runtime/ProtosIntegerValue.java` | 3 | Truffle exact host projection `asBigInteger`; `BigInteger.valueOf(long)` only on projection; retain |
| `runtime/ProtosNumberLiteral.java` | 2 | Parse genuinely large literal only after primitive accumulation overflows; retain |
| `runtime/ProtosLargeIntegerValue.java` | 5 | Immutable large exact payload and host projection; retain under Candidate C while F1 proof is completed |
| `runtime/ProtosNumericValueSupport.java` | 14 | Internal large arithmetic, normalization, unsigned encoding, exact host scalar admission; inspect invocation paths in final full census; no observed generic-client Java coupling |
| `runtime/ProtosBinary64Rounding.java` | 14 | True large-precision rounding/quotient/comparison and explicit integral-binary64 projection; signed-64 division primitive; retain exact algorithm, continue F3 proof |
| `runtime/ProtosNumericHashKey.java` | 3 | Exact unbounded semantic Map hash-key storage; removing would reintroduce truncation; retain |
| `runtime/ProtosFixedIntegerInteropValue.java` | 5 | Exact unsigned 64-bit Java/Truffle projection; retain |
| `runtime/ProtosSemanticTransferPayload.java` | 10 | PLAT051's inert host BigInteger leaves, now encapsulated for families; retain while that contract governs |
| `execution/ProtosHostJavaProvider.java` | 3 | Explicit Java `BigInteger.class` parameter conversion; retain |
| `execution/ProtosForeignPluginValueAdapter.java` | 2 | Public foreign SPI exact-Integer scalar boundary; retain |
| `execution/ProtosHostJavaCatalogue.java` | 2 | Explicit supported Java class catalogue; retain |
| `execution/CanonicalToBytecodeLowerer.java` | 2 | Large-literal host descriptor specialization; do not force ordinary literal paths through BigInteger |
| `execution/ProtosSemanticBytecodeRootNode.java` | 1 | Materialization of a genuinely large literal descriptor; retain |
| `spi/foreign/ProtosForeignValueClass.java` | 2 | Public foreign exact-Integer ABI; retain |
| `spi/foreign/polyglot/ProtosPolyglotValueOperations.java` | 3 | Explicit Polyglot integral projection and exact signed-64 index validation; retain |

**Subtotal by role:** internal numeric algorithms, literals and hash (48); PLAT051 semantic-transfer payload (10); host/SPI/Polyglot contracts (12); lowering/executable literal boundary (3). **Total checked = 73 / 16 files.** Counts are source-checked within this scope, **not** a certified 436-file census. Several stale GitHub code-index results still refer to the deleted `execution/ProtosBinary64Rounding.java`; a full repository `grep` or the authoritative `tools/numeric_dependency_census.py` at the product commit must settle the remaining uncertainty.

**Assessment:** No *confirmed unjustified* BigInteger occurrence was found in these 16 checked files. This does **not** classify unknown files or prove every call site cheap. Do not reopen correct Java/SPI ABI contracts to pursue an arbitrary zero-reference quota. Do not regard every retained reference as proven indispensable without the source-scoped acceptance work below.

### Important cost/algorithm qualifications

- `ProtosBinary64Rounding.dividePrimitiveIntegers` is source-confirmed to avoid BigInteger for signed-64/signed-64 quotients. Its `divideLarge` still shifts and divides arbitrary-precision operands. Large allocation may be legitimate, but deterministic differential/oracle tests and comparative analysis of representable powers-of-two, exponent boundaries, ties-even, subnormals, underflow, overflow and exact large ratios remain part of F3 acceptance. The earlier identified *small operand pair BigInteger fallback* is fixed by this publication.
- `compareLargeIntegerToBinary64` still uses unbounded magnitude operations when one operand is truly large; this does not burden ordinary signed-64 comparison and is **not** by itself an unjustified dependency.
- `ProtosSemanticTransferPayload.integer(long)` allocates/obtains a BigInteger for an explicitly requested PLAT051 transport leaf, including small integers. This is a **selected transport contract**, not evidence of an allocation in normal arithmetic. Any proposed redesign of transport leaves must respect PLAT051 authority.
- `ProtosNumericHashKey` must preserve exact arbitrary-precision hash equality; `hashCode()` being a 32-bit Java collection hash is allowed to collide, whereas `equals()` must compare the full numeric key.
- The 256 octet wrappers now live in the lazy nested `Octets` holder; byte-I/O users pay only when using octet materialization.

## 3. Related numeric-boundary and representation checks

Source-level evidence reviewed in the pinned product revision:

- `ProtosStandardObjectProtocol.call` rejects `ProtosLargeIntegerValue` through `ordinaryReceiver` exactly where earlier audit identified inconsistent small/large behavior.
- `ProtosValueLookup` and `ProtosIdentity` use semantic numeric recognition/identity rather than Java number identity.
- `ProtosActorValueTransfer`, `ProtosDetachedExecutionValue`, and `ProtosParallelRuntime` inspect/copy semantic numbers before ordinary object transfer/reflection. `ProtosActorBootstrap` disallows a large Integer as an Actor behavior object.
- `ProtosStandardMapProtocol` accepts a semantic Integer hash through `ProtosNumericHashKey`; `ProtosStandardIdentityMapProtocol` remains identity-based.
- `ProtosCurrentNumericRelations` and `ProtosStandardNumericConversionProtocol` contain no remaining code `BigInteger` mentions; the remaining physical numeric-value checks there are **within numeric-protocol implementation**, not incidental I/O clients.
- Representative direct consumers in byte/text streams, network, package adapters and CLI use the semantic service for exact bounded conversion. Existing code-search hits may be stale; each affected boundary needs the pinned full-tree census before unconditional F5 closure.

This is a focused audit of high-risk runtime and protocol surfaces, **not** proof that every `instanceof ProtosObjectValue` in 436 Java production files is representation-independent. Other generic object and debugger/tooling entry points are a separate concrete F4 audit obligation, not an authorization to change semantics speculatively.

## 4. Formal closure matrix

| Gate | Result at pinned revision | Remaining acceptance |
| --- | --- | --- |
| F1 immutable ordinary rich guest values | **PARTIAL**: signed-64/large Integer distinction, frozen ordinary large Integer, identity/hash/transfer tests; current visible large values remain semantic Integer | Exhaustive generic-object, debugger, reflection, interop and mutability audit for large representation; future distinct BigInteger/Fraction/Complex guest-family behavior remains I090's normative/public implementation responsibility; don't invent `Integer.recognizes` |
| F2 primitive-first execution | **SOURCE-LEVEL SUBSTANTIAL PASS, formal closure pending**: both Bytecode DSL roots, guarded send/return propagation, signed-64 arithmetic, no forced BigInteger in signed-64 quotient | Exact source-revision trace of primitive carriers across both roots, D013 overrides/deopt and materialization observation boundaries; required structural/affected regression evidence |
| F3 normalization/rounding | **STRONG IMPROVEMENT**, not all future-family semantics covered | Pin differential/oracle test outcomes for signed-64 and arbitrary-precision ratios; keep signed-zero, NaN, subnormals and overflow exact; D197 Fraction/Complex normalization is I090 |
| F4 semantic observation/isolation | **PARTIAL**: fixed Object.call, exact hashes, Map and major Actor/P/Process/Polyglot tests added | Cross-context/native-image and *all reachable generic object operations* should not leak the large-value Java subclass; human must supply any required Native Image acceptance evidence tied to final revision |
| F5 full containment census | **OPEN**: 73 checked BigInteger code occurrences in 16 pinned files; no confirmed unjustified one among them | Run `python3 tools/numeric_dependency_census.py` in product checkout at `ced3746f...`, classify **every** `BigInteger`, `ProtosIntegerValue`, `ProtosFloatValue` site, record a source fingerprint and before/after module counts; investigate any new/uncounted paths; avoid quota-driven rewrites |
| F6 compiled performance | **OUTSIDE I091's measured acceptance**: structurally improved hot paths, no measurements claimed | PERF040 owns pinned graph and stable timing once semantic/structural prerequisites are met; don't claim performance success from import counts |

## 5. Necessary continuation before closing #869

1. **Complete the reproducible whole-tree census**, not a GitHub-indexed search: at the actual checkout/commit run `python3 tools/numeric_dependency_census.py`; inspect `target/i091-numeric-census.json` and `.tsv`; classify every code site as numeric algorithm, payload/carrier, intentional external ABI, explicit transfer, lowering/materialization, incidental coupling or unnecessary small-path cost, then compare with I091-A's published baseline and record the source fingerprint. An output with `UNCLASSIFIED_NEEDS_SOURCE_AUDIT` remains open evidence, not closure.
2. **Audit generic object reachability** for `ProtosLargeIntegerValue extends ProtosObjectValue`: debugger/tooling/reflective/interop paths and each generic operation that assumes `instanceof ProtosObjectValue` means ordinary mutable/prototypable object. The fixed `Object.call` is only one example; classify others and add targeted tests only when a real reachable divergence exists. Preserve original receiver and D013 selected aliases/overrides.
3. **Reconcile F2/F3/F4 with final runnable test evidence**: two roots, primitives through locals/closures/returns/guarded sends, correct fallback and invalidation, exact mixed/large rounding and hashing, signed zero/NaN, Actor/P/Process/Context ownership. The maintainer has reported all local tests PASS for the published commit, but exact invoked commands/coverage and a fresh Native Image result were not supplied. Do not request repeated full-suite runs without new source changes.
4. **Separate external owners**: I090/#868 is BLOCKED on the distinct future D197 public recognition/numeric tower gate; PLAT056's architecture must remain compatible, but do not install I090 public semantics in I091. PERF040/#862 independently owns graph/timing measurements after a correct structural architecture.
5. When all I091-owned source fixes and checks finish, the human performs required validation at final changed-path closure (including `make test` where policy requires), then updates `pom.xml` and root `CHANGELOG.md` **only after green tests**, immediately before source commit/push, **never running tests after those metadata edits**. Pin resulting product SHA; publish full final source fingerprint/census and test provenance to this docs repository; then update/close I091 with GITHUB020 closure verification. This current audit is **not that final acceptance record**.

**No new Ixxx was created**: these remain acceptance obligations of I091/#869, not a quota or a new design selection.

## Source provenance

Read-only inspection of pinned `protos@ced3746f9664ec5ceb562461c077bdf0627e5663` and its Git tree (436 production Java files), exact changed paths, the 16 individual BigInteger-bearing production files enumerated above, the previously indexed candidate paths with now absent references, high-risk runtime consumers (`ProtosStandardObjectProtocol`, `ProtosValueLookup`, `ProtosIdentity`, `ProtosCurrentNumericRelations`, `ProtosStandardNumericConversionProtocol`, `ProtosActorValueTransfer`, `ProtosDetachedExecutionValue`, `ProtosParallelRuntime`, `ProtosStandardMapProtocol`, `ProtosStandardIdentityMapProtocol`, `ProtosActorBootstrap`, `ProtosDebuggerScope`, and additional selected generic-object consumers), plus focal JUnit/conformance test names including `ProtosI091Binary64RoundingTest`, `ProtosI091IdentityRepresentationTest`, `ProtosI091PrimitiveCarrierRootsTest`, `ProtosStandardMapProtocolTest` and `ProtosForeignInteropTest`.

Governance and specifications consulted: product `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/REFERENCE.md`, `src/AGENTS.md`, currently normative values/collections specification, PLAT056/D197 ratified records, prior I091 evidence, and live GitHub I091/I090/PERF040 status. No new language semantics are selected here.

**AUDIT_DISPOSITION=OPEN; PRODUCT_SOURCE_CHANGED_BY_AUDITOR=NO; NEW_TESTS_RUN_BY_AUDITOR=NO.**
