# AUD025 — Standard Library Protos-first implementation ownership

**Work item:** [AUD025 / #1209](https://github.com/guillermomolina/protos/issues/1209).  
**Date:** 2026-11-27.  
**Status:** open umbrella audit; phases allocated, no implementation completed by this record.  
**Product inspection revision:** [52591e7ce5c7e068f98101db94e77e68a64320da](https://github.com/guillermomolina/protos/tree/52591e7ce5c7e068f98101db94e77e68a64320da).  
**Prior project-record inspection revision:** [1989f40f3e4e794f111a6ae7b20c69491c97044f](https://github.com/guillermomolina/protos-project-docs/tree/1989f40f3e4e794f111a6ae7b20c69491c97044f).  
**Source-backed investigation:** [initial census and source references](https://github.com/guillermomolina/protos/issues/1209#issuecomment-7209250239).

## Owner-directed adjustment

The owner directed dividing the audit into independently actionable work items, moving the previous modularization phase F before A and retaining the remaining order, and removing success measurements. The original checkpoint's quantitative acceptance gates and original phase order are superseded. Attempt the migration where feasible, document a concrete reason when it is not, and move on. No Java line-count targets, native-entry quotas, benchmark campaigns, or mandatory removal targets. Normal applicable correctness and validation requirements for actual edits remain intact; no extra audit-specific metrics.

I denotes actual implementation within the established contract; PLAT denotes unresolved architecture, not authority to change product code. A PLAT may conclude that retaining a mechanism is appropriate. Only allocate a separate implementation if a viable, exactly owner-approved candidate warrants it. This instruction is not approval of any new substantive architecture or normative semantics.

## Initial investigation: family-level findings

The non-truncated revision-pinned tree contained 25 library families, including Core. TOML, JSON, URI, CSV, SemVer, integer math, datetime, collections, SHA256 and much of HTTP/TLS already own their domain algorithms in Protos. Splitting a large guest file improves cohesion but is not a Java migration. The inspected revision does not contain the owner's unpublished Markdown library.

| Family | Current source ownership / disposition to investigate |
|---|---|
| auth | Guest JWT; retain established secure backend. |
| cli | Guest parser/help with dedicated Java Array construction facility; candidate for removal. |
| collections | Guest algorithms; retain storage and runtime primitives; Array modularity. |
| core | Existing guest Path, IP and import policy; retain genuine bootstrap/runtime mechanisms. |
| crypto | Guest SHA256; preserve secure HMAC/Argon2id backends and contracts. |
| csv | Guest parser and writer. |
| datetime | Guest calendar calculations; retain host clock. |
| ffi | Guest declarations/projection; split cohesive responsibilities; retain generic native provider. |
| http | Modular guest protocol, framing, routing and sessions; retain required async bridge. |
| inspect | Runtime introspection is genuine host responsibility. |
| interop | Genuine foreign execution and access boundary. |
| io | Buffered reader/writer currently runtime-backed; investigate separable guest policy. |
| json | Guest engine; modularize nodes, parsing, encoding and events/adapters. |
| logging | Guest formatters/sinks; review native domain validation. |
| math | Guest integer algorithms; native Vector/Matrix algorithm boundary is a candidate. |
| network | Guest address/endpoint/DNS policy; preserve actual host transport and Unicode facts. |
| regex | Guest compiler/matcher; modularize and review native Unicode/transfer policy. |
| runtime | Runtime metadata genuinely native. |
| semver | Guest implementation. |
| sqlite | Guest ABI, statements and resource policy; generic FFI support. |
| test | Guest tests/assertions; retain host execution integration. |
| text | Guest Base64url and formatting; portable codec policy candidate; preserve host codecs and transfer. |
| tls | Guest session/options/identity policy; retain secure TLS engine and trust backend. |
| toml | Guest dialect-aware engine; modularize without duplicating parser. |
| uri | Guest parser/formatter/resolver; modularize. |

This census is evidence for prioritizing work, not a complete transitive call graph or proof that all candidate migrations can be implemented. The independently owned phases now investigate their precise boundaries incrementally.

## Historical reconciliation

- [AUD017 #804](https://github.com/guillermomolina/protos/issues/804) / I079: duplicate guest TOML engines consolidated. Preserve the single dialect-aware implementation.
- [AUD020 #872](https://github.com/guillermomolina/protos/issues/872) / I093: IP domain behavior moved to source and host NIO decoupled. Do not reopen indiscriminately.
- [AUD021 #874](https://github.com/guillermomolina/protos/issues/874) / I094: Path domain behavior moved into source; architectural boundary improvement did not imply net Java removal.
- [I128 #972](https://github.com/guillermomolina/protos/issues/972): residual HTTP private helper consolidation; HTTP is already predominantly guest-owned.

## Approved work order and allocations

| Order | Original phase | Issue | Kind and scope |
|---|---|---|---|
| 1 | F | [I186 #1210](https://github.com/guillermomolina/protos/issues/1210) | Implement cohesive modularization: TOML, JSON, Regex, CommandLine, FFI, URI and Array where warranted. Preserve exports, public identity, dialects, semantics and bootstrap. No forced file count. |
| 2 | A | [I187 #1211](https://github.com/guillermomolina/protos/issues/1211) | Implement feasible guest replacement of the dedicated ProtosCommandLineArrayConstructionFacility; remove it only when no longer needed. |
| 3 | B | [PLAT080 #1212](https://github.com/guillermomolina/protos/issues/1212) | Owner-approved retention of current native fundamental operations; prefer source for naturally composable higher-level algorithms. [Decision and final evidence](../../evidence/PLAT080/PLAT080_NATIVE_LINEAR_ALGEBRA_RETENTION_2026_11_27.md). No I required. |
| 4 | C | [PLAT081 #1213](https://github.com/guillermomolina/protos/issues/1213) | [PLAT081-A owner-approved disposition](../../decisions/platform/PLAT081_BUFFERED_BYTE_IO_SOURCE_OWNERSHIP_A_RATIFIED_2026_11_27.md): retain the existing operation-owned private runtime boundary and source-owned higher-level policy. No migration I required; independent static defect hypotheses are recorded without implementation allocation. |
| 5 | D | [PLAT082 #1214](https://github.com/guillermomolina/protos/issues/1214) | Decide portable encoding and text I/O boundary, including ProtosEncodingValue, line framing and host codec compatibility under TEXT_IO. |
| 6 | E | [I188 #1215](https://github.com/guillermomolina/protos/issues/1215) | Implement bounded feasible removal of redundant Java domain policy in ProtosLoggingFacility and Regex helpers without changing security, exact recognition, Unicode or transfer contracts. |

The ordering is owner direction, not a hard dependency chain or explicit priority assignment. Live lifecycle authority remains in GitHub Issues/Project. The parent stays open until the dispositions of these phases and warranted approved implementation follow-through are resolved; progressively reconcile its checklist.

## Invariants

Retain public semantics and ordinary message syntax with specialized optimizable hot paths, isolation, mutability, suspension, exact error behavior, resource custody, and Native Image compatibility. Do not invent production Java infrastructure; any necessary production Java change needs an exact method/annotation/behavior equivalent in GraalPy or another established Truffle language. Guest primitives remain long/double/boolean. No public API or substantive architecture change without exact owner approval. Do not add a different Java helper solely to remove an existing one. Do not treat this plan as a normative or architectural ratification.

## Coordination and evidence

All six allocated identifiers were re-searched and found unique; the live issues were re-read open with matching family labels, status:ready, no assignee, and priority intentionally unset. The native parent endpoint of each child returned AUD025 #1209, and the parent's native sub-issues endpoint returned all six. Project projection could not be verified using the available connector (project_items endpoint returned 404); leave this postcondition unverified rather than infer it from labels.

The owner explicitly authorized direct publication to the project-record repository for this task. No Protos source, test, specification or version files were changed. No builds, tests, benchmarks or local validation commands were executed. The initial census consulted root and scoped AGENTS instructions, the blocker ledger, relevant normative owners and previous work records; further exact feasibility evidence belongs to the individual phases. This record is non-normative.

## I186 — Phase F implementation evidence (2026-11-27)

- Work item: [I186 #1210](https://github.com/guillermomolina/protos/issues/1210), child of [AUD025 #1209](https://github.com/guillermomolina/protos/issues/1209).
- Product revision: [`5f2b40910f09f163e2c2987a4731c0d0d499b373`](https://github.com/guillermomolina/protos/commit/5f2b40910f09f163e2c2987a4731c0d0d499b373), confirmed on `main` at inspection; implementation version `0.4.253-SNAPSHOT`.
- Published commit: `I186: Modularize Standard Library implementations into private Protos SOURCE components`.
- Source changes: canonical public module entry points retained; private SOURCE components separate CommandLine specification, normalization, parsing and help; Array traversal, editing, ordering and parallel operations; FFI descriptors, layout, native types, bindings, memory and projection; JSON nodes, lexical/number algorithms, tree/event parsing, encoding and reader/writer adapters; Regex compilation, matching, captures, ranges, Unicode and replacement; TOML document, key, value, numeric, temporal and string parsing plus encoding; URI parsing, formatting, authority, components and resolution.
- JSON tree and event parsers share lexical and exact-number algorithms while retaining distinct parsing behavior. The published changelog records preserved ordinary public-member selection, invocation-local parser/encoder state, private Actor-local facilities, lazy FFI provider acquisition and existing isolated-parallel Array execution. Regression changes cover public operation selection, temporal constructor selection, return behavior and private implementation boundaries. The inspected commit file list contains Protos source/tests plus version and changelog changes, not production Java or normative specification changes.
- Validation provenance: maintainer explicitly reported a clean `git diff --check` and all tests passing locally, followed by publication confirmation. No raw test log, exact command list, independent CI result, or Native Image-specific result was supplied; those are not independently claimed. Product tests were not rerun by the coordinating agent.
- Disposition: Phase F implemented and maintainer-validated. This is modularization, not evidence that the later Java ownership migrations are complete. No new architecture, acceptance metrics, issues, or dependency edges follow from this publication. Remaining AUD025 phases retain their own dispositions.

## PLAT080 — Phase B retention evidence (2026-11-27)

Owner explicitly approved justified retention after the source-backed feasibility investigation at [5f2b40910f09f163e2c2987a4731c0d0d499b373](https://github.com/guillermomolina/protos/tree/5f2b40910f09f163e2c2987a4731c0d0d499b373). The [maintained decision](../../decisions/platform/PLAT080_D_SELECTIVE_PROTOS_FIRST_LINEAR_ALGEBRA.md) and [final comparative evidence](../../evidence/PLAT080/PLAT080_NATIVE_LINEAR_ALGEBRA_RETENTION_2026_11_27.md) supersede the conditional checkpoint: preserve existing native storage, construction, indexed access, arithmetic and fundamental operations; no Array intermediate or cosmetic migration. Prefer Protos for naturally composable higher-level mathematical algorithms. Phase B disposition complete without an I; no product edits or tests. Other phases unchanged; live closure/status authority remains GitHub.
