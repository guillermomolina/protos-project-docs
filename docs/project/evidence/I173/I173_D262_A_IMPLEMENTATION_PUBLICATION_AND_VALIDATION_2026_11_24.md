# I173 / D262-A — Implementation publication and observed validation

Date: 2026-11-24

## Stable identities

- PROTOS_REVISION=8c3be22391db29bd4ff1abd102884d07d3b2a50e: [published implementation](https://github.com/guillermomolina/protos/commit/8c3be22391db29bd4ff1abd102884d07d3b2a50e), parent c98be4618fa5e8dc7a89f5092d3712616803e501.
- Implementation version: 0.4.194-SNAPSHOT; specification revision: 0.1.508.
- [I173 #1164](https://github.com/guillermomolina/protos/issues/1164); native parent [D262 #1163](https://github.com/guillermomolina/protos/issues/1163), verified through both GitHub parent and sub-issues endpoints on 2026-11-24.
- Exact owner selection "apruebo A" is preserved in the [selected decision record](https://github.com/guillermomolina/protos-project-docs/blob/e5aa2ed6f792449d5bb6532818eb7859fbf5b9bb/docs/project/decisions/language/D262_A_REFLECTIVE_LOCAL_SLOT_CREATION_AND_UPDATE_SELECTED_2026_11_24.md) and [comparative evidence](https://github.com/guillermomolina/protos-project-docs/blob/b8ed47535bed22594fd11fb5ccd026ae1da070a6/docs/project/evidence/D262/D262_A_GITHUB010_GITHUB021_OWNER_SELECTION_AND_SOURCE_AUDIT_2026_11_24.md).
- Maintainer publication report: "I173: Add reflective Object.createSlot and updateSlot for dynamic local slot names pushed. el git diff check esta limpio. todos los tests pasan en local."

## Published delta (verified by reading exact product commit)

- [Object.protos](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/protos/lib/core/Object.protos): two source-backed public Closures with public name/value formals, missing-argument defaults and excess positional checks. Ordinary named association; exact supplied null remains admissible.
- Production Java changes are limited to existing ProtosStandardObjectProtocol.java (two private-bootstrap primitive bodies, explicit target, semantic String and receiver checks, delegation to createLocalSlot/assignLocalSlot and conversion of structural IllegalStateException to existing generic guest Error) and ProtosCoreBootstrap.java (private captured native primitives, existing source publication, Context-local executable projection and frozen shared graph). No new production Java class, parser, Bytecode DSL, lexical authority, storage, special dispatch or static-slot route change.
- [OBJECT_MODEL.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/OBJECT_MODEL.md), [EXECUTION_AND_CONTROL.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/EXECUTION_AND_CONTROL.md), [specification changelog](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/PROTOS_SPEC_CHANGELOG.md): normative creation/update contract, exact Unicode name semantics, local membership and state, exact result, ordinary argument association and control transfers, live lexical membership and explicit target versus bare assignment.
- Published diff contains 21 paths, including implementation version/changelog, guest conformance, SOURCE fixture and manifest, five Java test classes and two existing architecture/completeness test files. Native-boundary inventory adjusts private Core native Closure count 151 to 153 (bootstrap 1 to 3); repository completeness recognizes the new SOURCE fixture. No native provider or discovery infrastructure changes.
- D179 C0/E: existing structural authority remains the only mutation authority; declared frame local/cell and lazy overflow are unchanged. Focal tests cover physical binding preservation, removal/recreation, near-owner selection, reparse, suspension and unchanged direct local instructions. D242 private/public MODULE behavior is covered through existing MemoryResolver.publicModule fixture; SOURCE tests use existing direct-file fixture routes. Actor transfer/aliasing and host read-only interoperability are covered. No FFM dependency or D261 semantic change.
- External implementation precedent used in the editing plan: GraalPy BuiltinFunctions.SetAttrNode.setAttr (@Builtin, @GenerateNodeFactory, @Specialization, @Cached PyObjectSetAttrO), inspected blob 3dc09955f87b201b9f8750d5c076dcbbfc03c039; WriteAttributeToPythonObjectNode.write (@GenerateUncached, @Specialization, @Cached DynamicObject.PutNode), blob ba036bf121448f4818c055b436ef22ac47533fc9; WriteAttributeToObjectNode storage/receiver admission specializations, blob ba1c5b3cc19b80a858849b31a692b09530151dd1; PythonBuiltins.initializeEachFactoryWith/addFunctionsToModuleObject and existing @Builtin metadata, blob 0a5b3c4b47a20be48cfdbba88a06aa1eddf70de8. Reuse established Protos bridges and authority, not Python upsert or a new native dispatcher. These are mechanism-level precedents; no claim of identical guest semantics.

## Observed focal evidence

Maintainer-supplied transcript dated 2026-11-24, 08:44:04 UTC: Maven build 0.4.193-SNAPSHOT, before final version/spec wording and subsequent integrated candidate fixes, BUILD SUCCESS, 34.634 s. Compiled main and test sources. Command:

    mvn -Dtest=ProtosCoreBootstrapTest,ProtosI115PublishedLexicalContextTest,ProtosPerf025D179LexicalMembershipStabilityReadTest,ProtosObjectInteropTest,ProtosModuleRuntimeTest test

| Java class | Passed |
|---|---:|
| ProtosModuleRuntimeTest | 25/25 |
| ProtosI115PublishedLexicalContextTest | 36/36 |
| ProtosCoreBootstrapTest | 10/10 |
| ProtosPerf025D179LexicalMembershipStabilityReadTest | 7/7 |
| ProtosObjectInteropTest | 5/5 |
| Total | 83/83 |

Zero failures/errors/skipped. Guest commands were bin/protos test --file with the following paths:

| Guest path relative to protos/tests/conformance/ | Passed |
|---|---:|
| reflection/create-and-update-slot.protos | 58/58 |
| execution-context/reflective-slot-creation-and-update.protos | 24/24 |
| modules/module-source-separation.protos | 18/18 |
| actor/lifecycle-and-transfer.protos | 6/6 |
| Total | 106/106 |

Combined focal evidence: 189/189, zero failures. Do not attribute these exact counts to a later full-suite run. The final maintainer report states that git diff check is clean and all tests pass locally after the final edits and push; accepted as maintainer-reported local validation, not independently executed or an exact full-suite transcript. The final commit includes the architecture and completeness adjustments not present in the focal transcript.

## Gate reconciliation

- PRODUCT_PUBLICATION=VERIFIED; SPEC_NORMATIVE_PUBLICATION=VERIFIED; DECISION_APPROVAL_PROVENANCE=PASS; NATIVE_PARENT=PASS.
- FOCAL_COMPILATION=PASS; FOCAL_TESTS=PASS (observed transcript above); FINAL_LOCAL_TESTS=PASS (maintainer-reported); PRODUCT_DIFF_CHECK=PASS (maintainer-reported).
- Integrated make test is the required profile; exact final command, counts and native-lane results were not supplied separately. No standalone Native Image build/runtime-compilation result is asserted.
- STATIC_DIRECT_INSTRUCTION_REGRESSION=PASS (focal tests); DYNAMIC_NAME_CARDINALITY_FUNCTIONAL=PASS (100 names, reflected update and enumeration test). TIER1_TIER2_COMPARATIVE_MEASUREMENTS=NOT_REPORTED; no quantitative no-regression claim.
- Targeted published-delta consistency with D179, D242, frozen root, ordinary reflection/dispatch and transfer is supported by source and tests; the prior D262 whole-document/exhaustive optimized-path audit was explicitly pending and has not been independently completed by this publication record. Do not retroactively label it PASS.
- This is durable evidence of delivered implementation and normative publication, not a new design approval, independent performance validation or assertion that every formal closure gate has passed. Remaining explicit acceptance gates must be reconciled before formal closure.

No agent-executed builds/tests or product modifications in this coordination step.
