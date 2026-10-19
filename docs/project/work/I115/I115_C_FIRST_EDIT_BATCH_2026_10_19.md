# I115-C — agent's first edit batch (read-only inspect, then Python patch)

**TYPE = IMPLEMENTATION**; execute in local `guillermomolina/protos`, assuming it is already at its real HEAD. The agent may edit; the human runs any validation, build, tests and Git. **Assume one execution mode.** Do not inspect other repositories/web. This batch is a _foundation_ for I115-C, not a complete PIC implementation.

Exact starting semantics: `ProtosObjectValue.trackSlotSelectionDependency` currently refuses all subclasses. The only subclass newly authorized here is final `ProtosClosureValue` **for selector `call`**. Its local ordinary slots use the inherited single binding authority. A PIC-selected absence of a local `call` may be guarded by a lazily registered selector dependency only if the dependency invalidates **before** a structural create/remove/compose becomes visible. Assigning a present slot changes value but not selection; value lookups must still invalidate before the new value appears. A tracker may be created only when a PIC entry installs: no per-Closure eager token/shape.

**First commands for agent to edit** (never run tests/builds as agent):

```bash
python3 - <<'PY'
from pathlib import Path

p = Path("src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java")
s = p.read_text()

def replace_once(before, after):
    global s
    found = s.count(before)
    if found != 1:
        raise RuntimeError(f"Expected one exact edit anchor, found {found}: {before[:100]!r}")
    s = s.replace(before, after, 1)

selection_start = s.index("    final boolean trackSlotSelectionDependency(String name, Assumption dependency) {")
selection_end = s.index("\n    private void invalidateValueLookupDependencies(", selection_start)
segment = s[selection_start:selection_end]
old = "if (getClass() != ProtosObjectValue.class) {"
new = 'if (getClass() != ProtosObjectValue.class\n                && !(getClass() == ProtosClosureValue.class && "call".equals(name))) {'
if segment.count(old) != 1:
    raise RuntimeError("Cannot safely restrict selector dependency tracking to Closure.call")
s = s[:selection_start] + segment.replace(old, new, 1) + s[selection_end:]

replace_once(
    "        ProtosLexicalBindingAuthorityCalls.put(\n                writableLexicalBindingAuthority(), name, value);\n        invalidateLookupDependencies(name);",
    "        invalidateLookupDependencies(name);\n        ProtosLexicalBindingAuthorityCalls.put(\n                writableLexicalBindingAuthority(), name, value);"
)
replace_once(
    "        ProtosLexicalBindingAuthorityCalls.put(lexicalBindingAuthority, name, value);\n        invalidateValueLookupDependencies(name);",
    "        invalidateValueLookupDependencies(name);\n        ProtosLexicalBindingAuthorityCalls.put(lexicalBindingAuthority, name, value);"
)
replace_once(
    "        Object removed =\n                ProtosLexicalBindingAuthorityCalls.remove(lexicalBindingAuthority, name);\n        invalidateLookupDependencies(name);",
    "        invalidateLookupDependencies(name);\n        Object removed =\n                ProtosLexicalBindingAuthorityCalls.remove(lexicalBindingAuthority, name);"
)
before = '''            ProtosLexicalBindingAuthorityCalls.put(
                    writableLexicalBindingAuthority(),
                    contributionNames.get(index),
                    contributionValues.get(index));
            invalidateLookupDependencies(contributionNames.get(index));'''
after = '''            invalidateLookupDependencies(contributionNames.get(index));
            ProtosLexicalBindingAuthorityCalls.put(
                    writableLexicalBindingAuthority(),
                    contributionNames.get(index),
                    contributionValues.get(index));'''
replace_once(before, after)

p.write_text(s)
print("Edited:", p)
print("Foundation only: agent must audit all mutation producers and finish full PIC + tests.")
PY
```

If any anchor is missing on current HEAD, **stop before writing** and adapt from inspected current source; never silently use partial substitutions. Audit any direct authority mutation/bulk transfer and receiver publication path. Add focused tests for mutating `call` during operand evaluation, on fresh and warmed Closure instances, and after JIT. Only after all invalidation paths are proven may the agent replace the per-hit PIC guards. Do not claim source completeness based on this first edit batch alone.

After the agent's complete C patch, the human runs `git diff --check` and focused tests, then `make test`; afterward metadata/version then commit/push, **no test rerun after version files**. See accompanying human-executor document.
