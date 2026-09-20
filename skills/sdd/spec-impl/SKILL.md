---
name: spec-impl
description: 'Implementa una spec aprobada. Valida que el estado signifique "Approved" (en cualquier idioma); solo si AutoCreateBranch esta en true crea una rama de git con el nombre de la spec, cambia a ella, y arranca la implementación paso a paso con pausas para revisar los diffs. Caso contrario no pregunta nada y trabaja directo en la rama main.'
disable-model-invocation: true
argument-hint: <NN-nombre-spec>
allowed-tools: Read, Glob, Grep, Edit, Write, AskUserQuestion, Agent, SendMessage, Bash(git status:*), Bash(git branch:*), Bash(git checkout:*), Bash(git log:*), Bash(git diff:*), Bash(git stash:*), Bash(cat:*), Bash(ls:*)
---

# /spec-impl — Implementador de specs aprobadas

## Contexto de sesión

Estado actual del repositorio:
!`git status --short`

Rama actual:
!`git branch --show-current`

Specs disponibles en esta carpeta:
!`ls specs/ 2>/dev/null || echo "La carpeta specs/ no existe"`

Configuración de creación de rama:
!`cat specs/.spec-config.yml 2>/dev/null || echo "AutoCreateBranch: true (default, sin archivo de config)"`

---

## Instrucciones

Seguí estas cuatro fases en orden estricto. **No avances a la fase siguiente si la anterior no se completó correctamente.**

---

### Fase 1 — Identificar la spec

El argumento recibido es: `$ARGUMENTS`

Si `$ARGUMENTS` está vacío:

- Listá los archivos disponibles en `specs/` (ya los tenés arriba).
- Pedile al usuario que especifique el nombre exacto de la spec.
- Parar y esperar respuesta. No continuar.

Si `$ARGUMENTS` tiene un valor:

- Buscá el archivo en `specs/`. El usuario puede haber escrito el nombre completo (`01-mvp-arkanoid`), solo el número (`01`), o solo el slug (`mvp-arkanoid`). Intentá encontrar el archivo correcto en cualquiera de esos casos.
- Si no encontrás el archivo, mostrá las specs disponibles y pedile al usuario que corrija el nombre.
- Si lo encontrás, continuá a la Fase 2.

---

### Fase 2 — Validar el estado de la spec

Leé el archivo de spec que ubicaste en la Fase 1 usando la herramienta Read o `cat`.

En el contenido del archivo, buscá la línea que contiene el estado de la spec. La etiqueta del encabezado típicamente es `**Status:**` (inglés) o `**Estado:**` (español), pero puede estar en cualquier idioma. Identificala por posición (línea de estado cerca del inicio de la spec) y por la máquina de estados circundante, no por la etiqueta exacta.

**Regla absoluta:** Solo podés continuar si el estado **significa "Approved"** — sin importar el idioma usado.

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
| Draft                                         | `Draft`, `Borrador`, …                              | Parar. Mostrar el mensaje de error de abajo.                                  |
| In review                                     | `In review`, `En revisión`, …                       | Parar. Mostrar el mensaje de error de abajo.                                  |
| Implemented                                   | `Implemented`, `Implementado`, …                    | Parar. Mostrar el mensaje de error de abajo.                                  |
| Obsolete                                      | `Obsolete`, `Obsoleto`, …                            | Parar. Mostrar el mensaje de error de abajo.                                  |
| Línea de estado no encontrada / valor no reconocido | —                                              | Parar. El archivo no sigue el formato esperado. Decírselo al usuario.         |

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

No ofrezcas alternativas, no sugieras "puedo arrancar igual si querés". El bloqueo es intencional.

---

### Fase 3 — Crear la rama de git y cambiar a ella (por defecto no crear la rama, solo si AutoCreateBranch: true)

Una vez que confirmaste que el estado significa `Approved`:

0. **Verificá primero el working tree.** Mirá la salida de `git status --short` en el contexto de sesión de arriba. Si **no está vacía**, parar y mostrar los cambios pendientes, después preguntar:

   ```
   ⚠️ Hay cambios sin commitear en el working tree.
   Cambiar de rama los va a arrastrar. ¿Qué querés hacer?
     1. Commitearlos o guardarlos con stash vos mismo, y volver a correr este comando  (recomendado)
     2. Continuar igual — los cambios viajan a la rama nueva
   ```

   Esperar la respuesta. **No hagas stash ni commit en nombre del usuario** a menos que lo pida explícitamente. Si el working tree está limpio, saltá directo al paso 1 sin mencionarlo.

1. Derivá el nombre de la rama a partir del nombre completo del archivo de spec, sin la extensión. Formato: `spec-NN-slug`. Ejemplos:

   - `01-mvp-arkanoid.md` → rama `spec-01-mvp-arkanoid`
   - `02-powerups.md` → rama `spec-02-powerups`

2. Leé el flag `AutoCreateBranch` de la **configuración de creación de rama** mostrada en el contexto de sesión de arriba.

   - Si el archivo de config no existe, el valor falta, o el valor no se reconoce → tratalo como `true` (el default).
   - Solo un `false` explícito (en cualquier capitalización) deshabilita la creación automática de rama.

   **Si `AutoCreateBranch` es `true` (default):** proceder sin preguntar.

   - Si la rama **no existe**: creala con `git checkout -b spec-NN-slug`.
   - Si **ya existe**: esto significa que se está retomando trabajo previo. Cambiá a ella, leé `git log --oneline` en la rama, y decile al usuario qué pasos del plan ya parecen hechos y desde cuál proponés retomar. Esperá confirmación sobre el punto de retoma antes de implementar nada.
   - En ambos casos: cambiá a la rama con `git checkout spec-NN-slug` y confirmá que el cambio fue exitoso antes de continuar.

   **Si `AutoCreateBranch` es `false`:** no preguntar nada ni comentar sobre ramas. Asumí directamente que se trabaja en `main` (o `master`, la rama principal del repo) y seguí. Si la rama actual no es `main`, cambiá a `main` con `git checkout main` sin pedir confirmación. No crear ninguna rama nueva.

3. Confirmale visualmente al usuario que la spec está lista y qué rama está activa:

   ```
   ✅ Listo para implementar.

   Spec:   specs/NN-slug.md
   Rama:   spec-NN-slug  (activa)   (← o la rama actual, si no se creó una rama nueva)
   Estado: Approved   (← repetir el valor real encontrado en la spec)
   ```

4. **Todavía no empieces a implementar.** Primero mostrale al usuario el resumen de la spec para que la tenga fresca. Extraé y mostrá:
   - El **objetivo** (la línea después de `**Objective:**` / `**Objetivo:**` / equivalente).
   - El **alcance** (la sección `## Scope` / `## Alcance` / equivalente).
   - El **plan de implementación** (la sección con los pasos numerados — `## Implementation plan` / `## Plan de implementación` / equivalente).
   - Los **criterios de aceptación** (el checklist — `## Acceptance criteria` / `## Criterios de aceptación` / equivalente).

Identificá los títulos de sección por significado, no por redacción exacta — la spec puede estar escrita en cualquier idioma.

---

### Fase 4 — Implementar paso a paso (delegado a subagentes fork)

Después de mostrar el resumen de la spec, decile al usuario:

```
Voy a implementar la spec siguiendo el plan de implementación exactamente.
Cada paso lo delego a un subagente (fork) para no acumular en esta conversación
el ruido de cada lectura/edición/test — vos y yo solo vemos el resumen y el diff.
Voy a pausar después de cada paso para que lo revises.

¿Arrancamos con el Paso 1?
```

Esperar confirmación explícita ("sí", "dale", "adelante", o equivalente). No empezar sin ella.

**Por qué delegar a un fork:** cada paso del plan típicamente implica leer varios archivos, editarlos, correr typecheck/lint/tests y a veces arreglar tests existentes que el cambio rompió. Ese trabajo genera mucho ruido de herramientas que no aporta nada a la conversación una vez terminado — solo el resultado importa. Un fork (Agent tool, `subagent_type: "fork"`) hereda toda esta conversación (la spec, las convenciones ya descubiertas, las decisiones ya tomadas) así que no necesita re-explicación, comparte el cache de contexto, y su ruido de herramientas queda fuera de esta conversación. Esto no acelera el reloj de pared — los pasos son secuenciales y cada uno puede depender del anterior — pero evita que specs largas de muchos pasos terminen compactando o saturando el contexto a mitad de camino. El fork corre en el mismo working directory que el coordinador (sin aislamiento de worktree): edita el repo real.

**Regla:** un fork por paso, nunca en paralelo. No lances el fork del Paso N+1 hasta que el Paso N esté confirmado por el usuario. Incluso un paso que parezca trivial conviene delegarlo igual, para no romper la consistencia del flujo — el costo de un fork es bajo porque comparte tu cache.

**Cómo delegar cada paso:**

1. Armá un prompt de fork directivo y acotado a ese paso específico (no repitas toda la spec — el fork ya la tiene en su contexto heredado — pero sé explícito sobre el alcance exacto y lo que tiene que reportar):

   ```
   Implementá exactamente el Paso <N> del plan de implementación de la spec
   que estamos trabajando: "<pegá acá el texto literal del paso, tal como
   aparece en la spec>".

   Alcance estricto: solo este paso. No toques nada que corresponda a otro
   paso del plan, aunque lo veas relacionado o rompiendo la compilación por
   ahora.

   Antes de reportar terminado:
   - Corré el typecheck y el linter del proyecto sobre el código tocado
     (revisá package.json si no sabés los comandos exactos).
   - Corré la suite de tests relevante (o completa si es rápida) y arreglá
     cualquier test existente que tu cambio haya roto — no lo dejes para
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

   Reportá en tu mensaje final (es lo único que va a leer el coordinador,
   sé completo pero conciso):
   - Lista de archivos tocados, con una línea de qué cambiaste en cada uno.
   - Resultado de typecheck/lint/tests.
   - Cualquier ambigüedad, desvío del plan u observación relevante.
   ```

2. Lanzá el fork: `Agent({ subagent_type: "fork", name: "spec-impl-paso-<N>", description: "Implementar Paso <N> de la spec", prompt: <lo de arriba> })`.

3. Avisale al usuario en una línea que estás trabajando en el paso (p. ej. "Trabajando en el Paso <N>...") y terminá el turno. No inventes progreso ni resultado mientras el fork corre — la notificación llega sola en un turno posterior.

4. Cuando llegue la notificación del fork:

   - **Si reportó una ambigüedad bloqueante:** no la resuelvas vos. Presentale al usuario la ambigüedad y las opciones tal como las trajo el fork (podés usar `AskUserQuestion`). Cuando el usuario decida, retomá **el mismo fork** — no lances uno nuevo para el mismo paso — con `SendMessage({ to: "spec-impl-paso-<N>", message: "<la decisión del usuario>" })` para que termine el trabajo.
   - **Si terminó el paso:** mostrale al usuario el resumen que trajo el fork (archivos tocados + resultado de verificación) y decile:

     ```
     Paso N completado. ¿Podés revisar el diff y avisarme si sigo con el Paso N+1?
     ```

   - Esperar confirmación antes de lanzar el fork del paso siguiente.

**Nunca commitear automáticamente.** Ni el coordinador ni los forks. Ni por paso, ni al final. Vos escribís el código y mostrás el diff; commitear es decisión del usuario y orden del usuario. Solo commitear si lo pide explícitamente.

**Una regla por encima de todas:** implementá lo que dice la spec. Si algo en la spec te parece subóptimo, mencionalo como observación pero implementá lo acordado. Los cambios a la spec van en la spec, no en el código por sorpresa. Esto aplica también al prompt de cada fork — dejale claro que implemente la spec tal cual, no una versión mejorada.

**Si el usuario pide algo que está fuera del alcance de la spec:**

- Recordarle que está fuera del alcance de esta spec.
- Sugerir anotarlo para la próxima spec.
- No implementarlo en esta rama (ni delegarlo a un fork).

**Al terminar el último paso:**

```
✅ Todos los pasos del plan están implementados.

Próximo paso: verificar los criterios de aceptación de la spec uno por uno.
Si todos pasan, actualizá el estado de la spec a "Implemented" (o el equivalente
en el idioma de tu repo).

Antes del commit final, corré /spec-pre-commit sobre los cambios staged.
```

---

## Resumen del comportamiento esperado

```
/spec-impl 01-mvp-arkanoid

  Fase 1  →  Encuentra specs/01-mvp-arkanoid.md
  Fase 2  →  Lee el estado → "Approved" (o "Aprobado", etc.) → ✅ continúa
  Fase 3  →  git checkout -b spec-01-mvp-arkanoid → git checkout spec-01-mvp-arkanoid
              Muestra objetivo, alcance, plan y criterios
  Fase 4  →  Delega cada paso a un fork, pausa después de cada uno
              Termina recordando verificar los criterios de aceptación
              y correr /spec-pre-commit antes del commit final

/spec-impl 02-powerups  (estado: Draft / Borrador)

  Fase 1  →  Encuentra specs/02-powerups.md
  Fase 2  →  Lee el estado → "Draft" → ❌ para
              Muestra el mensaje de error estándar
              No crea rama, no toca código
```

**La creación de rama está controlada por el flag `AutoCreateBranch`** en `specs/.spec-config.yml`. Por defecto es `true` (crea la rama automáticamente, como se muestra arriba). Ponelo en `false` para que la Fase 3 no cree ninguna rama ni pregunte nada — trabaja directo en `main`.
