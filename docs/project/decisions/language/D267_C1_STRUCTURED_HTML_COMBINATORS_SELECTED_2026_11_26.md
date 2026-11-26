# D267-C1 — Structured HTML combinators: owner-selected architecture

Date: 2026-11-26  
Design owner: [D267 #1195](https://github.com/guillermomolina/protos/issues/1195)  
Implementation owner: [I179 #1198](https://github.com/guillermomolina/protos/issues/1198)  
Motivating consumer: [WEB016](https://github.com/guillermomolina/protos-website/issues/12)  
Inspected product baseline: [1afc4ceeb4b6df9878fb2d2f168235277f8e7e70](https://github.com/guillermomolina/protos/tree/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70)

**Status: C1 architecture explicitly selected by the owner; exact public contract and normative reconciliation pending.** This is a non-normative decision record, not a declaration that all of D267 has been ratified or implemented.

## Approval provenance

In the D267 investigation on 2026-11-26 the owner objected to verbose generic element construction, comparing it with Python Dominate's concise construction. The subsequent comparison distinguished pure value-returning combinators (C1), explicit Canvas (C2), implicit construction context (C3), and a hybrid (C4). The owner then explicitly approved C1: "ok aprubeo C1."

The owner selected the combinator architecture, including concise tag-specific construction, automatic textual escaping and ordinary structured composition, not an exact module path, selector spelling, attribute encoding, mutation policy, or a full serialization/security contract. No approval of Canvas, a new syntax, ambient construction state, or an HTML parser is inferred. The owner also explicitly authorized this bounded durable-record publication in this repository.

## Selected architecture

Common construction must be concise; for example, illustrative ordinary bindings to exported constructor behavior allow:

```protos
div(
    h1(title),
    p(description)
)
```

These calls construct ordinary composable nodes with ordered children, rather than appending to an ambient parent. Ordinary Strings used as children denote text, not executable markup; one serializer owns context-correct escaping. Components are ordinary Closures. The underlying generic element representation is not required boilerplate at every call site. A reusable HTML representation allows the D266 Markdown projection to compose with HTML application components without a String-as-trusted-HTML bypass.

The precise public topology is pending; the example is not an implemented or ratified API. No new Core semantic family, grammar, special Closure invocation, DOM, template language, Java infrastructure, global construction stack, or implicit authority is required. C2/C4 Canvas is deferred; C3 implicit context is not selected. D267's earlier D recommendation is superseded at the architectural-selection level by this C1 direction, retaining the useful distinction between escaping, projection and consumer policy.

## Invariants and boundaries

- Preserve ordinary Closure capture, receiver, argument evaluation, name association and rest semantics (D237); arbitrary named HTML attributes cannot silently become an undeclared named-argument Map.
- Preserve exact String identity, Unicode scalar semantics, explicit Encoding, Actor-local state and normal Error/control behavior. No privileged SafeString or trust propagation.
- Separate text and attribute escaping, structural/name/context validation, and URL/content admission. An escaped URL is not necessarily safe. Preserve std:uri RFC 3986 authority; do not assume WHATWG equivalence.
- Markdown parsing remains D266's responsibility; its projection must not execute source HTML or introduce a second parser. WEB016 keeps catalogue identity, routing and HTTP/S allowlisting, inert raw HTML and image handling. D268 XML remains a distinct syntax/serialization contract.
- No network, filesystem or other authority acquired by ordinary rendering. No claim that a caller-provided policy is intrinsically pure.
- Output and resource limits, failure behavior, cycles/aliasing and deterministic serialization need explicit definition before implementation.

## Remaining decision checkpoints

Exact node and fragment contract (including ownership, mutability, validity, child admission and cycles); tag constructor surface and module topology; attribute form, ordering, duplicates, boolean/void elements and active contexts; escaping and Unicode/control policy; link/content validation and safe Markdown projection integration; deterministic output/work limits and failure/commitment semantics. Decide these explicitly under D267 before production implementation in I179; do not treat illustrative examples as final API. Reconcile normative Standard Library material and relevant tests/guide in the owning product repository after approval.

See [comparative research and approval evidence](../../evidence/D267/D267_C1_COMPARATIVE_RESEARCH_AND_OWNER_APPROVAL_2026_11_26.md). No product source changes, tests, builds, version changes or runtime validation are claimed.
