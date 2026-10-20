# PERF042 / PLAT066-C — BASELINE congelado: run → identity, Protos 0.3.386 vs GraalPy 25.4.4.1.1, compilación fría

**Fecha:** 2026-10-20. **Naturaleza:** evidencia histórica de origen humano + interpretación reproducible de Bytecode DSL y BGV; no ratificación de diseño, aceptación de rendimiento ni implementación.
**Producto Protos de referencia:** `guillermomolina/protos@607973c493dcb0c5ba3a7eeba9897b045eb049d8`, `0.3.386-SNAPSHOT` (revisión identificada por el handoff existente; los logs de la captura de `run` no vuelven a declarar SHA del checkout). **JVM:** GraalVM CE 25.4.4.1.1.
**Harness GraalPy:** `guillermomolina/protos-benchmarks@1b28b390b61d5a59e9b1573ea30e26306a6dde86` como referencia documentada; driver temporal `GraalPyRunCold.java` aportado en el archivo, imagen Podman `protos-benchmarks-graalpy:25.4.4`. El driver está **sin publicar en el harness**.
**Coordina:** [PERF042/#871](https://github.com/guillermomolina/protos/issues/871), [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930), [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924). No cerrar ninguna por este baseline.

## Programa y alcance de las medidas

```protos
run: () => {
    identity: (value) => { value }
    identity(1)
}
```

```python
def run():
    def identity(value):
        return value
    return identity(1)

truffleRun = run
```

Ambos dan `1`. Se cuentan **tres raíces guest separadas** — módulo, `run` e `identity` — al medir **FULL DSL**, independientemente de si luego se compilan. Las cifras IR representan **SOLO el StructuredGraph de `run`** en la fase `After TruffleTier`; ninguna cifra de módulo o `identity` compilada en esta ejecución se añade subrepticiamente.

| Métrica | Protos | GraalPy | Delta de Protos |
| --- | ---: | ---: | ---: |
| FULL DSL módulo | 16 | 12 | +4 |
| FULL DSL `run` | **112** | **12** | **+100** |
| FULL DSL `identity` | 5 | 5 | 0 |
| **FULL DSL, suma descriptiva de tres raíces** | **133** | **29** | **+104** |
| **IR Tier 1 de `run`, After TruffleTier** | **7137** | **268** | +6869 |
| IR final Tier 1 de `run` | 19720 | 731 | +18989 |
| **IR Tier 2 de `run`, After TruffleTier** | **7078** | **165** | +6913 |
| IR final Tier 2 de `run` | 12086 | 420 | +11666 |
| CodeSize Tier 1 de `run`, bytes | 123547 | 3544 | +120003 |
| CodeSize Tier 2 de `run`, bytes | 88854 | 2188 | +86666 |
| Inlined Tier 1 | 0Y / 0N | 0Y / 1N | distinto lowerer/Truffle |
| Inlined Tier 2 | 0Y / 0N | 1Y / 0N | distinto lowerer/Truffle |
| Calentamiento en `run` | 128 | 128 | igual |

**Compilación de `run`:** JVM, raíces `CACHED`, dos procesos/solicitudes separados `compile(false)` y `compile(true)`, respectivamente Tier 1 y Tier 2. GraalPy fija `FirstTierCompilationThreshold=1000000`, `LastTierCompilationThreshold=1000000`, deshabilita `DynamicCompilationThresholds` y `BackgroundCompilation` para evitar que el calentamiento provoque compilación automática de `run`. El bootstrap del runtime GraalPy aún compila auxiliares ajenos: **no atribuirlos a `run`**. Misma versión JVM y mismas 128 invocaciones no implican identidad de perfiles, ABI guest, motor o complejidad semántica.

**No sustituir GraalPy frío de 165 nodos por el resultado histórico de 50 nodos:** aquel deriva de compilación estabilizada tras 16.000/64.000 llamadas; evidencia histórica independiente.

## Inventario íntegro de TODAS las operaciones FULL

**Anexo obligatorio, datos físicos completos y ordenados por índice, una fila por instrucción:**
[PERF042_386_COLD_RUN_IDENTITY_FULL_BASELINE_2026_10_20_DSL_FULL.csv](PERF042_386_COLD_RUN_IDENTITY_FULL_BASELINE_2026_10_20_DSL_FULL.csv).
Incluye **145 instrucciones Protos** (16 + 112 + 5) y **24 instrucciones GraalPy** (12 + 12), con ordinal, nombre exacto, raíz y segmentación del `run` de Protos. **Corrección de exhaustividad:** la raíz `identity` de GraalPy tiene recuento FULL=5 confirmado en el log, pero su *secuencia de cinco nombres* no fue entregada; no se reconstruyen nombres no observados. Por eso el CSV reproduce las **169 instrucciones con nombres verificables**, y la cifra 29 incluye los cinco nombres de `identity` aún pendientes de lectura. Esta limitación no afecta a los recuentos.

Secuencia de `run` GraalPy (12):
```text
00 c.EnterCalleeContext
01 load.constant
02 load.constant
03 load.null
04 load.null
05 c.MakeFunction
06 store.local
07 load.local
08 load.constant
09 c.CallUnaryMethod
10 c.EpilogForReturn
11 return
```

Secuencia del módulo GraalPy (12):
```text
00 c.EnterCalleeContext
01 load.constant
02 load.constant
03 load.null
04 load.null
05 c.MakeFunction
06 c.WriteName
07 c.ReadName
08 c.WriteName
09 load.constant
10 c.EpilogForReturn
11 return
```

## Despiece exacto del `run` de Protos (112 FULL)

| Índices de instrucciones | Cantidad | Bloque físicamente emitido | Fuente Java y atribución |
| --- | ---: | --- | --- |
| 0–22 | 23 | prólogo, creación y registro de `identity`, lectura y staging de argumento | `emitRootBody`; `emitComposedCall` |
| 23–28 | 6 | `PrepareInlineClosureLiteralCall`, `AdmitsInlineClosureLiteralCall`, bifurcación | `emitInlineClosureLiteralCall` |
| 29–46 | 18 | cuerpo de `identity` copiado al bytecode del caller con binding y lectura inline | `emitInlineLiteralCallback` |
| 47–104 | **58** | fallback completo, despacho estructurado, continuaciones/yield y try/finally/complete/throw | `emitInlineLiteralFallbackInvocation` → `emitNestedStructuredInvocation` / `emitOrdinaryPreparedInvocation` |
| 105–111 | 7 | limpiezas, retorno y epílogo | `emitRootBody` |
| **Total** | **112** | **Índices 0–111, sin huecos ni superposición** | listado literal en CSV |

**Razonamiento fuente-a-fuente, no intuición:**
- En `CanonicalToBytecodeLowerer.java` líneas **3988–4007**, `admitsTerminalClosureCall` rechaza expresamente si `inlineClosureLiteralCallCandidate(call) != null` o si `currentRootBindingCallCandidates.containsKey(call)`. Para el `identity(1)` ligado a una definición local de este escenario, la ruta terminal `emitTerminalClosureCall` con `InvokeClosureOneTail` queda excluida.
- En el mismo fichero, líneas **10085–10238**, `emitComposedCall` reconoce candidato y selecciona `emitInlineClosureLiteralCall`. En **10541–10585**, esta rutina expande **tres regiones físicas**: preparación y admisión; cuerpo inlined; fallback.
- El fallback se emite por `emitInlineLiteralFallbackInvocation` (**6726–6780**) usando `RequiresStructuredDispatch`, `emitNestedStructuredInvocation`, `emitOrdinaryPreparedInvocation` y `CompleteClosureCall`. Esto explica el grupo FULL 47–104. Las instrucciones no son necesariamente ejecutadas en el caso común.
- En GraalPy `RootNodeCompiler.java` (tag `graal-25.4.4`), `StatementCompiler.emitCall` alrededor de **2385–2504** elige `beginCallNAry(1)` → `CallUnaryMethod` y el runtime implementa la preparación e invocación; **no copia el cuerpo `identity` al Bytecode DSL de `run`**. Graal registra `Cutoff` de `identity` en Tier 1 y `Inlined` en Tier 2. Esto prueba una diferencia **de responsabilidad física en el lowerer**, no una autorización para imitar las semánticas Python.
- El patrón de Protos no es inevitable para una llamada terminal; ya existe `InvokeClosureOneTail` con responsabilidad semántica PLAT062-T. **Candidato de investigación prioritario:** comprobar si la selección I096 (inline explícito de cuerpo) excluye de forma innecesaria el terminal compacto para el caso `run → identity`, conservando toda la semántica mediante los guards, PIC/assumptions y fallback internos ya existentes. No declarar automáticamente que la alternativa produce 12 FULL ni que el ahorro será de 100: requiere emisión y tests/BGV A/B. No eliminar D013 dinámico ni el fallback necesario, ni trasladarlo a otro handler para mejorar un recuento nominal.

## Diagnóstico de grafos Tier 2 de Protos (BGV decodificado)

**Atribución disjunta por handler del ancestro `CachedBytecodeNode` externo**, no porcentaje de CPU/hot-path. Valores Tier 1/Tier 2:

| Handler | Tier 1 | Tier 2 |
| --- | ---: | ---: |
| `PrepareInlineClosureLiteralCall` | 3111 | 3111 |
| Sin handler atribuible | 1482 | 1423 |
| `BindInlineClosureFrameParameter` | 352 | 352 |
| `store.local` | 339 | 339 |
| `load.local` | 336 | 336 |
| `CreateCurrentIndexedLocalSlot` | 306 | 306 |
| `clear.local` | 223 | 223 |
| `LoadInlineClosureArgument` | 168 | 168 |
| `ReadInlineFrameLocal` | 161 | 161 |
| `MaterializeCurrentClosure` | 134 | 134 |
| `CurrentActivation` | 87 | 87 |
| `InstallFrameLexicalAuthority` | 78 | 78 |
| `EpilogForReturn` | 75 | 75 |
| Otros handlers | 285 | 285 |
| **TOTAL** | **7137** | **7078** |

Dentro de los **3111** nodos de `PrepareInlineClosureLiteralCall` Tier 2 (disjuntos *entre sí*): despacho/fallback compuesto **1558**, otras alternativas fallback **486**, preparación intrínseca estándar **326**, admisión/lookup/carrier **741**. Estas cantidades representan **ascendencia de métodos inlined Java** y no equivalen a 3111 nodos semánticamente superfluos.

Clases Tier 2 del grafo Protos: `FrameState=1419`, `BeginNode=803`, `EndNode=578`, `FixedGuardNode=382`, `PiNode=369`, `IfNode=361`, `LoadFieldNode=355`, `IsNullNode=272`, `MergeNode=218`, `ValuePhiNode=199`, `RawStoreNode=190`, `InvokeWithExceptionNode=75`, `MethodCallTargetNode=75`, `ExceptionObjectNode=76`, `CommitAllocationNode=124`.

**Importante:** handlers `RequiresStructuredDispatch`, `EnterNestedStructuredDispatch`, `IsContinuationForCall`, `ResumeContinuation`, `FinishClosureCall` y `CompleteClosureCall` tienen **cero nodos de handler atribuibles** en el `After TruffleTier` Protos capturado, pese a ocupar instrucciones FULL. Por tanto, quitar 58 instrucciones del fallback sin preservar selección/semántica **no** asegura eliminar 58 nodos IR; la diferencia DSL e IR debe contrastarse por separado.

## Procedencia, archivos crudos y SHA-256

Archivos **aportados por el propietario** y disponibles como adjuntos originales de la conversación, **no subidos como BGV binarios a este repositorio documental**. Los hashes permiten verificar cualquier copia y el CSV anterior preserva el censo FULL completo con nombres conocidos.

| Archivo | Tamaño / identificador | SHA-256 |
| --- | --- | --- |
| `perf042-run-identity-386.tar.gz` | 32.334.752 bytes | `2215f3813ff2bbf9ef41faf49f837f12db3bcdb056638f3137630702a9c99300` |
| Protos `tier1/maven.log` | 48.354 bytes | `affb0011c2d0afe55a96dc15c902a3a7fb6f4b2e3f6884a44d012fc91a3c37ce` |
| Protos `tier2/maven.log` | 48.354 bytes | `2622e5229b783f1f66487951814b7aeaf96287736198c49488200eed7f2164be` |
| Protos BGV Tier 1 `CompId 3554` | 87.847.109 bytes | `a573f8a8c329b160ca888ca864ac2285bc2d34f73a2db6af403a54a4400e10c1` |
| Protos BGV Tier 2 `CompId 3569` | 126.808.591 bytes | `475bccb23071286d89a9580851854a78b6ce0349fea7483155862e90c2422d10` |
| `perf042-graalpy-run-cold-20261020T090032Z.tar.gz` | 24.605.428 bytes | `d1f20cc2c8bda8240f871d24a023c50c7360908c7032ff8bc0cb16ecd8b12057` |
| GraalPy driver original `GraalPyRunCold.java` | 8.216 bytes | `c364dfcc622cff8b50ce80f3a8abc6bf5dc532411187ec07c2ccbbd0aac2faa2` |
| GraalPy `tier1/run.log` | 20.483 bytes | `e79c1513801f65aa77ee7347d86f792ee63864b25911c2a076d537e73ed882ac` |
| GraalPy `tier2/run.log` | 21.325 bytes | `203b9c966fe2c10b446b8e048ed1dc7f0842e50a7af13b6f0c0a6fb77bf3a6e5` |
| GraalPy BGV Tier 1 `CompId 5912` | 6.103.210 bytes | `5395479aded2cbdbd6e4db5d9ed556efd2b86cec48ac68f03288f13dd4b3c382` |
| GraalPy BGV Tier 2 `CompId 5551` | 12.497.264 bytes | `2c91e5ad7974fb63758a347073b21ef3da3eedfe087811f35ea3aed9a0bfef0e` |

**BGV GraalPy comprobados independientemente** en grafo `StructuredGraph / After TruffleTier` (no AST): 268 / 165. Los logs confirman CACHED, 128 warmup, resultado 1, compile-return true, tier 1/2 solicitado y alcanzado, y una coincidencia única de raíz `run` por tier.

**Censo módulo GraalPy posterior** (manteniendo la carga exacta y sin volver a compilar Tier 1/2): salida humana literal `GRAALPY_MODULE_FULL=12` y `GRAALPY_MODULE_OPS=[c.EnterCalleeContext, load.constant, load.constant, load.null, load.null, c.MakeFunction, c.WriteName, c.ReadName, c.WriteName, load.constant, c.EpilogForReturn, return]`. Este censo **no forma parte del tar original**, pues se obtuvo después en una invocación auxiliar. La salida humana también confirmó `GRAALPY_COLD_RUN_DSL_FULL=12`, `GRAALPY_COLD_IDENTITY_DSL_FULL=5`. No atribuir al tar original una captura de módulo que no contiene.

## Restricciones semánticas / de diseño y próxima validación

Normativos: `spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`; `D013` dynamic nearest `call`/override después de operandos, `D179` Context y presencia/binding referencial, `D203` argumentos/default/rest, ReturnHome `^` y error, Task/Actor/Process, suspensión/continuaciones, debugger/DAP y un intérprete PLAT062-B/T. No cambiar estas reglas para reducir recuentos. PLAT066-C continúa **investigación abierta**; I115 permanece abierta para slice futuro bajo instrucción previa del propietario; **no crear nueva I ni iniciar implementación** solo a partir del baseline.

**Acceptance de próximo experimento:** comparar mismo `run → identity(1)` en Protos en un A/B humano, con FULL para las tres raíces, Tier 1 y Tier 2 de `run` tras exactamente 128 llamadas, bytecode seleccionado `CACHED`, ambos BGV y logs, más regresiones de mutation/shadow/override de `call`, lexical capture/presence, error, ReturnHome, Task y DAP. No confundir bajada de DSL con bajada de IR ni con speedup.

## Fuentes leídas

- `guillermomolina/protos@607973c4`: `CanonicalToBytecodeLowerer.java` (`emitRootBody`, `admitsTerminalClosureCall`, `emitComposedCall`, `emitInlineClosureLiteralCall`, `emitInlineLiteralCallback`, `emitInlineLiteralFallbackInvocation`); `ProtosBytecodeRootNode.java` (operaciones); `ProtosClosureCallPreparation.java`, `ProtosInlineCallbackFrameBindings.java`, `ProtosSendPreparation.java` y diagnóstico `ProtosPerf042IdentityBuilderOracleTest.java`; los dos BGV/log Protos aportados.
- `oracle/graalpython@graal-25.4.4`: `compiler/bytecode_dsl/RootNodeCompiler.java` (emisión `CallUnaryMethod`), `builtins/objects/function/PFunction.java` (`getCallTarget`), `nodes/bytecode_dsl/PBytecodeDSLRootNode.java` (interpretación); los dos BGV/log GraalPy y el driver aportados.
- `guillermomolina/protos`: `AGENTS.md`, `AGENTS.work/PERFORMANCE.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`; issues PERF042, PLAT066 y PLAT066-C.
- `guillermomolina/protos-project-docs`: `AGENTS.md`, `docs/project/README.md`, `docs/project/evidence/README.md`, `PERF042_386_BARE_TOP_LEVEL_IDENTITY_THREE_METRIC_ACCEPTANCE_AND_RUN_IDENTITY_HANDOFF_2026_10_20.md`.

**NOTA:** no usar las cifras de bare `TOP_LEVEL → identity` (11+5 / 264+49 Protos, 12+5 / 286+51 GraalPy) como sustitutas de esta forma `run → identity`.
