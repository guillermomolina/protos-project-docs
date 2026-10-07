# LM011-B2 / TOOL010 — formal formatter allocation

## Scope

This record preserves the project-owner allocation that releases the LM011-B2
canonical formatter implementation from its TOOL identifier precondition.

The live coordination authority remains `guillermomolina/protos#670`
(**LM011 — Canonical source formatter and editor formatting integration**).
This record is non-normative project evidence and does not itself define Protos
language semantics or formatter policy.

## Formal allocation

On 2026-10-04, in the active implementation interaction, the project owner
explicitly assigned:

```text
TOOL010=Canonical Source Formatter
OWNING_SLICE=LM011-B2
FORMAL_WORK_ITEM=LM011/#670
NEW_FORMAL_ISSUE_REQUIRED=NO
IMPLEMENTATION_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
ALLOCATION_STATUS=FORMALLY_ASSIGNED
IMPLEMENTATION_AUTHORIZED=YES
```

This is the exact allocation required by the LM011-B2 implementation
precondition. TOOL010 is owned inside LM011-B2 rather than by a separate GitHub
Issue, consistent with the existing LM011 coordination state.

## Governing authorities

The allocation does not reopen or modify the already-ratified authorities:

```text
D183_STATUS=RATIFIED
D183_SELECTED_CANDIDATE=B_FIXED_STRUCTURAL_STYLE_PLUS_CONSERVATIVE_LEXICAL_PRESERVATION

PLAT050_STATUS=RATIFIED
PLAT050_SELECTED_CANDIDATE=F_ON_DEMAND_HYBRID_PLUS_BUNDLED_PROTOS_POLICY
```

Therefore TOOL010 means the exact toolchain-bundled Protos formatter policy
selected by PLAT050. Host code remains limited to tool-neutral source/parser
mechanisms; formatter policy remains Protos-owned.

## Baseline

At allocation time the published product baseline is:

```text
PROTOS_REVISION=d49361af6fcda2815b89e9ae9ff22bc766ecf45d
PROTOS_VERSION=0.3.190-SNAPSHOT
LM011_B1=COMPLETE
```

LM011-B1 supplies the on-demand exact-source / token / Surface-AST / trivia /
parser-source-facts / comment-attachment / preservation-projection foundation.

The product repository may advance concurrently after this allocation. LM011-B2
implementation must therefore continue from the maintainer's real current HEAD
and preserve unrelated later work.

## Boundary

This allocation authorizes LM011-B2 implementation only. It does not authorize:

```text
public formatter CLI spelling
LSP textDocument/formatting
VS Code formatting integration
range formatting
check mode
on-type formatting
style configuration
parser recovery
full CST/lossless syntax layer
host-owned formatter policy
```

Those remain governed by the existing LM011/D183/PLAT050 decomposition.

AI assistance: this allocation evidence was drafted with ChatGPT from the
project-owner's explicit TOOL010 assignment, the live LM011 coordination state,
and the published LM011-B1 evidence.
