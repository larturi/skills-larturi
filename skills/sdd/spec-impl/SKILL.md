---
name: spec-impl
description: 'Implementa una spec aprobada. Valida que el estado signifique "Approved" (en cualquier idioma); solo si AutoCreateBranch esta en true crea una rama de git con el nombre de la spec, cambia a ella, y arranca la implementación paso a paso con pausas para revisar los diffs. Caso contrario no pregunta nada y trabaja directo en la rama main.'
disable-model-invocation: true
argument-hint: <NN-nombre-spec>
allowed-tools: Read, Glob, Grep, Edit, Write, AskUserQuestion, Agent, SendMessage, Bash(git status:*), Bash(git branch:*), Bash(git checkout:*), Bash(git log:*), Bash(git diff:*), Bash(git stash:*), Bash(cat:*), Bash(ls:*), Bash(date:*), mcp__plugin_engram_engram__mem_current_project, mcp__plugin_engram_engram__mem_search, mcp__plugin_engram_engram__mem_context, mcp__plugin_engram_engram__mem_save, mcp__plugin_engram_engram__mem_session_summary
---

# /spec-impl - Implementador de specs aprobadas

## Contexto de sesión

Fecha de hoy (usar esta al actualizar el roadmap, nunca adivinarla):
!`date +%F`

Estado actual del repositorio:
!`git status --short`

Rama actual:
!`git branch --show-current`

Specs disponibles en esta carpeta:
!`ls specs/ 2>/dev/null || echo "La carpeta specs/ no existe"`

Configuración de creación de rama:
!`cat specs/.spec-config.yml 2>/dev/null || echo "AutoCreateBranch: false (default, sin archivo de config)"`

---

## Instrucciones

Seguí estas cinco fases en orden estricto. **No avances a la fase siguiente si la anterior no se completó correctamente.**
-
---
-
### Fase 0 - Detectar si hay memoria persistente (Engram)

Antes de identificar la spec, fijate si en esta sesión tenés disponible el protocolo de Engram (herramientas `mem_search`, `mem_context`, `mem_save`, `mem_session_summary` - se anuncian como "core tools" al arrancar la sesión cuando el plugin está activo).

- **Si Engram está disponible:** vas a usarlo durante el resto de la implementación (Fase 3 y Fase 4) para buscar contexto previo relevante y guardar de forma proactiva las decisiones, bugs y convenciones no obvias que vayan apareciendo. No hace falta avisarle nada al usuario por esto, salvo que encuentres contexto previo relevante para la spec (ver Fase 3).
- **Si Engram NO está disponible** (no hay proto-olo de Engram activo en esta sesión): decíselo al usuario en una sola línea, sin bloquear el flujo, y seguí:

  ```
  ℹ️ No tenés Engram configurado en esta sesión - voy a implementar la spec sin
  guardar memoria persistente entre sesiones (decisiones, bugs y convenciones
  van a quedar solo en este chat). Si querés que las próximas implementaciones
  arranquen con ese contexto, activá el plugin engram.
  ```

  No insistas ni lo vuelvas a mencionar en el resto de la ejecución.
-
---

### Fase 1 - Identificar la spec

El argumento recibido es: `$ARGUMENTS`

Si `$ARGUMENTS` está vacío:

- Listá los archivos disponibles en `specs/` (ya los tenés arriba).
- Pedile al usuario que especifique el nombre exacto de la spec.
- Parar y esperar respuesta. No continuar.

Si `$ARGUMENTS` tiene un valor:

- Buscá el archivo en `specs/`. El usuario puede haber escrito el nombre completo (`01-mvp-arkanoid`), solo el número (`01`), o solo el slug (`mvp-arkanoid`). Intentá encontrar el archivo correcto en cualquiera de esos casos.
- Si no encontrás el archivo, mostrá las specs disponibles y pedile al usuario que corrija el nombre.
- Si lo encontrás, comprobá si existe `specs/00-roadmap.md`. La integración es opcional: solo conservá internamente un vínculo si el roadmap está `Active` y contiene la ruta exacta de esta spec en el campo `Spec` de un único ítem. No busques coincidencias aproximadas por slug durante la implementación. Si no hay roadmap, está `Complete`, `Planning` o `Paused`, o no hay un vínculo exacto y único, continuá sin modificarlo.
- Después c-ntinuá a la Fase 2.

---

### Fase 2 - Validar el estado de la spec

Leé el archivo de spec que ubicaste en la Fase 1 usando la herramienta Read o `-at`.

En el contenido del archivo, buscá la línea que contiene el estado de la spec. La etiqueta del encabezado típicamente es `**Status:**` (inglés) o `**Estado:**` (español), pero puede estar en cualquier idioma. Identificala por posición (línea de estado cerca del inicio de la spec) y por la máquina de estados circundante, no por la etiqueta exacta.

**Regla absoluta:** Solo podés continuar si el estado **significa "Approved"** - sin importar el idioma usado.

Tratá cualquiera de los siguientes (y sus equivalentes en otros idiomas) como el estado **Approved** y continuá:

- Inglés: `Approved`
- Español: `Aprobado`
- Portugués: `Aprovado`
- Francés: `Approuvé`
- Alemán: `Genehmigt`
- Italiano: `Approvato`
- …o cualquier otra palabra en otro idioma que claramente signifique "aprobado"

Cualquier otra cosa (Draft / Borrador, In review / En revisión, Implemented / Implementado, Obsolete / Obsoleto, o cualquier valor no reconocido) significa **parar** y mostrar el mensaje de error de abajo.

| Categoría de estado                          | Ejemplos (cualquier idioma)                        | Acción                                                                        |
| --------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------ |
| Approved                                      | `Approved`, `Aprobado`, `Aprovado`, `Approuvé`, …  | Continuar a la Fase 3.                                                        |
| Draft                                         | `Draft-, `Borrador`, …                              | Parar. Mostrar el mensaje de error de abajo.                                  |
| In review                                     | `In review`, `En revisión`, …                       | Parar. Mostrar el mensaje de error de abajo.                                  |
| Implemented                                   | `Implemented`, `Implementado`, …                    | Parar. Mostrar el mensaje de error de abajo.                                  |
| Obsolete                                      | `Obsolete`, `Obsoleto`, …                            | Parar. Mostrar el mensaje de error de abajo.                                  |
| Línea de estado no encontrada / valor no reconocido | -                                              | Parar. El archivo no sigue el formato esperado. Decírselo al usuario.         |

Si no estás seguro de si un valor significa "aprobado", **no asumas**. Parar y pedirle al usuario que aclare o que actualice la spec con la redacción canónica.

**Mensaje de error estándar cuando el estado no significa Approved:**

```
❌ No puedo implementar esta spec.

Estado actual: [ESTADO ENCONTRADO]
Solo trabajo con specs cuyo estado signifique "Approved" (ej.: `Approved`, `Aprobado`,
o el equivalente en otro idioma).

Para continuar tenés dos opciones:
  1. Si la spec está lista para implementarse, abrila y cambiá el estado
     a "Approved" (o el término equivalente que use tu equipo) manualmente.
     Ese cambio lo hace la persona, no el agente.
  2. Si la spec todavía necesita trabajo, usá /spec-init [nombre] para retomarla.
```

No ofrezcas-alternativas, no sugieras "puedo arrancar igual si querés". El bloqueo es intencional.

---

### Fase 3 - Crear la rama de git y cambiar a ella (por defecto no crear la rama, solo si AutoCreateBranch: true)

Una vez que confirmaste que el estado significa `Approved`:

0. **Verificá primero el working tree.** Mirá la salida de `git status --short` en el contexto de sesión de arriba. Si **no está vacía**, parar y mostrar los cambios pendientes, después preguntar:

   ```-
   ⚠️ Hay cambios sin commitear en el working tree.
   Cambiar de rama los va a arrastrar. ¿Qué querés hacer?
     1. Commitearlos o guardarlos con stash vos mismo, y volver a correr este comando  (recomendado)
     2. Continuar igual - los cambios viajan a la rama nueva
   ```

   Esperar la respuesta. **No hagas stash ni commit en nombre del usuario** a menos que lo pida explícitamente. Si el working tree está limpio, saltá directo al paso 1 sin mencionarlo.

1. Derivá el nombre de la rama a partir del nombre completo del archivo de spec, sin la extensión. Formato: `spec-NN-slug`. Ejemplos:

   - `01-mvp-arkanoid.md` → rama `spec-01-mvp-arkanoid`
   - `02-powerups.md` → rama `spec-02-powerups`

2. Leé el flag `AutoCreateBranch` de la **configuración de creación de rama** mostrada en el contexto de sesión de arriba.

   - Si el archivo de config no existe, el valor falta, o el valor no se reconoce → tratalo como `false` (el default).
   - Solo un `true` explícito (en cualquier capitalización) habilita la creación automática de rama.

   **Si `AutoCreateBranch` es `false` (default):** no preguntar nada ni comentar sobre ramas. Asumí directamente que se trabaja en `main` (o `master`, la rama principal del repo) y seguí. Si la rama actual no es `main`, cambiá a `main` con `git checkout main` sin pedir confirmación. No crear ninguna rama nueva.

   **Si `AutoCreateBranch` es `true`:** proceder sin preguntar.

   - Si la rama **no existe**: creala con `git checkout -b spec-NN-slug`.
   - Si **ya existe**: esto significa que se está retomando trabajo previo. Cambiá a ella, leé `git log --oneline` en la rama, y decile al usuario qué pasos del plan ya parecen hechos y desde cuál proponés retomar. Esperá confirmación sobre el punto de retoma antes de implementar nada.
   - En ambos casos: cambiá a la rama con `git checkout spec-NN-slug` y confirmá que el cambio fue exitoso antes de continuar.

3. Confirmale visualmente al usuario que la spec está lista y qué rama está activa:

   ```
   ✅ Listo para implementar.

   Spec:   specs/NN-slug.md
   Rama:   spec-NN-slug  (activa)   (← o la rama actual, si no se creó una rama nueva)
   Estado: Approved   (← repetir el valor real encontrado en la spec)
   ```

4. **Si Engram está disponible** (ver Fase 0): antes de mostrar el resumen, llamá `mem_search` con el nombre/slug de la spec y palabras clave de su objetivo, para ver si hay decisiones, bugs o convenciones de sesiones anteriores relacionados con esta feature. Si aparece algo relevante, mencionáselo al usuario junto con el resumen (p. ej. "Encontré en la memoria que la vez pasada se decidió X").
-
5. **Todavía no empieces a implementar.** Primero m-strale al usuario el resumen de la spec para que la tenga fresca. Extraé y mostrá:
   - El **objetivo** (la línea después de `**Objective:**` / `**Objetivo:**` / equivalente).
   - El **alcance** (la sección `## Scope` / `## Alcance` / equivalente).-
   - El **plan de implementación** (la sección con los pasos numerados - `## Implementation plan` / `## Plan de implementación` / equivalente).
   - Los **criterios de aceptación** (el checklist - `## Acceptance criteria` / `## Criterios de aceptación` / equivalente).

Identificá -os títulos de sección por significado, no por redacción exacta - la spec puede estar escrita en cualquier idioma.

6. **Confirmación explícita antes de arrancar.** Usá `AskUserQuestion` para confirmar que el usuario está de acuerdo con cómo va a correr la implementación. La pregunta debe declarar estas cuatro condiciones con los datos concretos de esta ejecución (nombre de la spec, rama activa):

   ```
   Voy a implementar "<NN-slug>" así:
     - Corro todos los pasos del plan seguidos, sin pausar entre pasos
       (salvo que un fork reporte una ambigüedad bloqueante).
     - Trabajo en la rama <rama activa>.
     - Al terminar el último paso, corro una verificación final de punta
       a punta contra los criterios de aceptación.
     - Marco la spec como Implemented (o Implementado con observaciones,
       si la verificación final encuentra algo que no pasa) automáticamente.

   ¿Arrancamos así?
   ```

   Ofrecé dos opciones:
   - **Sí, dale** (recomendada): arrancá directo con el Paso 1 de la Fase 4, corriendo todos los pasos de corrido tal como se declaró.
   - **No, quiero ajustar algo**: parate ahí y pedile al usuario que aclare qué quiere cambiar (ejemplos: pausar después de cada paso, revisar el alcance antes de arrancar). Adaptá el resto de la ejecución a lo que pida - no asumas cuál es el ajuste.

---

### Fase 4 - Implementar paso a paso (delegado a subagentes fork)

La confirmación para arrancar ya se obtuvo en la Fase 3 (punto 6) - no la vuelvas a pedir acá.

Si en la Fase 1 encontraste un ítem de roadmap vinculado de forma exacta y única, actualizá solamente ese ítem a `Estado: En progreso` y el campo `Updated` del roadmap con la fecha actual. No agregues una entrada al historial: es una transición operativa, no una revisión estructural. Si la implementación se interrumpe o falla, dejá `En progreso`, porque describe correctamente el estado real.

**Por qué delegar a un fork:** cada paso del plan típicamente implica leer varios archivos, e-itarlos, correr typecheck/lint/tests y a vece- arreglar tests existentes que el cambio rompió. Ese trabajo genera mucho ruido de herramientas que no aporta nada a la conversación una vez terminado - solo el resultado importa. Un fork (Agent tool, `subagent_type: "fork"`) hereda toda esta conversación (la spec, las convenciones ya descubiertas, las decisiones ya tomadas) así que no necesita re-explicación, comparte el cache de contexto, y su ruido de herramientas queda fuera de esta conversación. Esto no acelera el reloj de pared - los pasos son secuenciales y cada uno puede depender del anterior - pero evita que specs largas de muchos pasos terminen compactando o saturando el contexto a mitad de camino. El fork corre en el mismo working directory que el coordinador (sin aislamiento de worktree): edita el repo real.

**Regla:** un fork por paso, nunca en paralelo. Lanzá el fork del Paso N+1 apenas el Paso N haya terminado exitosamente (sin ambigüedad bloqueante pendiente) - no hace falta esperar a que el usuario revise el diff primero. Incluso un paso que parezca trivial conviene delegarlo igual, para no romper la consistencia del flujo - el costo de un fork es bajo porque comparte tu cache.

**Cómo delegar cada paso:**

1. Armá un prompt de fork directivo y acotado a ese paso específico (no repitas toda la spec - el fork ya la tiene en su contexto heredado - pero sé explícito sobre el alcance exacto y lo que tiene que reportar):

   ```
   Implementá exactamente el Paso <N> del plan de implementación de la spec
   que estamos trabajando: "<pegá acá el texto literal del paso, tal como
   aparece en la spec>".

   Alcance estricto: solo este paso. No toques nada que corresponda a otro
   paso del plan, aunque lo veas relacionado o rompiendo la compilación por
   ahora.-

   Calidad de código: priorizá simplicidad - funciones chicas, sin
   duplicación (DRY), principios SOLID donde aplique. Evitá comentarios
   superfluos (el código debe ser auto-documentado) y expresiones crípticas;
   preferí siempre claridad. Aplicá buenas prácticas de seguridad cuando
   corresponda. Seguí las convenciones de estilo e idioma que ya existan en
   el proyecto (CLAUDE.md, linter, código circundante); si no hay ninguna
   definida, priorizá lo anterior por defecto.

   Antes de reportar terminado:
   - Corré el typecheck y el linter del proyecto sobre el código tocado
     (revisá package.json si no sabés los comandos exactos).
   - Corré la suite de tests relevante (o completa si es rápida) y arreglá
     cualquier test existente que tu cambio haya roto - no lo dejes para
     después.
   - Nunca corras comandos de Prisma migrate/db push/studio ni ningún
     comando destructivo o que afecte sistemas compartidos: si hace falta
     uno, avisalo en tu reporte con el comando exacto para que el usuario
     lo corra manualmente.
   - Nunca hagas commit.

   Si te encontrás con una ambigüedad que la spec no resuelve: NO la
   resuelvas por tu cuenta. Detené el trabajo en ese punto y en tu reporte
   final describí la ambigüedad con precisión y 2-3 opciones concretas, en
   vez de entregar una implementación completa.

   Si corriste el typecheck/lint/tests, encontraste un fallo, e intentaste
   corregirlo sin éxito (esto es distinto de una ambigüedad de diseño - es
   un fallo real que no cede): NO sigas intentando indefinidamente ni
   reportes que terminaste. Detené el trabajo y en tu reporte final
   describí exactamente qué falló - el comando que corriste y el
   error/output real - para que quien reintente este paso arranque
   informado.

   Reportá en tu mensaje final (es lo único que va a leer el coordinador,
   sé completo pero conciso):
   - Lista de archivos tocados, con una línea de qué cambiaste en cada uno.
   - Resultado de typecheck/lint/tests.-
   - Cualquier ambigüedad, desvío del plan u observación relevante.
   ```

2. Lanzá el fork: `Agent({ subagent_type: "fork", name: "spec-impl-paso-<N>", description: "Implementar Paso <N> de la spec", prompt: <lo de arriba> })`.--

3. Avisale al usuario en una línea que estás trabajando en el paso (p. ej. "Trabajando en el Paso <N>...") y terminá el turno. No inventes progreso ni resultado mientras el fork corre - la notificación llega sola en un turno posterior.

4. Cuando llegue la notificación del fork:

   - **Si reportó una ambigüedad bloqueante:** no la resuelvas vos. Presentale al usuario la ambigüedad y las opciones tal como las trajo el fork (podés usar `AskUserQuestion`). Cuando el usuario decida, retomá **el mismo fork** - no lances uno nuevo para el mismo paso - con `SendMessage({ to: "spec-impl-paso-<N>", message: "<la decisión del usuario>" })` para que termine el trabajo. Recién cuando termine, seguí con el paso siguiente.
   - **Si terminó el paso:** avisale al usuario en una línea corta que el paso terminó y qué archivos tocó, y lanzá directo el fork del Paso N+1 - no hace falta esperar que confirme antes de seguir. El usuario puede revisar el diff mientras el paso siguiente ya está corriendo.

     ```
     Paso N completado (<archivos tocados>). Sigo con el Paso N+1.
     ```

   - **Si reportó un fallo de verificación que no logró resolver** (distinto de ambigüedad - typecheck/lint/tests que no cedieron): descartá ese fork - no le mandes `SendMessage`, no lo retomes. Llevá la cuenta de reintentos de ese paso (arranca en 0 con el fork original).

     - **Si todavía no hiciste 2 reintentos:** lanzá un fork **nuevo** para el mismo Paso N (nombralo `spec-impl-paso-<N>-intento-<n>` para distinguirlo en los logs), con el mismo prompt del paso más un párrafo al principio con el error concreto que reportó el intento anterior, para que arranque informado en vez de repetir el mismo camino a ciegas.
     - **Si ya hiciste 2 reintentos** (3 intentos en total contando el fork original) y el fallo persiste: parar, mostrarle al usuario el historial de los intentos con el error de cada uno, y preguntarle cómo seguir (`AskUserQuestion`, opciones tipo "intentar una vez más con instrucciones tuyas", "saltar este paso por ahora y anotarlo", "parar acá"). No lances un cuarto fork sin que el usuario decida.

   - **Si Engram está disponible** (ver Fase 0): revisá el reporte del fork antes de lanzar el paso siguiente. Si menciona una decisión no obvia, un bug arreglado (con su causa raíz), una convención nueva o una ambigüedad que el usuario terminó resolviendo, guardalo con `mem_save`. No guardes ruido (qué archivos se tocaron, resultados de test que pasaron sin drama) - solo lo que le sirva de contexto a una sesión futura.

**Nunca commitear automáticamente.** Ni el coordinador ni los forks. Ni por paso, ni al final. Vos escribís el código y mostrás el diff; commitear es decisión del usuario y orden del usuario. Solo commitear si lo pide explícitamente.

**Una regla por encima de todas:** implementá lo que dice la spec. Si algo en la spec te parece subóptimo, mencionalo como observación pero implementá lo acordado. Los cambios a la spec van en la spec, no en el código por sorpresa. Esto aplica también al prompt de cada fork - dejale claro que implemente la spec tal cual, no una versión mejorada.

**Si el usuario pide algo que está fuera del alcance de la spec:**

- Recordarle que está fuera del alcance de esta spec.
- Sugerir anotarlo para la próxima spec.
- No implementarlo en esta rama (ni delegarlo a un fork).

**Al terminar el último paso:**

1. **Verificación final obligatoria.** Antes de pedirle nada al usuario, lanzá un fork dedicado a verificar la implementación completa contra la spec:

   ```
   Corré una verificación final de la spec que acabamos de implementar.

   1. Corré la suite COMPLETA de typecheck/lint/tests del proyecto (no
      solo lo tocado en el último paso - el proyecto entero).
   2. Para cada uno de estos criterios de aceptación, intentá verificarlo
      de forma concreta (corré el comando/test que lo compruebe si existe
      una forma automática de hacerlo):

      <pegá acá la lista completa de criterios de aceptación de la spec>

      Si un criterio requiere algo que no podés hacer (inspección visual,
      interacción manual de UI, algo que depende de un entorno que no
      tenés), marcalo como "no verificable automáticamente" - nunca
      asumas que pasa ni que falla.

   Nunca hagas commit.

   Reportá en tu mensaje final un checklist claro: criterios que pasaron
   (✅), que fallaron (❌, con el error concreto), y los no verificables
   automáticamente (⚠️), más el resultado de typecheck/lint/tests global.
   ```

   Lanzalo con `Agent({ subagent_type: "fork", name: "spec-impl-verificacion-final", description: "Verificación final de la spec", prompt: <lo de arriba> })`. Avisale al usuario en una línea que estás corriendo la verificación final y terminá el turno - no inventes el resultado mientras corre.

2. Cuando llegue el reporte del fork de verificación final, mostrale al usuario el checklist completo y seguí uno de estos tres caminos:

   - **Caso A - todo pasó:** ningún criterio verificable automáticamente falló y no hay fallos de typecheck/lint/tests. Si quedaron criterios marcados "no verificable automáticamente", pedile al usuario que confirme esos puntualmente (no hace falta `AskUserQuestion` para esto, alcanza con texto simple). Cuando confirme (o si no había ninguno pendiente):
     - actualizá el estado de la spec a `Implemented` (o el equivalente en el idioma del documento);
     - si hay un ítem de roadmap vinculado exactamente, cambialo a `Estado: Hecho`, actualizá `Updated`, y si todos los ítems comprometidos quedaron `Hecho` cambiá el estado general del roadmap a `Complete` (sin entrada al historial, transición operativa);
     - **nunca escribas el estado `Released`** - es siempre una edición manual posterior del usuario. Mencionalo en el mensaje final.

   - **Caso B - algo no pasó:** algún criterio falló, o typecheck/lint/tests tienen fallos, o el usuario dice que algún "no verificable automáticamente" en realidad no pasa. Mostrale qué no pasó y por qué:
     - actualizá el estado de la spec a `Implementado con observaciones` (no `Implemented`);
     - **no sincronices el roadmap a `Hecho`** - si había un ítem vinculado, dejalo en `En progreso`;
     - ofrecele reintentar lo que falló (podés lanzar un fork de corrección dirigido al fallo específico, mismo patrón que el rewind de un paso) o cerrar así por ahora y retomarlo después corriendo `/spec-impl` de nuevo sobre esta misma spec.

   - **Caso C - hay criterios "no verificables automáticamente" y el usuario todavía no confirmó** si pasan o no: no marques ni `Implemented` ni `Implementado con observaciones` todavía - la spec queda en `Approved`. Esperá la confirmación antes de cerrar.

3. Si Engram está disponible (ver Fase 0), llamá `mem_session_summary` antes del mensaje final, con Goal (la spec implementada), Discoveries (lo guardado paso a paso con `mem_save` durante la Fase 4, incluyendo el resultado de la verificación final), Accomplished (los pasos completados y el resultado de cierre), Next Steps (`/spec-finish` si quedó `Implemented`, o corregir y reintentar si quedó `Implementado con observaciones`) y Relevant Files (los archivos tocados a lo largo de la implementación).

Mensaje final del Caso A:

```
✅ Todos los pasos del plan están implementados y la verificación final pasó.

La spec quedó en "Implemented" (o el equivalente en el idioma del repo).
Cuando quieras cerrar la rama, corré /spec-finish - audita los cambios y hace
el squash-merge. El pase a "Released" es manual, cuando decidas que corresponde
(por ejemplo, después de un deploy).
```

Mensaje final del Caso B:

```
⚠️ Todos los pasos del plan están implementados, pero la verificación final
encontró algo que no pasa (ver el detalle arriba).

La spec quedó en "Implementado con observaciones". Corregí lo que falta y
volvé a correr /spec-impl sobre esta misma spec para reverificar.
```

**Regla de estilo:** nunca uses el carácter de guion largo. Usá siempre `-`.

---

## Resumen del comportamiento esperado

```
/spec-impl 01-mvp-arkanoid

  Fase 1  →  Encuentra specs/01-mvp-arkanoid.md
  Fase 2  →  Lee el estado → "Approved" (o "Aprobado", etc.) → ✅ continúa-
  Fase 3  →  git checkout -b spec-01-mvp-arkanoid → git checkout spec-01-mvp-arkanoid
              Muestra objetivo, alcance, plan y criterios
              Confirmación explícita vía AskUserQuestion (corre todo seguido,
              rama, verificación final, cierre automático)
  Fase 4  →  Delega cada paso a un fork, sin pausar entre pasos (salvo ambigüedad
              o fallo de verificación con rewind)
              Al terminar, un fork de verificación final chequea todo de punta
              a punta: si todo pasa, marca Implemented y sincroniza el roadmap;
              si algo no pasa, marca Implementado con observaciones sin tocar
              el roadmap. Released queda siempre para el usuario. Siguiente
              paso sugerido: /spec-finish

/spec-impl 02-powerups  (estado: Draft / Borrador)

  Fase 1  →  Encuentra specs/02-powerups.md
  Fase 2  →  Lee el estado → "Draft" → ❌ para
              Muestra el mensaje de error estándar
              No crea rama, no toca código
```

**La creación de rama está controlada por el flag `AutoCreateBranch`** en `specs/.spec-config.yml`. Por defecto es `false` (la Fase 3 no crea ninguna rama ni pregunta nada - trabaja directo en `main`). Ponelo en `true` para que cree y cambie a `spec-NN-slug` automáticamente, como se muestra arriba.
