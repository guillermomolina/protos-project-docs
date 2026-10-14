# AUD020 / I093 — Owner-directed final disposition (2026-10-14)

**Record role:** non-normative, owner-directed scope-closure record. This is not a claim that every initial audit criterion was exhaustively verified, that Native TCP-with-Network was exercised, or that performance/Native Image costs were measured.

- **Owner instruction (2026-10-14):** further Native Image measurements are not necessary or proportionate to the problems currently being addressed. Explicit direction to **close both existing I093 and AUD020 issues**, with no additional Native Image measurement campaign.
- **I093 implementation owner:** [guillermomolina/protos#873](https://github.com/guillermomolina/protos/issues/873). Close as **completed within the owner-accepted final scope**.
- **AUD020 independent audit:** [guillermomolina/protos#872](https://github.com/guillermomolina/protos/issues/872). Close as **not planned for further investigation**: superseded by the implemented I093 boundary, the source review and this explicit stop decision. Do not imply its originally proposed exhaustive audit protocol ran to completion.
- **PROTOS_REVISION:** `8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff` ([published product commit](https://github.com/guillermomolina/protos/commit/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff)).
- **Implementation version:** `0.3.356-SNAPSHOT`.
- **Product changes in final outcome:** I093-A (Protos-source IP protocol and private codec), B1 (descriptor-only connect), B2 (descriptor-only listen/accept), and C1 (Actor termination queued-resource discard custody). The final accepted product state is the published C1 revision above.
- **Previous durable records:** [B1](../../evidence/I093/I093_B1_TCP_CONNECT_HOST_DESCRIPTOR_ACTOR_COMPLETION_PUBLICATION_2026_10_14.md), [B2](../../evidence/I093/I093_B2_TCP_LISTEN_ACCEPT_HOST_DESCRIPTOR_ACTOR_COMPLETION_PUBLICATION_2026_10_14.md), [C1](../../evidence/I093/I093_C1_ACTOR_TERMINATION_DISCARD_CUSTODY_PUBLICATION_2026_10_14.md), and [C source-coupling census and limitations](../../evidence/I093/I093_C_ARCHITECTURE_SOURCE_CENSUS_AND_ACCEPTANCE_GAPS_2026_10_14.md). The historical limitations in those records remain accurate descriptions of what was and was not verified at publication time.

## Accepted architectural result

1. Canonical IPv4/IPv6 `IpAddress` and `IpEndpoint` construction, structural equality/hash, parsing/formatting and private source-owned octet handling no longer need to be reimplemented throughout NIO.
2. Guest/network adapters authenticate canonical input without candidate callbacks, capture immutable 4-/16-octet descriptors and later materialize guest results only in the owning Actor.
3. Host NIO connect, listen, accept and transport operate without direct `ProtosObjectValue`, `ProtosNumericValueSupport`, guest endpoint prototype or guest integer-family dependencies in the principal backend classes. The published pre-B1→C1 source-token census in the earlier architectural record demonstrates the bounded *source separation*, not a runtime speedup.
4. Actor-targeted delivery and claim-once host resource custody include C1's additional `onDiscard` cleanup for the corner case of a resource-bearing queued completion dropped during Actor termination after an earlier post-commit cancellation request.
5. Existing authority/no-ambient-Network and no-eager-poller restrictions remain governed by current specifications. Nothing in this closure changes D172, D173, I090, I091 or the public TCP/Network semantics.

## Validation and explicit owner-accepted limitations

**Human reported:** focused C1 Java acquisition tests **29 PASS**; `make test` PASS; `make check` PASS; Native Image build/test PASS; native IP-address 10 PASS, native IP-endpoint 8 PASS and native endpoint parse/format 2 PASS, totaling **20 Native IP data cases**; `git diff --check` clean. These are human-reported execution results on the code candidate. A later `chaca` invocation printed `bash: mvn: orden no encontrada`; it did **not** run Maven tests and does not contradict or replace the earlier reported runs.

**Explicitly NOT_VERIFIED / NOT_MEASURED, accepted for issue closure by the owner:**

- No additional Native Image performance, allocation, timing or image-footprint measurement is required for I093/AUD020; this is explicitly out of the current work scope by owner direction.
- Authorized Native Image TCP `connect/listen/accept` end-to-end execution was not demonstrated by the reported Native IP data cases and is not retrospectively claimed.
- Version-matched before/after allocation and throughput measurements for the network refactor are unavailable; direct source-token reductions are not performance measurements.
- The full originally proposed AUD020 comparative twelve-criterion/exhaustive audit protocol and all possible Actor/P/Context/scope interleavings were not established as completely executed; the owner elects to stop this audit rather than pursue increasingly high-level probes.

The closure is a **scope and priority decision**, not a failed-test waiver or proof that no additional bugs could exist. Newly observed concrete defects can be reported through the ordinary issue process when supported by fresh evidence; they are not implicit blockers keeping these two issues open.

## Issue disposition

- **I093/#873:** implementation delivered and sufficiently validated within current owner-accepted scope. Close `completed`.
- **AUD020/#872:** investigation completed to the useful architecture decision and related I093 implementation; remaining exhaustive/higher-level work intentionally discontinued. Close `not_planned` rather than claiming the original audit matrix completely executed.
- **No new Issues, releases, tags, specification revisions, project priority or dependencies** are allocated by this record.

```text
OWNER_DIRECTED_SCOPE_CLOSURE=APPROVED_2026_10_14
PROTOS_REVISION=8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff
IMPLEMENTATION_VERSION=0.3.356-SNAPSHOT
I093_A=PUBLISHED
I093_B1=PUBLISHED
I093_B2=PUBLISHED
I093_C1=PUBLISHED
I093_FINAL_DISPOSITION=CLOSE_COMPLETED_OWNER_ACCEPTED_SCOPE
AUD020_FINAL_DISPOSITION=CLOSE_NOT_PLANNED_FURTHER_AUDIT
PRODUCT_MAKE_TEST=PASS_HUMAN_REPORTED
PRODUCT_MAKE_CHECK=PASS_HUMAN_REPORTED
NATIVE_GATE=PASS_HUMAN_REPORTED
NATIVE_IP_DATA_CASES=20_PASS_HUMAN_REPORTED
NATIVE_TCP_WITH_NETWORK=NOT_VERIFIED
NATIVE_PERFORMANCE_MEASUREMENTS=OUT_OF_SCOPE_OWNER_DIRECTED
NETWORK_COST_MEASUREMENTS=NOT_MEASURED
AUD020_EXHAUSTIVE_AUDIT=NOT_FULLY_EXECUTED_OWNER_STOPPED
PUBLIC_SEMANTIC_CHANGE=NONE
```
