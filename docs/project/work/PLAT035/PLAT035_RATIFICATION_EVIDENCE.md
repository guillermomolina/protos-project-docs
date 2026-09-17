# PLAT035 — Ratification evidence

Status: `RATIFIED`

Decision Issue: `guillermomolina/protos#551`

Project-owner approval: **explicitly given on 2026-09-17**

Selected architecture: **Candidate C — Bytecode DSL as the sole target executable backend; the legacy executable Truffle AST is bounded migration scaffolding only and must be removed after its audited dependency inventory reaches zero.**

Research / ratification source baseline:
`5529fa515016f6a44fb15934c4691e162c417d73`

Canonical durable decision record:

`docs/project/decisions/platform/PLAT035_SINGLE_EXECUTION_BACKEND_LEGACY_AST_RETIREMENT.md`

The earlier decision packet in this directory was produced before owner approval and remains research/history only. Any `OPEN` / `NEEDS PROJECT-OWNER DECISION` status inside that packet is superseded by this ratification evidence and the canonical decision record above.

Invariant/delta consistency review: **PASS**

```text
TARGET_EXECUTION_BACKEND=TRUFFLE_BYTECODE_DSL_ONLY
LEGACY_AST=TRANSITIONAL
NEW_LEGACY_DEPENDENCIES=FORBIDDEN
END_STATE=LEGACY_EXECUTION_BACKEND_REMOVED
PLAT014=PASS
SURFACE_AST=UNCHANGED
CANONICAL_AST=UNCHANGED
TEST002_AUTHORITY=UNCHANGED
JAVA_TEST_OWNERSHIP=UNCHANGED
TOOLING_CONTRACTS=UNCHANGED
PROTOS_VISIBLE_SEMANTICS=UNCHANGED
```

Immediate downstream consequence:

AUD012 is now a retirement-routing audit. An intentionally Java-owned or deliberately unhosted test may remain Java-owned/unhosted, but that no longer justifies permanent use of `CanonicalToTruffleLowerer` / the executable `ProtosExpressionNode` backend. Executable guest source must converge on a Bytecode-backed boundary.
