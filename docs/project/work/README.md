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

- [`LIB013/LIB013_0_DATETIME_FOUNDATIONS_DECISION.md`](LIB013/LIB013_0_DATETIME_FOUNDATIONS_DECISION.md)
  — ratified LIB013-0 Candidate A datetime foundation: pure civil/timeline values first, explicit future timezone/clock authorities, and LIB013-A through D implementation routing.
- [`LIB013/LIB013_A_PURE_CIVIL_TEMPORAL_KERNEL_EVIDENCE.md`](LIB013/LIB013_A_PURE_CIVIL_TEMPORAL_KERNEL_EVIDENCE.md)
  — LIB013-A implementation evidence at Protos `b0bf563c30a612210cd67c6a0bbea01f44617c3c` / `0.3.242-SNAPSHOT`: Date, Time and LocalDateTime pure civil values published with validation, equality/hash/order and frozen-value tests; maintainer reports `git diff --check` and all local tests PASS; LIB013-B is next.
- [`LIB013/LIB013_B_TEMPORAL_AMOUNTS_CALENDAR_ARITHMETIC_EVIDENCE.md`](LIB013/LIB013_B_TEMPORAL_AMOUNTS_CALENDAR_ARITHMETIC_EVIDENCE.md)
  — LIB013-B implementation evidence at Protos `41a06e07d5d74053911f291ac27f911678c61ff5` / `0.3.243-SNAPSHOT`: exact Duration amounts, structural Period values, and ratified Gregorian calendar arithmetic published; maintainer reports `git diff --check` and all local tests PASS; LIB013-C is next.
- [`LIB013/LIB013_C_INSTANT_FIXED_OFFSET_DOMAIN_EVIDENCE.md`](LIB013/LIB013_C_INSTANT_FIXED_OFFSET_DOMAIN_EVIDENCE.md)
  — LIB013-C implementation evidence at Protos `d6981daa330a2a3a767865e9cd88a6fb9f626e81` / `0.3.246-SNAPSHOT`: fixed Offset, unbounded Instant, structural OffsetDateTime, exact fixed-offset conversion, and canonical datetime family recognition published; maintainer reports `git diff --check` and all local tests PASS; LIB013-D is next.
- [`LIB013/LIB013_D_EXPLICIT_TEMPORAL_TEXT_PROFILES_AND_CLOSURE_EVIDENCE.md`](LIB013/LIB013_D_EXPLICIT_TEMPORAL_TEXT_PROFILES_AND_CLOSURE_EVIDENCE.md)
  — LIB013-D and parent closure evidence at Protos `c851c6570b93503a9269024da34d71fdf4d0eb12` / `0.3.251-SNAPSHOT`: bounded ISO8601 and RFC3339 profiles published, maintainer validation passed, and the ratified LIB013 A-through-D sequence is complete with no remaining authorized slice.

- [`LIB015/LIB015_0_LOGGING_ARCHITECTURE_DECISION.md`](LIB015/LIB015_0_LOGGING_ARCHITECTURE_DECISION.md)
  — ratified LIB015-0 logging architecture: immutable structured events, explicit derived Logger + Sink authority, early filtering, reuse of TextWriter/std:json/std:datetime, and reusable terminal-styling ownership outside the logging core.

- [`LIB015/LIB015_A_STRUCTURED_LOGGING_CORE_EVIDENCE.md`](LIB015/LIB015_A_STRUCTURED_LOGGING_CORE_EVIDENCE.md)
  — LIB015-A implementation evidence at Protos `c5bce4ef03417cce96b99924a0ae58e7cb95e7bb` / `0.3.247-SNAPSHOT`: canonical logging levels, frozen deep-snapshot LogEvent values, explicit derived Logger context, early filtering and contained sink Error semantics published; maintainer reports all local tests PASS.
- [`LIB015/LIB015_B0_PLAIN_FORMATTING_TEXTWRITER_CONTRACT_DECISION.md`](LIB015/LIB015_B0_PLAIN_FORMATTING_TEXTWRITER_CONTRACT_DECISION.md)
  — owner-ratified LIB015-B0 formatter/sink contract: deterministic single-line plain grammar, Unicode-scalar key ordering, presence-only Error rendering, safe LogEvent recognition hardening, borrowed TextWriter sink awaiting `writeLine(...).value()`, minimal MemorySink, and LIB015-B1 released for implementation.
- [`LIB015/LIB015_B1_PLAIN_FORMATTER_EXPLICIT_SINKS_EVIDENCE.md`](LIB015/LIB015_B1_PLAIN_FORMATTER_EXPLICIT_SINKS_EVIDENCE.md)
  — LIB015-B1 implementation evidence at Protos `6252f3b6ddaffd2246e5f96eb2e746df8f1ff267` / `0.3.253-SNAPSHOT`: canonical LogEvent recognition hardening, deterministic plain TextFormatter, borrowed TextSink, and identity-preserving MemorySink published; maintainer reports `git diff --check` and all local tests PASS; focused LIB015-C0 JSON projection/formatter contract research is next.
- [`LIB015/LIB015_C0_JSON_EVENT_FORMATTER_CONTRACT_DECISION.md`](LIB015/LIB015_C0_JSON_EVENT_FORMATTER_CONTRACT_DECISION.md)
  — owner-ratified LIB015-C0 JSON contract: nested fields, exact Integer projection, strict Float policy, deterministic ordering, std:json ownership, and TextSink framing reuse.
- [`LIB015/LIB015_C1_JSON_FORMATTER_EVIDENCE.md`](LIB015/LIB015_C1_JSON_FORMATTER_EVIDENCE.md)
  — LIB015-C1 implementation evidence at Protos `ddf59b1b3f5c97758354688088845d641f7a4e04` / `0.3.257-SNAPSHOT`: deterministic compact JsonFormatter over std:json published; maintainer reports `git diff --check` and all local tests PASS.
- [`LIB015/LIB015_D0_TIMESTAMP_EXPLICIT_TIME_AUTHORITY_CONTRACT_DECISION.md`](LIB015/LIB015_D0_TIMESTAMP_EXPLICIT_TIME_AUTHORITY_CONTRACT_DECISION.md)
  — owner-ratified LIB015-D0 timestamp contract: one five-slot LogEvent family with always-present null-or-Instant timestamp, optional explicit Logger time-source callable after filtering, ISO8601 human prefix, exact Integer nanoseconds in JSON, and LIB015-D1 released.
- [`LIB015/LIB015_D1_TIMESTAMPED_LOGEVENT_TIME_SOURCE_EVIDENCE.md`](LIB015/LIB015_D1_TIMESTAMPED_LOGEVENT_TIME_SOURCE_EVIDENCE.md)
  — LIB015-D1 implementation evidence at Protos `915fa3739e3a0a6e7f0934e3975e79d3bb24eba8` / `0.3.260-SNAPSHOT`: five-slot timestamped LogEvent, explicit pay-as-you-grow Logger time source, ISO human timestamps and exact JSON nanoseconds published; maintainer reports `git diff --check` and all local tests PASS; parent remains open pending separately owned reusable terminal styling for the ratified color adapter.
- [`LIB015/LIB015_E0_COLORED_HUMAN_LOGGING_ADAPTER_CONTRACT_DECISION.md`](LIB015/LIB015_E0_COLORED_HUMAN_LOGGING_ADAPTER_CONTRACT_DECISION.md)
  — owner-ratified LIB015-E0 colored human logging contract at Protos `246a24994d74ca084dc43ceddd4c400c2af94b15`: `std:logging/ColoredFormatter(mode, autoEnabled)` returns a String formatter, reuses canonical `TextFormatter` grammar and unchanged `TextSink`, styles only the level token with the fixed TRACE blue / DEBUG cyan / INFO default / WARN yellow / ERROR red palette, and releases LIB015-E1.
- [`LIB015/LIB015_E1_COLORED_HUMAN_LOGGING_FORMATTER_EVIDENCE.md`](LIB015/LIB015_E1_COLORED_HUMAN_LOGGING_FORMATTER_EVIDENCE.md)
  — LIB015-E1 and parent closure evidence at Protos `d80536af4db3632d7268d370d216418c1c4fa533` / `0.3.271-SNAPSHOT`: `std:logging/ColoredFormatter` publishes the ratified pure level-color adapter over `TextFormatter`, `ColorMode`, `StyledText`, `ANSI`, and unchanged `TextSink`; maintainer reports `git diff --check` and all local tests PASS; LIB015 has no remaining approved slice.

- [`LIB020/LIB020_A_STYLE_STYLEDTEXT_SEMANTIC_VALUES_EVIDENCE.md`](LIB020/LIB020_A_STYLE_STYLEDTEXT_SEMANTIC_VALUES_EVIDENCE.md)
  — LIB020-A implementation evidence at Protos `c495b31dc21f71929ae460addad60613acb3d695` / `0.3.267-SNAPSHOT`: ratified `std:text/Style` and `std:text/StyledText` semantic values published with runtime-safe exact-family recognition, opaque sealed state, Style semantic equality/hash, StyledText flat opaque runs, and explicit Actor/parallel non-portability; maintainer reports `git diff --check` and all local tests PASS; LIB020-B is next.

- [`LIB020/LIB020_B_PURE_ANSI_RENDERER_EVIDENCE.md`](LIB020/LIB020_B_PURE_ANSI_RENDERER_EVIDENCE.md)
  — LIB020-B implementation evidence at Protos `b10569680b1532b277ba3ce49eced2e818833d93` / `0.3.268-SNAPSHOT`: pure `std:text/ANSI.render` published with exact Boolean policy input, deterministic reset-correct basic foreground SGR rendering, separate no-ANSI plain path, arbitrary-text pass-through, and private reuse of the existing Style/StyledText sealing facilities; maintainer reports `git diff --check` and all local tests PASS; LIB020-C is next.

- [`LIB020/LIB020_C_COLORMODE_AND_CLOSURE_EVIDENCE.md`](LIB020/LIB020_C_COLORMODE_AND_CLOSURE_EVIDENCE.md)
  — LIB020-C and parent closure evidence at Protos `246a24994d74ca084dc43ceddd4c400c2af94b15` / `0.3.269-SNAPSHOT`: pure canonical-String `std:text/ColorMode` AUTO/ALWAYS/NEVER resolution published with unconditional exact-Boolean validation and no ambient environment/TTY authority; maintainer reports `git diff --check` and all local tests PASS; the complete LIB020 A/B/C reusable styled-text foundation is closed and the LIB015 colored-human-output adapter blocker is resolved.

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

- [`AUD017/AUD017_TOML_IMPLEMENTATION_OWNERSHIP_AUDIT.md`](AUD017/AUD017_TOML_IMPLEMENTATION_OWNERSHIP_AUDIT.md)
  — completed Package Tool/public TOML ownership audit at Protos `61c475650bf5b5d385e9482b639d310c682e2b0d`: current `std:` bootstrap is independent of project package resolution, dual parser maintenance is materially duplicated and already drifting, and the owner selects one public `std:toml/TOML` implementation with explicit persisted-dialect pinning; private `tool-shared:Toml10` is routed for removal.

- [`AUD013/AUD013_A_ADOPTION_BASELINE_RECONCILIATION.md`](AUD013/AUD013_A_ADOPTION_BASELINE_RECONCILIATION.md)
  — AUD013-A adoption-baseline reconciliation: historical activation anchor retained, post-baseline owners classified, active TEST003/D142/D143/D147/D144 adoption manifest established, already-completed owner cutovers excluded from duplicate sweeps, and AUD013-B1 released as the first integrated source pass.

- [`AUD013/AUD013_B1_LIBRARY_TEST_ASSERTIONS_ADOPTION_EVIDENCE.md`](AUD013/AUD013_B1_LIBRARY_TEST_ASSERTIONS_ADOPTION_EVIDENCE.md)
  — AUD013-B1 implementation evidence at Protos `c8f61ae87ef6a26f8525b9d24f5ad466b665a29c`: 21 maintained library-test files migrated from TEST003-proven local assertion helpers to `std:test/Assertions`; maintainer validation passed, while parent AUD013 remains open because production library, tooling, conformance, examples and other maintained Protos surfaces still require integrated classification.
- [`AUD013/AUD013_B2_STANDARD_LIBRARY_ADOPTION_EVIDENCE.md`](AUD013/AUD013_B2_STANDARD_LIBRARY_ADOPTION_EVIDENCE.md)
  — AUD013-B2 implementation evidence at Protos `629161d5e2c6c6bc659a9f5b23998feda186b99e`: the complete Standard Library production-source pass produced three semantics-preserving current-Protos migrations (Integer/String recognizers and D143 multislot); maintainer validation passed, and all remaining maintained Protos surfaces are deliberately consolidated into one final large AUD013-B3 sweep and closure attempt.

- [`AUD007/AUD007_PARALLEL_PATCH_LAUNCHER_ISOLATION_AUDIT.md`](AUD007/AUD007_PARALLEL_PATCH_LAUNCHER_ISOLATION_AUDIT.md)
  — AUD007-A parallel launcher isolation/publication-safety audit: retains the GITHUB003 optimistic fail-closed publication model, inventories shared Git/build/runtime resources, identifies Maven multi-process, DAP port-allocation and untracked-candidate evidence gaps, and defines AUD007-B deterministic dynamic proof without repeating the long FULL run.
  - [`AUD007-B1 publication race and cleanup evidence`](AUD007/AUD007_B1_PUBLICATION_RACE_AND_CLEANUP_EVIDENCE.md) — deterministic local bare-remote proof for disjoint validation reuse, overlap/dependency fail-closed behavior, simultaneous publication, bounded retry, catchable interruption cleanup, SIGTERM cleanup and SIGKILL residue recovery.
  - [`AUD007-B2 final isolation closure`](AUD007/AUD007_B2_FINAL_ISOLATION_CLOSURE.md) — final closure at Protos `b2b132af1338e474857a0e1c9012f2c32f56e869`: isolates Maven writable state per validation with a shared read-only tail, removes DAP reserve-close-rebind port ownership, binds validation-observable untracked/ignored inputs to candidate evidence, retains B1 race/cleanup proofs, and leaves `AUD007_CLOSURE_READY=YES` with no further slice.

- [`AUD005/AUD005_E2_A_TEMPORAL_FRACTION_LINEAR_ENCODING_EVIDENCE.md`](AUD005/AUD005_E2_A_TEMPORAL_FRACTION_LINEAR_ENCODING_EVIDENCE.md)
  — AUD005/LIB010-E2-A evidence at exact Protos revision `62f3f5710210f247aad8574d3d0d56d3254bfd70`: temporal fraction encoding now uses one mutable octet buffer and one final decode, structurally restoring linear cost in emitted length; a 400-digit suite-native round-trip and a source-level anti-prepend guard retain F6, while F3, F4, F5 and final closure reconciliation remain open.

- [`AUD005/AUD005_E2_B_OFFICIAL_TOML_11_CONFORMANCE_RESEARCH.md`](AUD005/AUD005_E2_B_OFFICIAL_TOML_11_CONFORMANCE_RESEARCH.md)
  — AUD005/LIB010-E2-B research at Protos `62f3f5710210f247aad8574d3d0d56d3254bfd70`: freezes official `toml-test` v2.2.0 / TOML 1.1 provenance and case counts, classifies 11 byte-domain invalid fixtures as explicitly non-applicable to the ratified String-input API, and selects exact retained upstream fixtures plus a deterministic suite-native projection; F3 is implementation-ready but not yet resolved.

- [`AUD005/AUD005_E2_C_OFFICIAL_TOML_11_CONFORMANCE_EVIDENCE.md`](AUD005/AUD005_E2_C_OFFICIAL_TOML_11_CONFORMANCE_EVIDENCE.md)
  — AUD005/LIB010-E2-C implementation evidence at Protos `22bdcdc455cdaff3c4509e96555f28623d6254b7`: retains the pinned official TOML 1.1 corpus and deterministic 884-case suite-native projection, mechanically reconciles 11 non-UTF-8 byte-domain fixtures, and records the two parser conformance defects exposed and repaired; F3 is resolved while F4, F5, F7 and final closure validation remain open.

- [`AUD006/AUD006_A1_COMMAND_LINE_ACCUMULATION_COMPLEXITY_EVIDENCE.md`](AUD006/AUD006_A1_COMMAND_LINE_ACCUMULATION_COMPLEXITY_EVIDENCE.md)
  — AUD006-A1 static proof that the current `CommandLine` balanced chunk accumulation remains `Theta(N log N)`, plus the retained-evidence plan and platform/runtime repair boundary.
- [`AUD006/AUD006_A2_FULL_PARSE_SCALING_INSTRUMENT_LIMIT.md`](AUD006/AUD006_A2_FULL_PARSE_SCALING_INSTRUMENT_LIMIT.md)
  — AUD006-A2 negative evidence showing that full-parse allocation/timing scaling cannot discriminate the builder's `Theta(N log N)` term from dominant Truffle/interpreter cost, leaving isolated retained evidence blocked on the platform/runtime decision.
- [`AUD006/AUD006_B1_COMMAND_LINE_DEPTH_REMEDIATION_ANALYSIS.md`](AUD006/AUD006_B1_COMMAND_LINE_DEPTH_REMEDIATION_ANALYSIS.md)
  — AUD006-B1 depth/recursion analysis reconciled at Protos `62f3f5710210f247aad8574d3d0d56d3254bfd70`: command canonicalization and selected-child parsing remain execution-stack-depth proportional, both admit mechanical iterative linked-frame repair entirely in Protos, no public depth limit or new platform decision is required, and B2 → B3 is the implementation order.
- [`AUD006/AUD006_B2_AND_FINAL_CLOSURE_EVIDENCE.md`](AUD006/AUD006_B2_AND_FINAL_CLOSURE_EVIDENCE.md)
  — final AUD006 closure at Protos `698f90c3a7785fc9f51d2685ae6024b67b6ea1a2`: B2 removes CommandLine canonicalization stack-depth dependence and retains 4096-level/cycle/reuse evidence; maintainer validation passed, while selected-child parse recursion and the unexecuted documentation pass are explicitly accepted as non-blocking residual debt under owner-directed closure, with no further AUD006 slice.

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
