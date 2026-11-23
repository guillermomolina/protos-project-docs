# D260-B1 — GITHUB010 comparative packet and GITHUB021 approval evidence

**Date:** 2026-11-23
**Live Issue:** [D260 #1154](https://github.com/guillermomolina/protos/issues/1154)
**Decision record:** [D260-B1](../../decisions/language/D260_B1_INTERACTIVE_TERMINAL_SCREEN_CONTRACT_SELECTED_2026_11_23.md)
**Downstream:** [LIB032 #1152](https://github.com/guillermomolina/protos/issues/1152) and [PLAT078 #1155](https://github.com/guillermomolina/protos/issues/1155)
**PROTOS_REVISION:** [9c56aba2c8b84a9c44440cb361db85b53fcf30b4](https://github.com/guillermomolina/protos/commit/9c56aba2c8b84a9c44440cb361db85b53fcf30b4)
**PROJECT_RECORD_REVISION:** identified by the immutable documentation-publication commit, separately linked in the live D260 issue.

## Exact owner approval

The first bounded LIB032 recommendation (Protos-first, ncursesw first candidate, termbox2 comparison, no new Java infrastructure) was explicitly accepted with **"si auebo"**, but public semantics and backend selection remained open. Following exhaustive D260 research, the user was explicitly asked:

> ¿Apruebas D260-B1 con el contrato público y las reglas exactas expuestas arriba, manteniendo en PLAT078 la obligación de demostrar su viabilidad antes de implementar LIB032?

The next user turn answered verbatim **"Apruebo D260-B1"**, also granting a specific exception for direct commit/push **only** in `guillermomolina/protos-project-docs`. This authorizes the exact D260-B1 selection, **not** unspecified new public behavior, a native backend or claims that tests have passed.

## Current sources materially inspected

| Exact Protos path | Binding fact |
| --- | --- |
| `AGENTS.md`, `AGENTS.work/DESIGN.md`, `COORDINATION.md`, `REFERENCE.md` | GITHUB010 breadth/scoring, GITHUB021 invariants, GITHUB015 publication and source-authority discipline |
| `spec/semantics/VALUES_AND_COLLECTIONS.md` | String indexing counts Unicode scalars, not display cells or graphemes |
| `spec/semantics/MODULES.md` | Actor-local modules and foreign facades; import does not mint native authority |
| `spec/semantics/FFI.md`, `docs/guide/library/ffi.md` | Explicit host-allowlisted trusted native libraries, limited public ABI, synchronous calls |
| `providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmBinding.java` | Finite physical Native Image downcall signature set; C structs cannot be assumed represented |
| `spec/io/PROCESS_IO.md` | Standard streams are byte capabilities, not assumed terminals or default Closables |
| `spec/io/IO_CORE.md`, `spec/concurrency/FUTURES_AND_TASKS.md` | Explicit irreversible Closable lifecycle and distinct result Future identities |
| `spec/concurrency/ACTORS.md` | Nontransferable native handles; synchronous native operations occupy Actor segments; trusted C global-state exception does not make guest state shareable |
| `spec/semantics/ERRORS.md` | Error prototype/occurrence identity, standard IO and FFI error categories |
| `protos/lib/text/style/module.protos`, `protos/lib/text/styled_text/module.protos`, `protos/lib/text/ansi/module.protos` | D190/D191 pure styled text is opaque; no public style-state introspection |
| `spec/semantics/SQLITE.md`, `protos/lib/sqlite/module.protos` | Existing lazy source-backed native-library acquisition and cleanup pattern |

## Cross-system comparison (GITHUB010)

| Approach / systems | Primary references | Reasoned contribution |
| --- | --- | --- |
| Immediate cursor and batched physical updates: Python curses, ncurses | [Python curses](https://docs.python.org/3/library/curses.html), [ncurses](https://invisible-island.net/ncurses/) | `noutrefresh`/display flush illustrates logical/physical separation; native windows/cursor semantics unnecessary as a public model |
| Cell backbuffer and events: termbox2, Go tcell | [termbox2](https://github.com/termbox/termbox2), [tcell](https://github.com/gdamore/tcell) | Explicit screen cells, `present`/`Show`, keyboard and resize events; avoid scalar-count-as-width |
| Public Grid, diff and frames: Rust Ratatui | [Ratatui](https://ratatui.rs/) | Differential rendering is useful backend machinery; mandatory public Frame/Grid overcharges a minimal screen |
| Application model/event loop: Bubble Tea | [Bubble Tea](https://github.com/charmbracelet/bubbletea) | Good downstream framework, not a primitive Screen contract |
| Layered terminal/screen/widgets: Lanterna | [Lanterna](https://github.com/mabe02/lanterna) | Screen distinct from a full widget system |
| Command/event substrate: crossterm | [crossterm](https://docs.rs/crossterm/) | Low-level terminal control alone does not supply a logical cell image |
| Existing Protos pure styled text/ANSI: D190/D191 | [D190 #826](https://github.com/guillermomolina/protos/issues/826), [D191 #827](https://github.com/guillermomolina/protos/issues/827) | Pure String renderer is deliberately not terminal/screen authority |

At least seven systems and multiple design families were compared. The survey selects **public semantics**, not a Truffle/JVM implementation. No custom Java production change has an approved justification here.

## Meaningful alternatives and complete twelve-criterion comparison

Candidates: **A** immediate cursor writes, **B** logical Screen with explicit `present` (owner-selected), **C** mandatory exposed Grid, **D** callback Frame, **E** Model–Update–View runtime, **F** defer. ANSI streaming is excluded because it does not satisfy the requested screen abstraction.

1 = poor, 5 = strong; **H** high, **M** medium confidence. Each cell gives the rationale; these are qualitative design scores, **not measured performance**.

| Dimension | A immediate | B logical Screen | C public Grid | D Frame | E app runtime | F defer |
| --- | --- | --- | --- | --- | --- | --- |
| Correctness | 3M cursor state | **5M clear publish** | 5M explicit image | 5M scoped view | 5M event structure | 5H no change |
| Protos alignment | 3M native shape | **5M ordinary messages** | 4M extra objects | 4M callbacks | 2H framework | 4H unchanged |
| Present need | 4M minimal | **5H sufficient** | 3M extra surface | 2M extra protocol | 1H overbuilt | 1H unmet |
| Incremental growth | 2M retrofit batching | **5M additive** | 4M layers | 4M widgets | 4M growing engine | 3M delayed |
| Future options | 3M immediate lock | **5M backend neutral** | 5M generic image | 4M fixed frames | 3M architectural lock | 5H open |
| Scalability | 3M per call | **4M grouped changes** | 4M buffer scaling | 4M frame reuse | 4M own loop | 5H zero |
| Conceptual simplicity | 4M few methods | **5M few rules** | 3M Grid lifecycle | 3M callback protocol | 1H extensive | 5H no API |
| Portability | 3M terminal cursor | **4M abstract cells** | 5M pure Grid | 4M generic | 4M portable logic | 5H no lock |
| Runtime/resources | 3M repeated I/O | **4M native renderer reuse** | 3M redundant state | 3M rebuild | 2M allocation | 5H zero |
| Failures/operation | 2M immediate prefix | **4M present boundary** | 4M inspectable | 4M frame cutover | 3M more states | 5H none |
| Reversibility | 2M breaking change | **5M stable surface** | 3M Grid promise | 3M Frame promise | 2M migration | 2M ad hoc future |
| Evidence maturity | 5H curses | **5H tcell/termbox2** | 5H Ratatui | 5H Ratatui | 5H Bubble Tea | 5H simple |

**Overengineering:** E prescribes scheduler, model and widget behavior without today's need. **Underengineering:** A makes future atomic logical image updates an observable behavior change. **B's smallest sufficient model** is a single acquired screen with logical cell writes, presentation, keyboard/resize, and Closable.

## GITHUB021 explicit invariant and delta audit

| Previously approved invariant | Selected B1 result |
| --- | --- |
| Terminal screen management, not stream formatting | **PASS:** logical cells and `present`; no ANSI/Writer fallback |
| Protos-first; no new Java terminal subsystem | **PASS:** ordinary module/objects/messages; existing FFM |
| Explicit D259/PLAT077 native host authorization | **PASS:** import inert, acquisition explicit; no sandbox claim |
| ncursesw first candidate and termbox2 A/B | **PASS:** PLAT078 remains undecided, tests required |
| No initial widget/GUI/WUI runtime | **PASS:** no Grid/Frame/widgets/event loop mandatory |
| Explicit bounded resource owner | **PASS:** single active screen, native nontransferability and proper Closable |
| D190/D191 styled text purity and opaque state | **PASS:** string foreground without hidden style inspection |
| Existing Actors and Context isolation | **PASS:** no guest-shared mutable MODULE; no implicit Actor suspension |
| Prior B1 public API was not yet approved | **EXACT NEW APPROVAL:** named D260-B1 details explicitly selected by owner; not inferred from the earlier broad-direction approval |
| Native feasibility conditional | **PASS:** backend choice, ABI, Unicode and signals held in open PLAT078 |

**DECISION_APPROVAL_PROVENANCE=PASS. DECISION_INVARIANT_CONSISTENCY=PASS.** These do not imply `SPEC_NORMATIVE_PUBLICATION=PASS`, which remains unsatisfied.

## Adversarial cases, deferred cost, strongest objection

Rejection before logical mutation: input `"A\nB"`, ESC/ANSI, invalid coordinates, invalid color, unsupported grapheme. A wide glyph clipped at the final column cannot render partially; overwriting its continuation column clears its earlier footprint. Repeated `put` before `present` must not force physical effects. An actual TTY may resize after a size snapshot; pending image is invalidated upon recognized resize. A second Actor or Context cannot acquire an independent active Screen, and a Screen in an Actor message must fail nontransferable rather than proxy. `nextEvent` must not secretly yield to unrelated Actor work. Repeated `close` observes one destruction and same recorded close failure via different result Futures. Physical partial output cannot be rolled back.

**Pay as you grow:** import has no acquisition; no buffers/workers for programs that do not open Screen. **Grow as needed:** cursor, scroll, event families, Grid, async input, richer styles and widgets can be additive. **High cost of deferral:** immediate writes or scalar-equals-column representation require breaking future redesign. **Low cost of deferral:** extra keys/mouse/layout are additional library operations. **Future stress:** many Actors and distributed nodes exchange inert data, not native Screen; host process ownership limitation remains explicit. **Strongest objection:** current FFM cannot directly express some C signatures/structs; ncursesw/termbox2 grapheme and exclusive-session behavior are unproven. PLAT078 must resolve or return to owner; implementation cannot silently weaken B1.

## Evidence and publication ledger

- `PROTOS_REVISION=9c56aba2c8b84a9c44440cb361db85b53fcf30b4` — current inspected product revision; **no D260 normative spec** or implementation claimed.
- `PROJECT_RECORD_REVISION` — the immutable commit publishing this record, to be added to the live Issue from GitHub's returned SHA.
- **APPROVAL=PASS** and **INVARIANTS=PASS**, only for the exact owner-reviewed design.
- **SPEC_NORMATIVE_PUBLICATION=PENDING**, **PLAT078_NATIVE_VIABILITY=PENDING**, **LIB032_PRODUCT_IMPLEMENTATION=PENDING**.
- No builds, tests, native PTY probes, Java/Native Image executions or local `git diff --check` were run in this documentation/Issue-only task. The authorized docs commit should be re-read by exact SHA; local validation remains human-executed. The native GitHub parent/sub-issue relation and Project routing cannot be asserted from textual links alone.
