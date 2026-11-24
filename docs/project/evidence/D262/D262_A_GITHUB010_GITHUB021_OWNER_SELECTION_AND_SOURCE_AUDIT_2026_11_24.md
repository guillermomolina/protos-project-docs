# D262-A — Comparative research, invariant review and owner selection

Date: 2026-11-24. Non-normative evidence; not a completed ratification or a test result.

- Live owner: [D262 #1163](https://github.com/guillermomolina/protos/issues/1163).
- Inspected Protos revision: [c98be4618fa5e8dc7a89f5092d3712616803e501](https://github.com/guillermomolina/protos/tree/c98be4618fa5e8dc7a89f5092d3712616803e501).
- Exact owner response in the active interaction, following presentation of D262-A: **"apruebo A"**. The owner also explicitly authorized bounded publication of project-docs evidence and creation of an implementation issue if required.
- Selected candidate: A, two ordinary inherited messages, `createSlot(name, value)` and `updateSlot(name, value)`. The proposed spelling was explicit in the recommendation. This is owner selection, not authorization to claim tests or normative publication complete.
- GITHUB010 comparison was presented with six candidates and twelve axes. Full-document coverage and complete optimized-path verification were explicitly outstanding; a final GitHub read disconnected. Do not mark the exhaustive research gate PASS yet.

## Existing contract and source evidence

Revision-pinned normative sources (paths relative to the inspected product revision):
- `spec/semantics/OBJECT_MODEL.md` §§3, 20, 22–24 and Core Reflection: separate local create/update, read delegation only, shallow state, exact name scalar sequence, deterministic fresh reflection snapshots, root Object frozen before observation.
- `spec/semantics/EXECUTION_AND_CONTROL.md` §§4, 6, 7, 8.1: live lexical membership, captured context identity, bare assignment selects destination before RHS, explicit member assignment evaluates RHS before mutation, exact value result.
- `spec/semantics/CALLABLES.md`: ordinary call packet and receiver/methodHome, capture, invocation.
- `spec/semantics/ERRORS.md` §25 taxonomy: generic Error for structural mutation failures, no new prototype.
- `spec/semantics/MODULES.md`: SOURCE context and MODULE private initializer/public object; frozen prelude.
- `spec/semantics/VALUES_AND_COLLECTIONS.md` Indexed Access Syntax; `spec/PROTOS_GRAMMAR.md` §§8–10, 18: index access stays at/atPut.
- `spec/concurrency/ACTORS.md` §§16–18, 24H–I: transfer and isolation unchanged.

Inspected scoped rules: AGENTS.md, AGENTS.work/DESIGN.md, AGENTS.work/REFERENCE.md, spec/AGENTS.md, src/AGENTS.md. Coordination additionally inspected AGENTS.work/COORDINATION.md, AGENTS.work/IMPLEMENTATION.md and the project-docs implementation blocker ledger (no matching D262/D179/PLAT036 entry found by targeted search). Full normative coverage remains pending; targeted retrieval is not equivalent to whole-document review.

Implementation source evidence under src/main/java/com/guillermomolina/protos/:
- runtime/ProtosObjectValue.java (blob 8b8354e034ac97f257803ef2dffce1075e286fe4), createLocalSlot, assignLocalSlot, removeLocalSlot: same single binding authority; create rejects frozen/closed/duplicate; assign rejects frozen/absent; removal rejects frozen/closed/absent.
- execution/protocols/values/ProtosStandardObjectProtocol.java (75b8d94fcef1add73ec7a4bbf919b5bbacb1adcd), existing hasSlot, slotValue, slotNames, removeSlot and receiver-domain guard. No public create/update.
- runtime/lexical/ProtosMapBackedLexicalBindingAuthority.java (f8429a4b4fdc3f308f01554173693c0a478e3852), DynamicObject property operations and interned keys; no new key infrastructure authorized.
- execution/bytecode/activation/ProtosBytecodeLexicalFrameView.java (995f479bda7418101b5d96b8e590e09158cd6271), containsBinding/readBinding/putBinding/removeBinding: declared names use original local/cell and cleared presence; names outside layout use existing lazy overflow. No duplicated value for declared bindings.
- execution/bytecode/activation/ProtosFrameLocalBindings.java (f879372835a0c6d20a86f771befc52a715762955), assignResolvedLexicalTarget preserves exact destination.
- execution/bytecode/activation/ProtosLexicalSelectionNode.java (7de12c1ab07ae6a94edff4ab6a599af1601f1e47), live selection and owner presence.
- execution/bytecode/lowering/CanonicalToBytecodeLowerer.java (8bd7e232f3ed29e5f5eea31ea604901d4fa1ae09), selected/direct lexical paths identified; exhaustive direct/selected/global/cell path audit outstanding.

## Comparative research

Primary references and observations (external moving branches are not claimed to be release-pinned; exact inspected blobs are listed where available):
- Io: [language guide](https://iolanguage.org/guide/guide.html), [IoObject.c](https://github.com/IoLanguage/io/blob/master/libs/iovm/source/IoObject.c), inspected blob 95df6e680a00fea7a8c783a44df6049231814828. setSlot is upsert, updateSlot relies on existing lookup, newSlot adds a setter. C setter delegates to inline setter. Do not import Io delegation or locals special cases.
- Self 2024.1: [language reference](https://handbook.selflanguage.org/2024.1/langref.html), [mirrors](https://handbook.selflanguage.org/2024.1/progguid.html). Separate reflectors permit structural changes, but introduce an additional authority/protocol surface.
- ECMAScript: [ordinary object internal methods](https://tc39.es/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots). Separate Set and DefineOwnProperty; descriptors/prototypes/proxies and indexed property syntax are incompatible with Protos slot/index separation.
- Python 3.13+: [setattr](https://docs.python.org/3/library/functions.html#setattr), [PEP 667](https://peps.python.org/pep-0667/). Attribute upsert can execute custom behavior; write-through frame locals proxy supports extra names, but compiled local deletion and lexical semantics differ from Protos. Do not import Python unbound-local policy.
- Ruby: [Object](https://docs.ruby-lang.org/en/master/Object.html#method-i-instance_variable_set), [Binding](https://docs.ruby-lang.org/en/master/Binding.html#method-i-local_variable_set). Instance variable upsert and dynamic binding names; new Binding names do not become compiled lexical declarations. Do not adopt that distinction.
- Pharo: [Object.class.st](https://github.com/pharo-project/pharo/blob/master/src/Kernel/Object.class.st), inspected blob 0467517352883f3a524d18bd9fcf6e0f150e1bad. instVarNamed:put: resolves class slot and writes or signals InstanceVariableNotFound; does not create instance structure.
- Lua 5.4: [rawset](https://www.lua.org/manual/5.4/manual.html#pdf-rawset). Bypasses __newindex, but table keys and nil-as-removal differ from slots and guest null.

Truffle implementation evidence (mechanism precedents, not semantic imports or authorization of a Java patch):
- [GraalPy BuiltinFunctions.java](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/BuiltinFunctions.java), blob 3dc09955f87b201b9f8750d5c076dcbbfc03c039: SetAttrNode.setAttr, @Specialization, @Cached PyObjectSetAttrO, delegates to existing write path.
- [WriteAttributeToPythonObjectNode.java](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/attributes/WriteAttributeToPythonObjectNode.java), blob ba036bf121448f4818c055b436ef22ac47533fc9: @GenerateUncached, @Specialization write, @Cached DynamicObject.PutNode; direct storage.
- [WriteAttributeToObjectNode.java](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/attributes/WriteAttributeToObjectNode.java), blob ba1c5b3cc19b80a858849b31a692b09530151dd1: writeToDynamicStorageNoType, writeToDynamicStoragePythonClass, writeToDynamicStorageManagedClass; storage guards and cached PutNode.
- [GraalJS WriteElementNode.java](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/access/WriteElementNode.java), blob d2fb7a6c7978470922b51d06e2820cad2b4f458f: guarded dynamic-key specializations. No JS index semantics imported.

## Candidates and GITHUB010 matrix

A: explicit strict create/update; B: one operation with modes; C: composition/Map/library reconstruction; D: generalized property/index protocol; E: status quo; F: separate mirrors. Scores reflect the presented comparative judgment, not measurements.

| Axis | A | B | C | D | E | F | Rationale |
|---|---:|---:|---:|---:|---:|---:|---|
| Correctness | 5 | 5 | 3 | 1 | 5 | 5 | A/B preserve local semantics; C loses identity; D conflicts with indexing; E retains status quo; F can preserve via view |
| Protos alignment | 5 | 3 | 4 | 1 | 3 | 2 | A completes ordinary mechanism; B conflates distinct intents; C composes indirectly; D/F add institutions; E leaves asymmetry |
| Present proportionality | 5 | 4 | 3 | 1 | 3 | 2 | A adds two primitives only; B adds modes; C copies; D transversal; E pushes cost to clients; F adds views |
| Incremental growth | 5 | 4 | 3 | 2 | 3 | 3 | A library growth; B mode growth; C reconstruction limits; D premature model; E later A; F extra layer |
| Future options | 5 | 4 | 3 | 2 | 4 | 4 | A representation-neutral; B mode contract; C/E leave gaps; D locks property model; F flexible |
| Scalability | 4 | 4 | 3 | 3 | 4 | 4 | A/B local storage; C copies; D extra interactions; E no new runtime; F view lifetime |
| Simplicity | 5 | 3 | 3 | 1 | 4 | 2 | A two existing concepts; B modes; C indirect; D rules; E incomplete; F new concept |
| Portability | 5 | 5 | 5 | 3 | 5 | 4 | A/B/C/E backend-neutral; D/F more machinery/contract |
| Runtime cost | 5 | 4 | 3 | 2 | 5 | 3 | A/E can retain static paths; B mode dispatch; C allocation; D generic routes; F materialization |
| Failure/operability | 5 | 4 | 3 | 2 | 3 | 3 | A explicit failure; B invalid modes; C/E clients verify; D precedence; F view failures |
| Reversibility | 5 | 3 | 3 | 1 | 3 | 2 | A additive with selector collision; B contract growth; C/E client rewrites; D breaking; F new API |
| Evidence/risk | 4 | 3 | 5 | 2 | 2 | 3 | A existing seam; B/D/F wider surface; C existing composition; E requirement unmet |
| Total | 58 | 46 | 41 | 21 | 44 | 37 | Not decision authority |

Confidence: high for directly inspected semantic distinctions, medium for runtime cost/scalability, medium for feasibility pending full optimized-path audit; external version coverage incomplete. D is disqualified by existing index invariants; B/F unjustified generalization; C/E fail the exact existing-receiver mutation requirement. Strongest objection to A: dynamic lexical shadowing can invalidate cached owner proofs and root selector addition changes reflection/collisions. The obligation to observe live membership already exists under D179, but cost neutrality is not measured.

Pay for what you use: no new per-object storage or new static bytecodes; reflective calls pay name resolution and structural checks. Grow via ordinary libraries, not primitive upsert/mirrors. Deferral can add A later without changing the foundational model, but clients using Maps or reconstructing objects may need rewriting, especially identity-sensitive callers. Large untrusted name sets warrant tests of intern/shape cardinality and memory; use Map for data if slot behavior is not wanted. FFM snapshots, JSON/CBOR String-key objects, schema-generated records, persisted objects and dynamic methods compose without new institutions. Arbitrary external keys such as call/init are behavior-bearing slots, not automatically safe data. Non-String CBOR keys remain Map keys. LSP cannot promise statically resolved dynamic names; no tooling redesign authorized. Actor transfer snapshots and non-transferable context boundaries unchanged. No native memory/host write authority, no new synchronization or cancellation semantics.

## GITHUB021 review and provenance

D179 C0/E approval recorded in [#703](https://github.com/guillermomolina/protos/issues/703) and implemented by closed [I071 #717](https://github.com/guillermomolina/protos/issues/717): creation while open, mutation while writable, removal while open, live lexical fallback, stable static layout and one authoritative binding value. Existing overflow is retained; no reopening approved or needed by the proposed contract. D140 #568 horizontal composition stays unchanged. I031 #242 existing Object reflection/state publication retained. D032 deterministic slotNames, D033 exact removed value, D237 strict argument packet, D239 ordinary invocation, D242 SOURCE/MODULE publication, PLAT035 Bytecode-only, PLAT036 single authority preserved. D261 #1158 remains independently actionable; no magic native properties or new FFM dependency. The only identified new observable surface is two root-local ordinary selectors, with possible inherited-name collisions; this was disclosed before approval. This is a proposal-level consistency review; the full normative and optimized-path audit remains a separate gate.

## Required conformance and remaining gates

Existing tests inspected by name/method inventory (not executed): ProtosObjectValueTest, ProtosLexicalBindingAuthoritySeamTest, ProtosI115PublishedLexicalContextTest, ProtosPerf025D179LexicalMembershipStabilityReadTest, CanonicalBareSlotMutationExecutionTest, ProtosObjectInteropTest, ProtosCoreContextSourceTest. New guest tests must cover strict create/update, state/receiver/arity/name domain, Unicode and empty names, exact identity including null and Closure, inherited-only failure/override, root/prelude freeze, slotNames snapshot and order, alias/composition, call/construct/init selection, override of reflective messages, RHS effects and errors before dispatch, non-local return, context dynamically created unknown names, original locals and captured cells, nearer shadowing, removal/recreation, escaped/suspended contexts, module private/public separation, Actor transfer and read-only host projection. Ensure named-argument association and no index changes. Include ordinary guest conformance, Java mechanism tests only as complementary, no legacy execution path.

Complete remaining whole-document coverage and optimized-path falsification before production edit; report conflicts rather than weakening semantics. Normative owners and spec changelog must be reconciled before/with implementation; no new grammar or Error prototype. Use exact upstream method/annotation/behavior precedents for each Java production change, no new infrastructure without evidence. Compare static bytecode and Tier-1/Tier-2 with baseline, stress dynamic name cardinality; human executes focal and integrated gates under AGENTS. No builds, tests, product edits or product publication performed. D262 remains OPEN; owner selection recorded, exhaustive gate and normative publication pending.
