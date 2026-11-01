# DOC014 — Executable bitwise learning material and guide navigation

**Work family:** DOC (one cohesive documentation implementation slice).
**Live issue:** [DOC014 #1020](https://github.com/guillermomolina/protos/issues/1020).
**Product edit/validation/commit owner:** `guillermomolina/protos`, local current HEAD.
**Design provenance:** [D233-B1 #1016](https://github.com/guillermomolina/protos/issues/1016), ratified; [I137 #1017](https://github.com/guillermomolina/protos/issues/1017), implemented and closed.
**Verified already-published product baseline:** [`guillermomolina/protos@c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5`](https://github.com/guillermomolina/protos/commit/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5), `0.3.431-SNAPSHOT`, spec `0.1.486`.
**Status at handoff:** planning/agent and human-executor prompts **published in this docs repository**; DOC014 product content **not yet edited, tested, or published**. This record is neither an approval for new semantics nor product publication evidence.

## Investigation floor and precise gap

DOC013 #1003 was completed on the prior documentation baseline. I137 subsequently introduced exact source syntax and added one reference subsection in the maintained numeric guide, alongside grammar/spec, source-style policy, registered conformance tests and private SHA-256 helpers. The current user-facing learning surfaces still have no dedicated beginner bitwise tutorial in `protos/tutorials/01-values-and-slots/`, no bit-flags/masks cookbook entry in `protos/examples/numbers/`, and no progression/cookbook index entries linking such lessons. This bounded gap needs a single DOC issue, not another language design/implementation issue and not a complete DOC013 rescan.

Reviewed project facts:
- `AGENTS.md`, `AGENTS.work/DOCUMENTATION.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/IMPLEMENTATION.md` validation/version rules, `AGENTS.work/REFERENCE.md` APL and durable role-first document path policy, `protos/AGENTS.md`.
- `docs/guide/05-values-identity-equality-and-collections.md` already contains the six-row operator reference, mathematical-width behavior and `&&`/`||` distinction. `docs/guide/README.md` is the conceptual navigation entry.
- `protos/tutorials/README.md` lists fourteen numbered tutorial directories; `01-values-and-slots/01-values.protos` through `03-assignment.protos` are deliberately short executable CLI lessons.
- `protos/examples/README.md` is the task-oriented cookbook; `numbers/exact-and-complex.protos` and `numbers/vector-matrix.protos` are existing standalone executable examples with full APL notices.
- `src/test/java/com/guillermomolina/protos/cli/ProtosCliLearningMaterialsTest.java` recursively discovers both roots and executes every `.protos` file containing `print(`, with 15-second per-program timeout, exit=0, nonempty stdout and blank stderr. This is the established learning-material validation owner; new Java tests or Test Tool corpus registrations are unnecessary.
- `docs/guide/SOURCE_STYLE.md` already sets the D233-B1 idiomatic spelling and does not need modification. Grammar §21 and `VALUES_AND_COLLECTIONS.md` own the normative semantics; `protos/tests/conformance/surface-sugar/bitwise-operators.protos` already owns I137 sugar conformance. No duplicate authority is created.

## Selected six-file single slice

| Product path | Exact bounded change |
| --- | --- |
| `protos/tutorials/01-values-and-slots/04-bitwise-operators.protos` | New CLI-runnable beginner lesson with six operators, `~` dual use, left-to-right strict evaluation, masking, negative arithmetic shift, standard Integer vs custom ordinary message dispatch; checked deterministic printed output and full APL Part 5 header |
| `protos/examples/numbers/bit-flags-and-masks.protos` | New task-oriented, CLI-runnable read/write/execute flags; test/set/clear/toggle, explicit mask, packed octet extraction; checked deterministic printed output and full APL Part 5 header |
| `protos/tutorials/README.md` | One introductory link and explanation inside existing numbered progression |
| `protos/examples/README.md` | One cookbook table entry plus clear no-authority CLI usage |
| `docs/guide/05-values-identity-equality-and-collections.md` | Extend existing bitwise subsection with explicit precedence, maximal-munch and grouping guidance, ordinary override explanation and links to the two programs |
| `docs/guide/README.md` | One cross-linked, discoverable route to existing numeric subsection and both executable sources |

No `pom.xml` / `CHANGELOG.md` / `spec/` edit or new implementation version: documentation, examples and tutorials use already shipped runtime capabilities; no change to `src/` or distributable `protos/lib/`. No own registry or new Protos runtime primitive. Preserve the current docs and old tutorial progression without unrelated rewrites.

## External comparison packaged for the editor — no external research or requests

Lua 5.4 has the same `~` unary/binary glyphs but fixed machine-width/coercion and different shifts; Python/GraalPy supports arithmetic shifts and arbitrary-precision bitwise values but spells XOR `^` (which is nonlocal return in Protos). Ruby/TruffleRuby teaches operator-as-message design, while Protos' exact method names are `bitAnd` and companions. GraalJS `JSBitwiseXorNode` has Number/`ToInt32` coercion and a separate BigInt path, neither applicable to Protos. Truffle SimpleLanguage `SLNodeParser` + `SLAddNode` illustrates parser-versus-execution separation; DOC014 changes neither. Espresso/Sulong model Java/LLVM, not this source language. These precedents were resolved in D233/I137; the editing agent should use only the complete prompt and local Protos repository.

## Publication and handoffs

- [**DOC014 editing-agent prompt**](DOC014_AGENT_ONE_SLICE_PROMPT.md) — exact file-by-file and executable-program content, six paths, external precedent included; **edits only**, no shell/Git/web/other repository.
- [**DOC014 human executor**](DOC014_HUMAN_EXECUTOR_ONE_SLICE.md) — source/whitespace review, automatic CLI JUnit, direct CLI programs, `make test` and `make check`, staging the six paths, one commit, source-style guard, push, publication report.
- Human final report must identify exact product SHA and attribute test PASS to the human. After product publication, a revision-bound record belongs under `guillermomolina/protos-project-docs:docs/project/evidence/DOC014/`, the live issue is updated, and completion is conditional on formal GITHUB020 closure evidence.
- No product tests or checks have been executed by this documentation-preparation step; no product changes are claimed.

```text
WORK_ITEM=DOC014
DOC_FAMILY=PASS
PRODUCT_REPOSITORY=guillermomolina/protos
BASELINE_REVIEWED=c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5
PREVIOUS_IMPLEMENTATION=I137_COMPLETED
DESIGN=D233_B1_RATIFIED
SLICES=1
TARGET_PRODUCT_FILES=6
NEW_PROTOS_FILES=2
EXISTING_CLI_DISCOVERY=YES
PRODUCT_IMPLEMENTATION=PENDING_HUMAN_EXECUTION
DOCUMENTATION_PROMPTS=PUBLISHED
```
