---
name: spec-finish
description: 'Cierra una spec en estado Implementado. Audita los cambios (con rama: spec-NN-slug contra main; sin rama: el working tree sin commitear en main) con la misma metodologia 4R de spec-pre-commit como gate obligatorio - bloquea si el veredicto es NO COMMITEAR. Si pasa, prepara el cierre (squash staged con rama, o deja el working tree como esta sin rama) y redacta un mensaje de commit que resume la spec. Nunca ejecuta git commit ni borra la rama - eso lo hace siempre el usuario a mano.'
disable-model-invocation: true
argument-hint: '[NN-nombre-spec] (opcional, infiere de la rama actual si no se pasa)'
allowed-tools: Read, Glob, Grep, AskUserQuestion, Agent, Bash(git status:*), Bash(git branch:*), Bash(git checkout:*), Bash(git log:*), Bash(git diff:*), Bash(git merge:*), Bash(cat:*), Bash(ls:*), Bash(date:*), mcp__plugin_engram_engram__mem_current_project, mcp__plugin_engram_engram__mem_search, mcp__plugin_engram_engram__mem_save, mcp__plugin_engram_engram__mem_session_summary
---

# /spec-finish - Cierre de una spec implementada

## Contexto de sesión

Fecha de hoy:
!`date +%F`

Estado actual del repositorio:
!`git status --short`

Rama actual:
!`git branch --show-current`

Rama principal del repo (main o master):
!`git branch --list main master`

Specs disponibles en esta carpeta:
!`ls specs/ 2>/dev/null || echo "La carpeta specs/ no existe"`

---

## Instrucciones

Seguí estas fases (0 a 6) en orden estricto. **No avances a la fase siguiente si la anterior no se completó correctamente.**

---

### Fase 0 - Detectar si hay memoria persistente (Engram)

Antes de identificar la spec, fijate si en esta sesión tenés disponible el protocolo de Engram (herramientas `mem_search`, `mem_save`, `mem_session_summary` - se anuncian como "core tools" al arrancar la sesión cuando el plugin está activo).

- **Si Engram está disponible:** vas a usarlo en la Fase 6 para guardar un resumen del cierre.
- **Si Engram NO está disponible:** decíselo al usuario en una sola línea, sin bloquear el flujo, y seguí:

  ```
  ℹ️ No tenés Engram configurado en esta sesión - el cierre de esta spec no va
  a quedar en memoria persistente entre sesiones. Si querés que los próximos
  cierres arranquen con ese contexto, activá el plugin engram.
  ```

  No lo vuelvas a mencionar en el resto de la ejecución.

---

### Fase 1 - Identificar la spec y el camino (con rama o sin rama)

El argumento recibido es: `$ARGUMENTS`

Si `$ARGUMENTS` tiene un valor: buscá el archivo en `specs/`. El usuario puede haber escrito el nombre completo (`01-mvp-arkanoid`), solo el número (`01`), o solo el slug (`mvp-arkanoid`). Intentá encontrar el archivo correcto en cualquiera de esos casos. Si no lo encontrás, mostrá las specs disponibles y pedile que corrija el nombre.

Si `$ARGUMENTS` está vacío: mirá la rama actual (contexto de sesión de arriba). Si sigue el patrón `spec-NN-slug`, usá esa spec. Si la rama actual ya es `main`/`master`, o no sigue ese patrón, listá las specs en estado `Implementado` (leé cada archivo de `specs/` para determinarlo) y pedile al usuario que aclare cuál cerrar. Parar y esperar respuesta.

Una vez identificada la spec (`NN-slug`), determiná el camino:

- **Con rama:** existe una rama local `spec-NN-slug` distinta de la rama principal (`git branch --list spec-NN-slug` no está vacío, y esa rama tiene commits propios sobre la principal).
- **Sin rama:** no existe esa rama y los cambios de la spec están sin commitear en la rama actual (normalmente la principal). Es lo esperado con `AutoCreateBranch: false` (el default).

Detectá cuál es la rama principal del repo a partir del contexto de sesión de arriba (`main` si existe, si no `master`). Conservá internamente el camino elegido - determina cómo se comportan las Fases 3 y 5.

---

### Fase 2 - Validar el estado de la spec

Leé el archivo de spec completo. Ubicá el campo de estado por posición (cerca del encabezado) y por la máquina de estados circundante, no por la etiqueta exacta - puede estar en cualquier idioma.

**Regla absoluta:** Solo podés continuar si el estado **significa "Implementado"**, en cualquier idioma (`Implementado`, `Implemented`, `Implémenté`, ...). `Implementado con observaciones` **no** es válido: todavía tiene pendientes.

Cualquier otro estado significa **parar** y mostrar el mensaje de error correspondiente:

```
❌ No puedo cerrar esta spec.

Estado actual: [ESTADO ENCONTRADO]
Solo cierro specs en "Implementado".

Para continuar:
  - "Borrador" o "Aprobado": falta implementar. Corré /spec-impl [nombre].
  - "Implementado con observaciones": hay pendientes. Corré /spec-impl
    [nombre] para resolverlos u omitirlos.
  - "Publicado": ya está cerrada. No hay nada más que hacer.
  - "Obsoleto": ya no aplica.
```

Si no estás seguro de si un valor significa lo que corresponde, **no asumas**. Parar y pedirle al usuario que aclare.

---

### Fase 3 - Mostrar el diff y auditar (delegado a un fork)

Mostrale al usuario el diff a auditar, según el camino de la Fase 1:

- **Con rama:** `git diff <principal>...spec-NN-slug --stat`.
- **Sin rama:** `git status --short` más `git diff --stat` del working tree.

Lanzá un fork de auditoría con el mismo patrón que usa `spec-impl` para delegar trabajo (`Agent({ subagent_type: "fork", name: "spec-finish-auditoria", description: "Auditar el cierre de la spec", prompt: <lo de abajo> })`):

```
Aplicá exactamente la metodología de la skill spec-pre-commit. Si podés
invocarla como skill, hacelo; si no, buscá su SKILL.md entre las skills
instaladas (ej. ~/.claude/skills/spec-pre-commit/SKILL.md o la carpeta de
skills del agente) y leelo completo. Si no la encontrás, decilo en el
reporte en vez de improvisar otra metodología. Aplicala completa: las cuatro
dimensiones 4R (Risk, Readability, Reliability, Resilience), el bloque de
higiene de commit, el mismo formato de hallazgos y el mismo veredicto
(LISTO PARA COMMIT / COMMIT CON CAMBIOS MENORES / NO COMMITEAR).

La única diferencia: en vez de auditar `git diff --staged`, auditá <"el diff de
la rama spec-NN-slug contra <principal> (git diff <principal>...spec-NN-slug)"
si el camino es con rama, o "los cambios sin commitear en el working tree de
la rama actual (git diff y git status --short)" si el camino es sin rama - completá
según corresponda>.

Nunca hagas commit ni ningún cambio en el código - esto es una auditoría de
solo lectura.

Reportá el resultado completo de la auditoría (hallazgos por bloque, con
severidad, ubicación, qué, por qué, cómo) y el veredicto final.
```

Avisale al usuario en una línea que estás auditando y terminá el turno - no inventes el resultado mientras el fork corre.

---

### Fase 4 - Resultado de la auditoría

Cuando llegue el reporte del fork, mostrale al usuario el resultado completo (hallazgos + veredicto).

- **Si el veredicto es `NO COMMITEAR`:** parar ahí. No avances a la Fase 5. Mostrale los hallazgos críticos y preguntale cómo seguir (podés usar `AskUserQuestion`): arreglar ahora (podés lanzar un fork de corrección dirigido a esos hallazgos, mismo patrón que el rewind de `spec-impl`) o corregir a mano y volver a correr `/spec-finish` después.
- **Si el veredicto es `LISTO PARA COMMIT` o `COMMIT CON CAMBIOS MENORES`:** seguí a la Fase 5. Si hubo advertencias o sugerencias menores, mencionáselas al usuario de todos modos antes de continuar.

---

### Fase 5 - Preparar el cierre (nunca commitea)

Redactá un mensaje de commit propuesto: un título corto con el objetivo de la spec y debajo los criterios de aceptación cumplidos. Si la spec tiene bullets `[aceptada]` en `## Observaciones`, listalos aparte (no como cumplidos):

```
<Objetivo de la spec en una línea>

Criterios de aceptación cumplidos:
- <criterio 1>
- <criterio 2>

Observaciones aceptadas:
- <criterio> - <motivo>
```

Omití el bloque de observaciones si no hay.

Mostraselo al usuario y confirmá vía `AskUserQuestion` si preparar el cierre con ese mensaje - opciones "Sí, preparalo" y "No, quiero ajustar el mensaje" (en ese caso, esperá su versión y usá esa en vez de la tuya).

Si confirma:

- **Camino con rama:** si no estás ya en la rama principal, `git checkout <principal>`. Después `git merge --squash spec-NN-slug` - esto deja el diff completo de la rama staged en la principal, sin crear ningún commit (`git merge --squash` nunca commitea por sí solo).
- **Camino sin rama:** no ejecutes ningún comando. Los cambios quedan sin commitear y sin stage; el usuario los agrega. Si en el working tree hay archivos ajenos a la spec (no relacionados con su plan ni con lo auditado), listalos aparte para que no los agregue por error.

**Regla dura, sin excepciones: nunca ejecutes `git commit` ni ningún comando que cree un commit, en ningún punto de esta skill.** El commit final es siempre una acción manual del usuario.

---

### Fase 6 - Mensaje final

Mostrale al usuario el mensaje de commit propuesto, formateado y listo para usar (por ejemplo con `git commit -F -` pegando el mensaje, o copiándolo a mano), y recordale que:

- **Con rama:** el diff ya está staged; revisarlo con `git diff --staged` y commitear.
- **Sin rama:** agregar los archivos de la spec con `git add <archivos>` (dale la lista exacta), revisar con `git diff --staged` y commitear.
- Si fue el camino con rama, una vez que commitee puede borrar la rama con `git branch -d spec-NN-slug` a mano - no se lo preguntes ahora ni lo hagas vos, mencionalo solo como paso posterior disponible.
- El pase de la spec a `Publicado` sigue siendo una decisión manual y posterior (por ejemplo, después de un deploy) - esta skill no toca el campo `Status`.

**Si Engram está disponible** (ver Fase 0): antes del mensaje final, llamá `mem_session_summary` con Goal (cierre de la spec `NN-slug`), Discoveries (resultado de la auditoría), Accomplished (diff preparado para commit), Next Steps (commit manual del usuario, y borrado de rama si aplica) y Relevant Files (la spec y los archivos del diff).

```
✅ Cierre preparado para "NN-slug".

La auditoría dio [LISTO PARA COMMIT / COMMIT CON CAMBIOS MENORES].
- Con rama: el diff está staged en <principal>.
- Sin rama: agregá los archivos con git add <lista>.
Revisá con git diff --staged y commiteá vos con el mensaje de arriba.
```

---

## Reglas duras

- **Nunca ejecutar `git commit`** bajo ninguna circunstancia, en ningún camino.
- **Nunca borrar la rama automáticamente.** Solo mencionarlo como paso manual posterior al commit.
- **Nunca modificar el campo `Status` de la spec.** Ni siquiera marcarla como `Publicado` - eso es siempre una edición manual del usuario, ajena a esta skill.
- **Nunca avanzar con un veredicto `NO COMMITEAR`.** El bloqueo es intencional, igual que la validación de estado de la Fase 2.
- **Puntuación:** nunca uses el carácter de guion largo. Usá siempre `-`.
