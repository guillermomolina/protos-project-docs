# PERF035 — Published slot-write repair and Tier-2 stabilization checkpoint (2026-10-08)

**Role:** durable, non-normative evidence. This record describes human-reported executions and read-only GitHub verification; it does not replace retained raw benchmark artifacts or the live GitHub Issue.

**Owner:** [PERF035 / protos#846](https://github.com/guillermomolina/protos/issues/846).  
**Published implementation:** [`guillermomolina/protos@203c0f912be52db1b1adf4e528851e7622c41d95`](https://github.com/guillermomolina/protos/commit/203c0f912be52db1b1adf4e528851e7622c41d95), version `0.3.302-SNAPSHOT`.  
**Benchmark repository:** `guillermomolina/protos-benchmarks` (exact benchmark HEAD was not supplied; source hashes were subsequently verified).  
**Workload:** `primitive-object-slot-write`, result `2`, unchanged reference policy.

## Published repair and source validation

The published product commit contains exactly eight paths: `pom.xml`, `CHANGELOG.md`, `ProtosBytecodeRootNode.java`, `ProtosSemanticBytecodeRootNode.java`, `ProtosObjectValue.java`, `ProtosValueLookup.java`, `ProtosGuardedLookupTest.java`, and `ProtosPrimitiveSlotWriteRegressionTest.java`. The implementation separates selection-only dependencies for guarded member reads from value-sensitive lookup dependencies, while retaining fresh value extraction and structural mutation invalidation.

The maintainer reported PASS for the complete focused tests and integrated `make test` **before** the Maven version and changelog finalization. The human executor published this candidate; no post-finalization test rerun is asserted.

## Measured compiler lifecycle

Reference capture from the benchmark devcontainer:

```sh
python3 truffle/measure_graphs.py capture --dir ../protos \
  --language protos --workload primitive-object-slot-write \
  --stage reference --output results/local/perf035-203c0f91 \
  --allow-dirty-product
```

The capture recorded product revision `203c0f912be52db1b1adf4e528851e7622c41d95` and:

```text
CORRECTNESS=primitive-object-slot-write/protos PASS
BUDGET 1000: NOT_STABLE (UNIT_NOT_AT_FINAL_TIER)
BUDGET 4000: NOT_STABLE (UNIT_NOT_AT_FINAL_TIER)
BUDGET 16000: STABLE_CANDIDATE, problems=-
BUDGET 64000: STABLE_CANDIDATE, problems=-
CAPTURE=primitive-object-slot-write/protos valid=YES stabilization=STABLE pair=[16000, 64000]
CAPTURE_INVALID_CASES=0
```

The previously reported 100 Tier-1 recompilations and 200 invalidations came from a **different, older checkout** (`b0776d0d...`), which was mounted in the benchmark devcontainer before publication. Those previous measurements must not be attributed to the repaired revision.

## Retained graph analysis and provenance

The maintainer ran `graphs-analyze`, `graphs-summarize`, and `graphs-verify` on the retained capture. Reported output:

```text
ANALYZE_FAILURES=0
UNIT=primitive-object-slot-write/protos valid=YES total=1122 graphs=1
INVALID_UNITS=0
CASES=1
WORKING_TREE_MATCHES_PRODUCER=YES
HEAD_MATCHES_PRODUCER=YES
PRODUCT_CLEAN=False
EVIDENCE_VALID=True
SELECTED_STABLE_TIER=2
AFTER_TRUFFLE_TIER_CONFIRMED=True
```

The two retained BGVs, at budgets 16000 and 64000, passed the `After TruffleTier` structural equality check. `1122` is the selected Protos compiled-graph node count, **not** timing, speedup, or a cross-language parity verdict. No GraalJS/GraalPy peer graphs were supplied in this capture (`peer=UNRESOLVED`, `protos_status=NOT_EVALUATED`).

**Important acceptance boundary:** the recorded `product_before.clean` is **false**; the command used `--allow-dirty-product`. Therefore this measurement establishes strong *diagnostic* correctness, Tier-2 stabilization, and structural agreement for the reported product revision, but **does not satisfy clean-checkout reference evidence eligibility**. Harness producer hashes matched its working tree and HEAD; these checks do not establish product checkout cleanliness. A fresh clean-checkout reference capture with the same published commit and unchanged harness, followed by graph analysis and verification, remains pending for formal PERF035 closure. No new source patch or redundant functional test is required solely by this provenance gap.

## Workstream routing

The project owner has explicitly **paused PERF032's graph-node research** until the separate [PERF034 local-read issue](https://github.com/guillermomolina/protos/issues/845) is repaired/validated. The demonstrated PERF035 compiler stabilization is retained; it is not a reason to resume arbitrary-node-count investigation or to reopen the discontinued broad PERF024/PERF030 campaigns. PERF035/#846 stays open for its clean provenance acceptance gate.
