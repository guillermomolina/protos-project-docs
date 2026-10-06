# Work-item records

`work/` contains durable non-normative records whose primary owner is one formal
tracked work item. Records are grouped by the **individual identifier**, for
example `I026/`, `LIB001/`, `TOOL001/`, or `DOC002/`, rather than by broad family
buckets.

A formal identifier is not created merely to classify documentation. New
unambiguous owner-specific records use `work/<formal-work-item>/`. If an
unexpected legacy/unclassified owner record is discovered later, classify and
migrate it explicitly rather than duplicating or moving it opportunistically.

Current role-first work records:

- [`D179/D179_EXECUTION_CONTEXT_OBJECT_MODEL_CAPABILITY_BOUNDARY.md`](D179/D179_EXECUTION_CONTEXT_OBJECT_MODEL_CAPABILITY_BOUNDARY.md)
  — open D179 execution-context/object-model capability-boundary decision trigger, PLAT036 blocking dependency, and comparative-runtime evidence checkpoint.

- [`D179-A/D179-A_STRUCTURAL_MUTATION_CAPABILITY_AUDIT.md`](D179-A/D179-A_STRUCTURAL_MUTATION_CAPABILITY_AUDIT.md)
  — open structural-mutation audit for execution-context removal, late growth, escaped/captured mutation, close, freeze, and Bytecode DSL constraints.
- [`D179-B/D179-B_FIRST_CLASS_CONTEXT_REFLECTION_ESCAPE_AUDIT.md`](D179-B/D179-B_FIRST_CLASS_CONTEXT_REFLECTION_ESCAPE_AUDIT.md)
  — open first-class context identity/reflection/escape audit separating semantic object behavior from physical lexical storage assumptions.
- [`D179-C/D179-C_LEXICAL_DYNAMISM_STATIC_IDENTITY_AUDIT.md`](D179-C/D179-C_LEXICAL_DYNAMISM_STATIC_IDENTITY_AUDIT.md)
  — open lexical-dynamism audit classifying static binding identity/presence/depth versus genuinely required dynamic fallback.

- [`TOOL002/TOOL002_TEST_TOOL.md`](TOOL002/TOOL002_TEST_TOOL.md)
  — canonical non-normative TOOL002 Test Tool lifecycle and implementation record.

- [`LIB018/LIB018_TEST_AUTHORING_MODEL.md`](LIB018/LIB018_TEST_AUTHORING_MODEL.md)
  — ratified minimal suite-native authoring model: module-as-suite plus canonical `std:test/Test` values.

- [`TOOL001/TOOL001_PACKAGE_TOOL.md`](TOOL001/TOOL001_PACKAGE_TOOL.md)
  — canonical non-normative TOOL001 Package Tool lifecycle record.
  - [`TOOL001-F2D workspace execution preflight`](TOOL001/TOOL001_F2D_EXECUTION_PREFLIGHT.md)
  - [`TOOL001-F2E external immutable-package execution`](TOOL001/TOOL001_F2E_EXTERNAL_MATERIALIZATION.md)

- [`PERF010-A/PERF010-A_THIRD_CAUSAL_ABLATION_READINESS.md`](PERF010-A/PERF010-A_THIRD_CAUSAL_ABLATION_READINESS.md)
  — PERF010-A third causal-ablation readiness evidence: activation lexical single-probe diagnostic is established, with no production optimization selected.

- [`PERF004/PERF004_RUNTIME_PERFORMANCE_CHARACTERIZATION.md`](PERF004/PERF004_RUNTIME_PERFORMANCE_CHARACTERIZATION.md)
  — canonical non-normative PERF004 cross-language runtime-performance characterization record.

- [`PERF001/PERF001_BENCHMARKING.md`](PERF001/PERF001_BENCHMARKING.md)
  — PERF001 non-normative benchmark ownership, reproducibility and evidence plan.
  - [`PERF001-F concurrency methodology`](PERF001/PERF001_F_CONCURRENCY_METHODOLOGY.md)

- [`LM008/LM008_CORE_LANGUAGE_SURFACE_COMPLETENESS.md`](LM008/LM008_CORE_LANGUAGE_SURFACE_COMPLETENESS.md)
  — LM008 Core-language surface completeness parent record.
  - [`LM008-B`](LM008/LM008_B_GRAMMAR_EVALUATION_BINDING_CALLABLE_AUDIT.md)
  - [`LM008-C`](LM008/LM008_C_OBJECT_STRUCTURAL_REFLECTION_MUTATION_AUDIT.md)
  - [`LM008-D`](LM008/LM008_D_VALUES_CORE_COLLECTIONS_AUDIT.md)

- [`AUD016/AUD016_TRUFFLE_BYTECODE_DSL_CAPABILITY_ADOPTION_AUDIT.md`](AUD016/AUD016_TRUFFLE_BYTECODE_DSL_CAPABILITY_ADOPTION_AUDIT.md)
  — completed Truffle Bytecode DSL capability-adoption audit, lexical-frame migration backlog, negative findings, and lazy-context decision packet.

- [`AUD013/AUD013_A_ADOPTION_BASELINE_RECONCILIATION.md`](AUD013/AUD013_A_ADOPTION_BASELINE_RECONCILIATION.md)
  — AUD013-A adoption-baseline reconciliation: historical activation anchor retained, post-baseline owners classified, active TEST003/D142/D143/D147/D144 adoption manifest established, already-completed owner cutovers excluded from duplicate sweeps, and AUD013-B1 released as the first integrated source pass.

- [`AUD005/AUD005_E2_A_TEMPORAL_FRACTION_LINEAR_ENCODING_EVIDENCE.md`](AUD005/AUD005_E2_A_TEMPORAL_FRACTION_LINEAR_ENCODING_EVIDENCE.md)
  — AUD005/LIB010-E2-A evidence at exact Protos revision `62f3f5710210f247aad8574d3d0d56d3254bfd70`: temporal fraction encoding now uses one mutable octet buffer and one final decode, structurally restoring linear cost in emitted length; a 400-digit suite-native round-trip and a source-level anti-prepend guard retain F6, while F3, F4, F5 and final closure reconciliation remain open.

- [`AUD006/AUD006_A1_COMMAND_LINE_ACCUMULATION_COMPLEXITY_EVIDENCE.md`](AUD006/AUD006_A1_COMMAND_LINE_ACCUMULATION_COMPLEXITY_EVIDENCE.md)
  — AUD006-A1 static proof that the current `CommandLine` balanced chunk accumulation remains `Theta(N log N)`, plus the retained-evidence plan and platform/runtime repair boundary.
- [`AUD006/AUD006_A2_FULL_PARSE_SCALING_INSTRUMENT_LIMIT.md`](AUD006/AUD006_A2_FULL_PARSE_SCALING_INSTRUMENT_LIMIT.md)
  — AUD006-A2 negative evidence showing that full-parse allocation/timing scaling cannot discriminate the builder's `Theta(N log N)` term from dominant Truffle/interpreter cost, leaving isolated retained evidence blocked on the platform/runtime decision.
- [`AUD006/AUD006_B1_COMMAND_LINE_DEPTH_REMEDIATION_ANALYSIS.md`](AUD006/AUD006_B1_COMMAND_LINE_DEPTH_REMEDIATION_ANALYSIS.md)
  — AUD006-B1 depth/recursion analysis reconciled at Protos `62f3f5710210f247aad8574d3d0d56d3254bfd70`: command canonicalization and selected-child parsing remain execution-stack-depth proportional, both admit mechanical iterative linked-frame repair entirely in Protos, no public depth limit or new platform decision is required, and B2 → B3 is the implementation order.

- [`AUD003/AUD003_PROTOS_SOURCE_STYLE_CONFORMANCE_AUDIT.md`](AUD003/AUD003_PROTOS_SOURCE_STYLE_CONFORMANCE_AUDIT.md)
  — canonical non-normative AUD003 source-style conformance audit record.

- [`DOC001/DOC001_PROGRAMMING_DOCUMENTATION.md`](DOC001/DOC001_PROGRAMMING_DOCUMENTATION.md)
  — canonical non-normative DOC001 programming-documentation work record.

- [`DOC002/DOC002_DOCUMENTATION_PATH_CONTRACT.md`](DOC002/DOC002_DOCUMENTATION_PATH_CONTRACT.md)
  — ratified role-first path and compatibility contract.
- [`DOC002/DOC002_DOCUMENTATION_ARCHITECTURE_AUDIT.md`](DOC002/DOC002_DOCUMENTATION_ARCHITECTURE_AUDIT.md)
  — historical DOC002-A information-architecture audit and Option A decision packet.
- [`DOC002/DOC002_NAVIGATION_FOUNDATION.md`](DOC002/DOC002_NAVIGATION_FOUNDATION.md)
  — DOC002-C2 navigation-foundation closure record.

- [`DOC003/DOC003_DOCUMENTATION_BRANDING.md`](DOC003/DOC003_DOCUMENTATION_BRANDING.md)
  — approved transparent Protos logo/symbol identity and documentation-integration closure.

- [`DOC004/DOC004_MATCHING_EXPRESSIONS_DOCUMENTATION.md`](DOC004/DOC004_MATCHING_EXPRESSIONS_DOCUMENTATION.md)
  — programmer-facing matching-expression documentation and validation closure.

- [`DOC005/DOC005_BUNDLED_TOOLS_TEST_TOOL_DOCUMENTATION.md`](DOC005/DOC005_BUNDLED_TOOLS_TEST_TOOL_DOCUMENTATION.md)
  — maintained Bundled Tools overview plus staged TOOL002 / Test Tool user documentation.

- [`DOC006/DOC006_TRY_PROTOS_GETTING_STARTED.md`](DOC006/DOC006_TRY_PROTOS_GETTING_STARTED.md)
  — maintained runnable onboarding through the Dev Container or official release distribution.

- [`GITHUB012/GITHUB012_ACTIONABLE_PROJECT_VIEWS.md`](GITHUB012/GITHUB012_ACTIONABLE_PROJECT_VIEWS.md)
  — actionable Project-view contract and canonical lifecycle-status hardening.

- [`GITHUB013/GITHUB013_FORMAL_IDENTIFIER_UNIQUENESS.md`](GITHUB013/GITHUB013_FORMAL_IDENTIFIER_UNIQUENESS.md)
  — concurrent formal-identifier uniqueness and fail-closed intake guard.

- [`GITHUB014/GITHUB014_FORMAL_UPSTREAM_TRACKING_FAMILY.md`](GITHUB014/GITHUB014_FORMAL_UPSTREAM_TRACKING_FAMILY.md)
  — formal external-upstream impact, compatibility, evidence, and collaboration tracking family.

- [`GITHUB015/GITHUB015_FORMAL_ISSUE_PUBLICATION.md`](GITHUB015/GITHUB015_FORMAL_ISSUE_PUBLICATION.md)
  — fail-closed formal Issue publication transaction, hierarchy/priority ordering, and decision-approval provenance.

- [`GITHUB017/GITHUB017_CLOSURE.md`](GITHUB017/GITHUB017_CLOSURE.md)
  — durable CI-reactivation closure record with exact product revision, live workflow-run evidence, and GITHUB020 closure-evidence reconciliation.

- [`UPSTREAM002/UPSTREAM002_OL10_CONTAINER_BASE_MIGRATION.md`](UPSTREAM002/UPSTREAM002_OL10_CONTAINER_BASE_MIGRATION.md)
  — GraalVM Community container base migration evaluation OL8 → OL10, impact classification, and closure evidence.

- [`DIST004/DIST004_OL10_CONTAINER_TOOLING_MIGRATION.md`](DIST004/DIST004_OL10_CONTAINER_TOOLING_MIGRATION.md)
  — owning record for the development-container OL10 base-OS and OS-tooling migration.

- [`DIST008/DIST008_MAVEN_COMPATIBILITY_AND_LEAN_PROVISIONING.md`](DIST008/DIST008_MAVEN_COMPATIBILITY_AND_LEAN_PROVISIONING.md)
  — Maven compatibility-floor and lean-provisioning record: DIST008-A PASS,
  owner-approved OL10 `maven-unbound` binding, and staged Protos/benchmark
  consumer reconciliation.

Live status, priority, assignment, and execution discussion remain in GitHub
Issues and the Protos Development Project.

- [`TOOL007/TOOL007_SOURCE_DOCUMENTATION_EXTRACTION.md`](TOOL007/TOOL007_SOURCE_DOCUMENTATION_EXTRACTION.md)
  — D138 source-local documentation extraction, Standard Library reconciliation, and closure record.
