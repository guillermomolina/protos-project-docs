# I176 — Contextual match publication and validation evidence

Date: 2026-11-25

- Owning issue: [I176 #1173](https://github.com/guillermomolina/protos/issues/1173).
- Ratified design: [D263-C](https://github.com/guillermomolina/protos-project-docs/blob/dce1659fef6e0795f1fbc9fb2a5ef38feae688db/docs/project/decisions/language/D263_CONTEXTUAL_MATCH_IMPLICIT_LAZY_ARM_CLOSURES_C_RATIFIED.md).
- PROTOS_REVISION: [a37f7703216b6e6b80bebd59f0dd92797d692213](https://github.com/guillermomolina/protos/commit/a37f7703216b6e6b80bebd59f0dd92797d692213).
- Published implementation version: `0.4.223-SNAPSHOT` (verified in the revision's pom.xml and implementation CHANGELOG).
- BUG040 predecessor: [efb2dd5dcc56b0d9b3752dde1d739ae672897946](https://github.com/guillermomolina/protos/commit/efb2dd5dcc56b0d9b3752dde1d739ae672897946); [revision-bound defect evidence](https://github.com/guillermomolina/protos-project-docs/blob/d4b744845ef1784a3780734261848f9872ec3cca/docs/project/evidence/BUG040/2026-11-25-maintainer-validation.md).

## Published scope

The exact product commit adds D263-C contextual match syntax, reusing the ordinary Map entry parser, ordinary caseOf send and implicit parameterless lexical Closures for non-Closure actions. Explicit Closure literals remain unchanged. It adds surface/source layout and tooling coverage, stable entry-owned deferred static scopes, shared Map and contextual table formatting, token-boundary comment ownership and the Protos-side preservation vocabulary. No new canonical matching node, bytecode operation or matching runtime mechanism is included in the changed-path inventory. Existing explicit caseOf remains supported.

The changed-path inventory contains parser, canonicalization, static analysis, layout, formatter, conformance, LSP and associated tests; the three governing normative documents and the control-flow, matching and source-style guides; pom.xml and the implementation CHANGELOG.

## Observed validation

The supplied local validation transcript identifies the pre-publication checkout as 0.4.221-SNAPSHOT. The following results are observed in that transcript, not a CI run against the published commit. All focal runs returned BUILD SUCCESS, with no failures, errors or skips.

| Test class | Passed |
|---|---:|
| ProtosSourceLayoutViewTest | 47 |
| ProtosFormatterToolBootstrapTest | 59 |
| ProtosLanguageServerFormattingTest | 20 |
| ProtosStaticDefinitionsTest | 119 |
| ProtosStaticHoverTest | 20 |
| ProtosStaticLintTest | 21 |
| ProtosStaticReferencesTest | 29 |
| ProtosLanguageServerDefinitionTest | 20 |
| ProtosLanguageServerDocumentSymbolsTest | 6 |
| ProtosParserMatchTest | 18 |
| ProtosParserSourceFactsTest | 27 |
| ProtosI176ContextualMatchCanonicalizationTest | 18 |
| ProtosHostEntryFutureSuspensionTest | 19 |
| ProtosCaseOfProtocolTest | 19 |
| **Total** | **442** |

- git diff --check HEAD: clean, as shown in the transcript.
- mvn -o test-compile: BUILD SUCCESS, 42.804 seconds; compilation finished 09:30:57 UTC on 2026-11-25. Nonfatal warnings were present.
- Focal groups: 126 formatter/layout/LSP tests, 278 parser/static-analysis tests, 38 protocol/suspension tests.
- The maintainer subsequently explicitly reported the published I176 commit, clean git diff --check and all local tests passing. The aggregate full-suite count, log and run identity were not supplied; the aggregate result is maintainer-reported PASS, not independently executed here.

## Publication reconciliation and outstanding boundary

The product commit and implementation version/changelog are verified. The global normative changelog at that exact product revision still begins at 0.1.515 (D261-R2 / I174-G) and contains no D263-C entry. The current visible main changelog also remains at 0.1.515 without that entry. The I176 changed-path list does not include spec/PROTOS_SPEC_CHANGELOG.md, although its grammar, language specification and matching semantics changes are present. Under spec/AGENTS.md this is a publication-metadata omission requiring a human-published reconciliation using the next revision derived from then-current main. Do not infer a completed global specification revision or formal closure from the implementation commit alone. Preserve the published implementation and tests; the missing record is not a request to change semantics or bump the implementation version again.

The shared Map/Object rendering defect was closed independently as BUG040. Guest-policy comparison coverage is not production whole-document acceptance integration: [I177 #1177](https://github.com/guillermomolina/protos/issues/1177) owns that separate work. [PERF044 #883](https://github.com/guillermomolina/protos/issues/883) owns shared matching performance specialization; no performance claim is made here. Additional instructional material and selective first-party adoption are tracked separately.

This evidence record retains the reported local validation and verified exact product revision; it does not assert I176 formal closure before the required specification-changelog reconciliation.
