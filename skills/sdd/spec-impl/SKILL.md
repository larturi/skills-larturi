---
name: spec-impl
description: 'Implementa una spec en estado Aprobado (en cualquier idioma): pide una confirmación al inicio, corre todos los pasos del plan delegando cada uno a un fork, y cierra con una verificación final contra los criterios de aceptación. Deja la spec en Implementado o Implementado con observaciones. Sobre una spec con observaciones, ofrece resolverlas u omitirlas. Por defecto trabaja en la rama principal; con AutoCreateBranch true usa una rama spec-NN-slug.'
disable-model-invocation: true
argument-hint: <NN-nombre-spec>
allowed-tools: Read, Glob, Grep, Edit, Write, AskUserQuestion, Agent, SendMessage, Bash(git status:*), Bash(git branch:*), Bash(git checkout:*), Bash(git log:*), Bash(git diff:*), Bash(cat:*), Bash(ls:*), Bash(date:*), mcp__plugin_engram_engram__mem_current_project, mcp__plugin_engram_engram__mem_search, mcp__plugin_engram_engram__mem_context, mcp__plugin_engram_engram__mem_save, mcp__plugin_engram_engram__mem_session_summary
---

# /spec-impl - Implementador de specs aprobadas

## Contexto de sesión

Fecha de hoy (usar esta al actualizar el roadmap, nunca adivinarla):
!`date +%F`

Estado actual del repositorio:
!`git status --short`

Rama actual:
!`git branch --show-current`

Rama principal del repo (main o master):
!`git branch --list main master`

Specs disponibles:
!`ls specs/ 2>/dev/null || echo "La carpeta specs/ no existe"`

Configuración:
!`cat specs/.spec-config.yml 2>/dev/null || echo "AutoCreateBranch: false (default, sin archivo de config)"`

---

## Instrucciones

Seguí las fases en orden. No avances si la anterior no se completó.

Estados que escribe esta skill (siempre en español, salvo que las specs del repo usen otro idioma): `Implementado`, `Implementado con observaciones`. Nunca escribe `Aprobado` ni `Publicado`.

---

### Fase 0 - Engram

Fijate si en esta sesión están disponibles las herramientas de Engram (`mem_search`, `mem_save`, `mem_session_summary`).

- **Disponible:** usalo en las Fases 3 y 4 para traer contexto previo y guardar decisiones, bugs y convenciones no obvias.
- **No disponible:** avisá una sola vez y seguí:

  ```
  ℹ️ Engram no está disponible en esta sesión: las decisiones y bugs de esta
  implementación van a quedar solo en este chat.
  ```

---

### Fase 1 - Identificar la spec

El argumento recibido es: `$ARGUMENTS`

- Vacío: listá las specs disponibles, pedí el nombre y pará.
- Con valor: buscá en `specs/` por nombre completo (`01-mvp-arkanoid`), número (`01`) o slug (`mvp-arkanoid`). Si no aparece, listá las disponibles y pedí que corrija.

Roadmap (opcional): si existe `specs/00-roadmap.md` en estado `Activo` (o equivalente) y un único ítem tiene en `Spec` la ruta exacta de esta spec, conservá ese vínculo. Sin coincidencias aproximadas. En cualquier otro caso no toques el roadmap.

---

### Fase 2 - Validar el estado

Leé la spec y ubicá la línea de estado (cerca del encabezado; etiqueta `**Estado:**`, `**Status:**` o equivalente). Identificá el valor por significado, en cualquier idioma.

| Estado | Acción |
|--------|--------|
| `Aprobado` / `Approved` / equivalente | Modo normal: Fase 3. |
| `Implementado con observaciones` / equivalente | Modo retomar: Fase R. |
| `Borrador`, `Implementado`, `Publicado`, `Obsoleto`, otro o no encontrado | Parar con el mensaje de abajo. |

Si dudás del significado, no asumas: pedí que lo aclare.

```
❌ No puedo implementar esta spec.

Estado actual: [ESTADO ENCONTRADO]
Solo trabajo con specs en "Aprobado" (implementar) o "Implementado con
observaciones" (retomar).

- Si la spec está lista, cambiá el estado a "Aprobado" a mano.
- Si necesita trabajo, editala o usá /spec-init.
- Si ya está "Implementado", el siguiente paso es /spec-finish.
```

No ofrezcas arrancar igual. El bloqueo es intencional.

---

### Fase 3 - Preparar la rama y confirmar

**3.1 Rama.** Leé `AutoCreateBranch` de la configuración de arriba. Solo un `true` explícito lo habilita; cualquier otra cosa es `false`.

- **`false` (default):**
  - Si la rama actual es la principal: seguí sin comentar nada.
  - Si no: preguntá con `AskUserQuestion` - "Cambiar a <principal>" o "Quedarme en <rama actual>". Si elige cambiar y el working tree tiene cambios, avisá que viajan con el checkout antes de ejecutarlo.
- **`true`:** rama `spec-NN-slug` (nombre del archivo sin extensión, con prefijo `spec-`).
  - Si el working tree no está limpio, mostrá los cambios y preguntá: "Commitearlos o guardarlos vos y volver a correr (recomendado)" o "Continuar: los cambios viajan a la rama". No hagas stash ni commit por tu cuenta.
  - Si la rama no existe: `git checkout -b spec-NN-slug`.
  - Si existe: `git checkout spec-NN-slug`, leé `git log --oneline`, decí qué pasos parecen hechos y desde cuál proponés retomar. Esperá confirmación.

**3.2 Contexto previo.** Si Engram está disponible, `mem_search` con el slug y palabras del objetivo. Si aparece algo relevante, mencionalo.

**3.3 Resumen.** Mostrá objetivo, alcance, plan y criterios de aceptación de la spec (identificá las secciones por significado, no por redacción exacta).

**3.4 Confirmación única.** Con `AskUserQuestion`:

```
Voy a implementar "<NN-slug>" así:
  - Corro todos los pasos del plan seguidos (solo paro ante una ambigüedad
    bloqueante o un fallo que no cede).
  - Trabajo en la rama <rama activa>.
  - Al final verifico todos los criterios de aceptación.
  - Dejo la spec en Implementado, o en Implementado con observaciones si
    algo no pasa o necesita tu confirmación.

¿Arrancamos así?
```

Opciones: **Sí, dale** (recomendada) o **No, quiero ajustar algo** (preguntá qué y adaptá la ejecución, sin asumir el ajuste).

---

### Fase 4 - Implementar (un fork por paso)

No vuelvas a pedir confirmación.

Si hay un ítem de roadmap vinculado, pasalo a `Estado: En progreso` y actualizá `Actualizado` con la fecha de hoy. Sin entrada al historial. Si la implementación se corta, queda `En progreso`.

**Por qué forks:** cada paso genera mucho ruido de herramientas (lecturas, ediciones, typecheck, tests). Un fork (`subagent_type: "fork"`) hereda esta conversación, comparte cache y deja ese ruido fuera del contexto principal. Corre en el mismo working directory: edita el repo real.

**Regla:** un fork por paso, nunca en paralelo. Apenas un paso termina bien, lanzá el siguiente sin esperar al usuario.

**Prompt de cada paso:**

```
Implementá exactamente el Paso <N> del plan de la spec que estamos
trabajando: "<texto literal del paso>".

Alcance estricto: solo este paso. Implementá la spec tal cual, no una
versión mejorada; si algo te parece subóptimo, mencionalo en el reporte.

Calidad: funciones chicas, sin duplicación, sin comentarios superfluos ni
expresiones crípticas, buenas prácticas de seguridad. Seguí las
convenciones del proyecto (CLAUDE.md, AGENTS.md, linter, código
circundante).

Antes de reportar terminado:
- Corré typecheck, linter y tests relevantes con los comandos propios del
  proyecto, y arreglá los tests existentes que tu cambio rompa.
- Nunca corras migraciones de base de datos ni comandos destructivos o que
  afecten sistemas compartidos: si hace falta uno, reportá el comando
  exacto para que lo corra el usuario.
- Nunca hagas commit.
- Nunca hagas escritura en la base de datos.

Si encontrás una ambigüedad que la spec no resuelve: no la resuelvas.
Pará y reportala con 2-3 opciones concretas.

Si un typecheck/lint/test falla y no logra corregirse: pará y reportá el
comando y el error real.

Reporte final (conciso): archivos tocados con una línea cada uno,
resultado de typecheck/lint/tests, ambigüedades u observaciones.
```

Lanzalo con `Agent({ subagent_type: "fork", name: "spec-impl-paso-<N>", description: "Implementar Paso <N>", prompt })`. Avisá en una línea ("Trabajando en el Paso <N>...") y terminá el turno. No inventes progreso.

**Cuando llega el reporte:**

- **Terminó:** `Paso N completado (<archivos>). Sigo con el Paso N+1.` y lanzá el siguiente.
- **Ambigüedad:** presentala con sus opciones (`AskUserQuestion`). Con la decisión, retomá el mismo fork con `SendMessage({ to: "spec-impl-paso-<N>", message })`. Recién después seguí.
- **Fallo que no cede:** descartá ese fork. Lanzá uno nuevo para el mismo paso (`spec-impl-paso-<N>-intento-<n>`) con el error anterior al principio del prompt. Máximo 2 reintentos. Si persiste, mostrá el historial y preguntá: "intentar con instrucciones tuyas", "saltar el paso y anotarlo como observación" o "parar acá".
- **Engram:** si el reporte trae una decisión no obvia, un bug con causa raíz o una convención nueva, `mem_save`. Sin ruido.

**Nunca commitear**, ni el coordinador ni los forks.

**Fuera de alcance:** si el usuario pide algo que la spec no incluye, recordáselo y sugerí anotarlo para otra spec. No lo implementes.

---

### Fase 5 - Verificación final y cierre

**5.1** Lanzá un fork `spec-impl-verificacion-final`:

```
Verificación final de la spec que acabamos de implementar.

1. Corré la suite completa de typecheck/lint/tests del proyecto.
2. Verificá cada criterio de aceptación de forma concreta (comando o test
   si existe):

   <lista completa de criterios>

   Si un criterio requiere algo que no podés hacer (inspección visual, UI
   manual, entorno que no tenés), marcalo "no verificable" - nunca asumas
   que pasa ni que falla.

Nunca hagas commit.

Reportá un checklist: ✅ pasó, ❌ falló (con el error concreto), ⚠️ no
verificable, más el resultado global de typecheck/lint/tests.
```

Avisá en una línea y terminá el turno.

**5.2** Con el reporte, mostrá el checklist y seguí:

- **Todo ✅ y sin fallos globales:** cierre exitoso (5.3).
- **Hay ⚠️:** pedí confirmación puntual en texto. Si confirma que pasan, se tratan como ✅.
- **Hay ❌, fallos globales, ⚠️ que el usuario dice que no pasan, o ⚠️ sin respuesta todavía:**
  1. Agregá a la spec `## Observaciones` con un bullet `[pendiente]` de una línea por cada uno (criterio - motivo). Ver formato en `spec-init/template.md`. No agregues nada más a la spec.
  2. Estado `Implementado con observaciones`. El roadmap queda `En progreso`.
  3. Ofrecé las opciones de la Fase R (resolver / omitir / dejar así).

**5.3 Cierre exitoso:**

- Estado `Implementado`.
- Roadmap vinculado: ítem `Estado: Hecho`, `Actualizado` con la fecha de hoy; si todos los ítems comprometidos quedaron `Hecho`, roadmap `Completo`. Sin entrada al historial.
- Nunca escribas `Publicado`.

**5.4** Si Engram está disponible, `mem_session_summary` (Goal, Discoveries, Accomplished, Next Steps, Relevant Files) antes del mensaje final.

Mensaje final si quedó `Implementado`:

```
✅ Spec implementada y verificada. Estado: Implementado.

Siguiente paso: /spec-finish <NN-slug> - audita los cambios y te deja el
mensaje de commit listo. El commit lo hacés vos.
```

Mensaje final si quedó con observaciones:

```
⚠️ Spec implementada con observaciones (ver ## Observaciones en la spec).
Estado: Implementado con observaciones.

Para cerrarla, volvé a correr /spec-impl <NN-slug>: podés resolverlas u
omitirlas.
```

---

### Fase R - Retomar una spec con observaciones

Se entra desde la Fase 2 (spec ya en `Implementado con observaciones`) o directo desde 5.2.

1. Mostrá los bullets `[pendiente]` de `## Observaciones`. Si la sección no existe, corré la verificación final (5.1) para reconstruirla.
2. Preguntá con `AskUserQuestion`:
   - **Resolverlas** (recomendada): lanzá un fork de corrección dirigido solo a esos puntos (mismas reglas del prompt de paso), y después la verificación final (5.1). Los que pasen se borran de `## Observaciones`; si no queda ninguno, borrá la sección y cerrá con 5.3.
   - **Omitirlas:** el usuario acepta cerrar así. Cambiá cada `[pendiente]` a `[aceptada]` y cerrá con 5.3 (`Implementado`, roadmap `Hecho`).
   - **Dejarlas así:** no cambies nada y terminá.
3. Si se entra desde la Fase 2, antes de lanzar forks aplicá la Fase 3.1 (rama) y confirmá en una línea en qué rama vas a trabajar.

---

## Reglas duras

- Nunca commitear.
- Nunca escribir en la base de datos.
- Nunca escribir `Aprobado` ni `Publicado`.
- Nunca implementar fuera del alcance de la spec.
- En la spec solo se tocan el estado y `## Observaciones`.
- Puntuación: nunca uses el carácter de guion largo. Usá siempre `-`.

## Resumen

```
/spec-impl 01-mvp   (Aprobado)
  F1-F2  encuentra la spec, valida estado
  F3     rama (default: principal; pregunta si estás en otra), resumen, 1 confirmación
  F4     un fork por paso, sin pausas salvo ambigüedad o fallo
  F5     verificación final → Implementado | Implementado con observaciones

/spec-impl 01-mvp   (Implementado con observaciones)
  FR     muestra pendientes → resolver | omitir | dejar así

/spec-impl 02-x     (Borrador)
  F2     ❌ para, no toca nada
```
