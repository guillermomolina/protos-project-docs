# PERF037-E — Published guarded member reads and terminal returns; graph-measurement handoff (2026-10-09)

**Live work item:** [guillermomolina/protos#851](https://github.com/guillermomolina/protos/issues/851) — **OPEN**, no graph-parity claim.

## Published product and validation provenance

- Exact product commit: [`guillermomolina/protos@70c40d269294937856c898a753af2f830e9fba25`](https://github.com/guillermomolina/protos/commit/70c40d269294937856c898a753af2f830e9fba25), subject `PERF037-E: optimize terminal returns and guarded member reads (#851)`. GitHub publication independently confirmed.
- Version: `0.3.323-SNAPSHOT` in `pom.xml`; `CHANGELOG.md` entry published. Exactly ten changed files: five production Java, three regression Java, and those two metadata files.
- Maintainer reports successful local complete `make test` and clean `git diff --check` before product publication; no logs or independent build execution retained here. Do not conflate a reported local PASS with a coordinator-executed test.
- The maintainer reports the commit pushed with clean diff. No separate clean-checkout benchmark evidence for `70c40d26` exists at the moment this record was authored.

## Changes covered by PERF037-E

- `CanonicalToBytecodeLowerer`: direct terminal return for admitted literal/lookup/member/identity value expressions; Closure terminal roots prepared before opening return; nonterminal simple values discarded rather than stored/reloaded; composed-expression scratch shared where admitted. The scalar-local read and exception/control-transfer fallback paths are retained.
- `ProtosSemanticBytecodeRootNode`: `ReadMemberAtRoot`, `ReadInlineMember` and ordinary `ReadMember` accept constant member names; `DiscardValue` consumes evaluated unused expressions. Stable ordinary-data member reads have their own Truffle DSL specialization alongside the preexisting generic fresh Closure-extraction path.
- `ProtosBytecodeRootNode` and runtime `ProtosValueLookup`/`ProtosMapBackedLexicalBindingAuthority`: guarded exact-receiver and shared-inherited reads can load the current physical `SlotCell` value under both selection stability and a lazily allocated, one-way non-Closure continuity assumption; a data-to-Closure write invalidates before replacement. The parent, shadowing, owner and fresh method-binding rules remain in the fallback.
- Regressions cover direct terminal return, sequence ordering, error/control transfer, instrumentation tags, constant member names, non-Closure assumptions and transitions, inherited shadowing, preserved fresh Closure extraction, and pay-as-you-grow object storage.

## Reference graph and strict next measurement (current-HEAD policy)

- Last measured **pre-E** product: `89e1b038c2fd510f47ddb991496882548d1bbe44`, retained valid/stable `After TruffleTier` graph **64 nodes** (mixed PERF037-D/PERF038-F revision) in [benchmarks published capture](https://github.com/guillermomolina/protos-benchmarks/tree/dd8b6518057de62d940a10e5b3db1de7ea97929b/results/perf037-d-89e1b038-graphs).
- Historical stable JavaScript comparison for `primitive-object-slot-read`: **36 nodes**, `executable-value` surface; Protos uses `canonical`, so a numerical node gap alone does not prove identical compiled work or a guaranteed net deletion count.
- **E measurement pending**. Do not claim a node count, stabilization, latency or attainment of the `<=36` target before the new evidence is captured and verified.
- Required new evidence unit: the **real current clean Protos HEAD at capture time**, recorded as an exact full SHA by the harness; workload `primitive-object-slot-read`, language `protos`, stage `reference`, currently committed `truffle/measure_graphs.py` and `truffle/measure/graphs.json` policies. The PERF037-E publication SHA `70c40d26` is historical implementation provenance, **not a checkout constraint**. Concurrent legitimate follow-up commits must not be rolled back. Name the output `results/perf037-current-<PRODUCT_HEAD_SHORT>-graphs` to avoid overwrites.
- Capture only Protos. Correctness must PASS. Do not suppress `NOT_STABLE`, expand budgets silently, overwrite old captures, or recapture unchanged GraalJS/GraalPy.

Human-executor commands (benchmark devcontainer, using the current clean `/workspaces/protos` HEAD; do not check out the earlier E revision):

```bash
cd /workspaces/protos-benchmarks
PRODUCT_SHA=$(git -C /workspaces/protos rev-parse --short=8 HEAD)
python3 truffle/measure_graphs.py capture --dir /workspaces/protos --stage reference --workload primitive-object-slot-read --language protos --output "results/perf037-current-${PRODUCT_SHA}-graphs"
```

On a host with the pinned Docker IgvUtility analyzer and the same `results/` volume:

```bash
cd ~/Fuentes/protos-benchmarks
# Set RESULTS_DIR to the exact "results/perf037-current-<SHA>-graphs" path from capture.
RESULTS_DIR="results/perf037-current-REPLACE_WITH_CAPTURED_PRODUCT_SHA-graphs"
python3 truffle/measure_graphs.py analyze --output "$RESULTS_DIR"
python3 truffle/measure_graphs.py summarize --output "$RESULTS_DIR"
python3 truffle/measure_graphs.py verify --output "$RESULTS_DIR"
```

Acceptance gate: read exact `capture.json.product_before` identity/clean flag, `unit.json.protos_revision`, `capture_valid`, `evidence_valid`, stabilization status, selected-phase graph and full histogram/attribution. If valid and stable, compare new Protos-only `total_nodes` with 64 and the archived 36-node JS baseline, **but attribute any change to the complete intervening commit range rather than uniquely to PERF037-E**; retain raw BGVs and selected filtered graph before declaring graph progress. Any failed correctness/stabilization invalidates the relevant claim. Issue remains OPEN pending graph evidence and independently evaluated latency/cost.


## Diagnostic capture received: dirty product state

The maintainer executed `truffle/measure_graphs.py capture` for `primitive-object-slot-read/protos` under the current working tree. The first reference capture was rejected because the product checkout was dirty. The maintainer then deliberately used `--allow-dirty-product`, preserving concurrent agents' work instead of resetting any files. Harness output:

```text
RESULT_DIR=results/perf037-current-70c40d26-graphs
SURFACE_SUPPORTED=YES
ADAPTER_CACHE=compiled
CORRECTNESS=primitive-object-slot-read/protos PASS
BUDGET=primitive-object-slot-read/protos 1000 NOT_STABLE units=1 problems=UNIT_NOT_AT_FINAL_TIER
BUDGET=primitive-object-slot-read/protos 4000 NOT_STABLE units=1 problems=UNIT_NOT_AT_FINAL_TIER
BUDGET=primitive-object-slot-read/protos 16000 STABLE_CANDIDATE units=1 problems=-
BUDGET=primitive-object-slot-read/protos 64000 STABLE_CANDIDATE units=1 problems=-
CAPTURE=primitive-object-slot-read/protos valid=YES stabilization=STABLE pair=[16000, 64000]
CAPTURE_INVALID_CASES=0
```

**Scope:** this is a **diagnostic capture of a dirty product working tree**. The `70c40d26` short revision embedded in the output directory labels only the repository commit; it does **not** mean the compiled sources match that clean commit. The capture metadata includes the product source-state SHA256 to distinguish the working tree. The graph driver accepts explicit dirty-product captures for diagnosis; its `summarize_case` structural validity check does not independently reject a dirty product, so `evidence_valid=YES` by itself would not make this a clean reference. Do not attribute the graph exclusively to PERF037-E and do not discard concurrent uncommitted work.

**Next:** run analyzer, summarizer and producer-hash verification against the existing output directory, inspect the measured total and node histogram; compare diagnostically with historical Protos 64 and JS 36. Retain this capture as dirty diagnostic evidence, and perform a fresh clean-HEAD reference capture later only if a durable, reproducible comparison is required.
