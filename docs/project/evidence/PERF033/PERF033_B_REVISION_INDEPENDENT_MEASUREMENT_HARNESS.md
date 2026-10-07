# PERF033-B — revision-independent Protos measurement harness

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=PERF033
SLICE=PERF033-B
ISSUE=guillermomolina/protos#832

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_REVISION=e3609eca8e678764a2b9453f248849ef8ff6e673
SUBJECT=PERF033: add revision-independent Protos measurement driver
~~~

This record is durable non-normative harness evidence. It does not claim that
the locally produced PERF033 reference result has already been retained in the
benchmark repository.

## Published harness capability

The published benchmark revision adds the stable driver:

~~~text
truffle/measure_protos.py
~~~

Its governing model is:

~~~text
PROTOS_REVISION_IS_DATA=YES
PRODUCT_SELECTION=EXPLICIT_CHECKOUT_DIRECTORY
PRODUCT_HISTORY_CHECKOUT_SUPPORT=NO

HARNESS_SOURCE_CHANGES_FOR_ORDINARY_PRODUCT_REVISION=NO
COMPILE_EMBEDDED_DRIVER_PER_SELECTED_PRODUCT=YES
COMPILED_DRIVER_CACHE_IS_EVIDENCE=NO

PRE_MEASUREMENT_HARNESS_COMMIT_REQUIRED=NO
DIRTY_HARNESS_MEASUREMENT_SUPPORTED=YES
EXACT_PRODUCER_SOURCE_HASHES_RECORDED=YES
~~~

The driver measures exactly the checkout supplied by `--dir`. It has no
revision/commit/tag checkout surface and uses read-only Git operations for
product identity.

For Protos, the selected surface adapter is compiled against the exact selected
checkout and cached under the ignored working area. The cache is disposable and
is not historical measurement evidence.

## Published measurement surfaces

The new path publishes independent adapters for:

~~~text
dynamic
  -> ProtosStandaloneHostedSession.invokeTopLevel("run")

prepared
  -> prepareTopLevel("run")
  -> PreparedTopLevel.invoke()

canonical
  -> prepareTopLevel("run")
  -> PreparedTopLevel.executable()
  -> Value.execute()

peer executable-value
  -> GraalJS / GraalPy executable Value.execute()
~~~

Each requested Protos surface is compiled independently against the selected
product checkout so an unavailable surface fails closed without forcing
unrelated surfaces to compile or silently falling back.

## Evidence identity and lifecycle

The driver records product identity before the batch and checks it again after
the batch. The published implementation also records the harness producer state
and exact hashes of the source files that produced a run.

A run is invalidated if the selected Protos product or harness producer changes
during the batch.

Measurement policy is data-owned by:

~~~text
truffle/measure/cases.json
~~~

and the driver supports correctness-before-timing, benchmark execution and an
optional steady-bounded JFR profile without requiring source edits for a new
ordinary Protos revision.

Existing valid output is not silently remeasured; explicit remeasurement is a
separate operation.

## Published files

The harness publication contains:

~~~text
BENCHMARKING.md
CHANGELOG.md
tests/test_measure_protos.py
truffle/Makefile
truffle/measure/cases.json
truffle/measure/engine/com/guillermomolina/protos/benchmarks/measure/MeasurementEngine.java
truffle/measure/surfaces/canonical/ProtosCanonicalSurface.java
truffle/measure/surfaces/dynamic/ProtosDynamicSurface.java
truffle/measure/surfaces/peer/PeerExecutableSurface.java
truffle/measure/surfaces/prepared/ProtosPreparedSurface.java
truffle/measure/surfaces/protos/ProtosSurfaceSupport.java
truffle/measure_protos.py
~~~

Historical PERF024/PERF025 exact-revision harnesses and the older generic JVM
path are intentionally not rewritten by this slice.

## Maintainer-reported validation and publication

The maintainer reported after local validation:

~~~text
GIT_DIFF_CHECK=PASS
LOCAL_TESTS=PASS
~~~

The harness was then pushed to benchmark `main`:

~~~text
PREVIOUS_MAIN=4aae2a211e376ed965239b87854fe0bb2758cdb0
PUBLISHED_MAIN=e3609eca8e678764a2b9453f248849ef8ff6e673
~~~

The push-time working-tree summary also showed:

~~~text
?? results/perf033-current/
~~~

Therefore:

~~~text
HARNESS_PUBLICATION=COMPLETE
REFERENCE_RESULT_DIRECTORY_EXISTS_LOCALLY=YES
REFERENCE_RESULT_PUBLICATION=NOT_YET_PROVEN
~~~

No numerical PERF033 reference result is admitted by this record. The
`results/perf033-current/` tree must be inspected, producer-verified and
published before its measurement claims become durable benchmark evidence.

## Relationship to PERF033-A

PERF033-A remains the product repair at:

~~~text
PROTOS_REVISION=c03370abca4592b95d35ccba5c4a185b955bd4bb
PRODUCT_RESULT=CANONICAL_VALUE_EXECUTE_WITHOUT_ROOT_TASK_OR_PROTOS_CONTEXT_ENVELOPE
~~~

The new harness is revision-independent: future measurements use whichever
clean Protos checkout is explicitly supplied at execution time and record that
exact product revision. A product revision change by itself does not require a
harness source change.

## Remaining PERF033 closure work

PERF033 remains open until the already-produced reference result is either
published as valid retained evidence or explicitly rejected by its own recorded
validation.

The next bounded action is evidence publication, not another product
optimization and not another performance investigation.

~~~text
PERF033_B_HARNESS_STATUS=COMPLETED
PERF033_REFERENCE_EVIDENCE_STATUS=PENDING_PUBLICATION
NEXT_PRODUCT_WORK_REQUIRED=NO
NEW_ISSUE_REQUIRED=NO
~~~

## AI-assistance disclosure

This durable record was materially prepared with AI assistance from ChatGPT
using the exact published benchmark commit and maintainer-reported validation
and push state.
