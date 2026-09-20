# Plantilla para una spec útil

Este archivo es la referencia que consulta el skill `/spec` al generar specs. Cada sección incluye su propósito y un ejemplo mínimo. **No es texto para copiar textualmente** — es la forma que el skill debe respetar.

---

## Encabezado

Toda spec arranca con metadata en formato blockquote (sin tablas, sin bloques, simple como se muestra abajo):

```markdown
# SPEC NN — Título corto y descriptivo

> **Status:** Draft
> **Depends on:** SPEC 01, SPEC 02
> **Date:** YYYY-MM-DD
> **Objective:** Una sola oración. Si necesitás dos oraciones, la feature es demasiado grande.
```

**Estados válidos:** `Draft`, `In review`, `Approved`, `Implemented`, `Obsolete`.

> Las etiquetas de arriba son los defaults en inglés. Los skills también aceptan equivalentes en cualquier idioma (ej. en español: `Borrador` / `En revisión` / `Aprobado` / `Implementado` / `Obsoleto`). Elegí un set por repo y mantené consistencia.

**Regla del objetivo:** una oración que una persona lea en 5 segundos y entienda qué se va a construir. Si no entra en una oración, dividí la feature.

---

## Sección 1 — Por qué existe esta spec (opcional)

Para specs que toman decisiones no obvias o rompen patrones del proyecto, una sección breve explicando el **por qué** del trabajo. No el qué — el qué viene después.

Para specs simples, omitirla.

---

## Sección 2 — Alcance

Dos sub-bloques explícitos. **Ambos son obligatorios.**

```markdown
## Scope

**In:**

- Cosa concreta uno.
- Cosa concreta dos.

**Out of scope (for future specs):**

- Algo que se podría hacer pero no ahora.
- Algo que surgió en la conversación pero no está incluido.
```

**Por qué importa el "out":** captura las cosas que el usuario mencionó durante la fase de preguntas pero que se decidió diferir. Sin ese registro, durante la implementación va a haber tentación de meterlas de contrabando "ya que estamos".

---

## Sección 3 — Modelo de datos

Las estructuras concretas que aparecen o cambian. Usá código real, no pseudocódigo abstracto.

```markdown
## Data model

\`\`\`js
// Estado del juego
const state = {
level: 1,
score: 0,
highScores: [/* { score, level, date } */],
};
\`\`\`

Convenciones:

- Coordenadas: origen arriba a la izquierda.
- Velocidades en píxeles/frame.
```

Si la feature no introduce datos nuevos, escribilo explícitamente: _"Esta feature no introduce estructuras de datos nuevas. Reutiliza el modelo de SPEC 01."_

---

## Sección 4 — Plan de implementación

Pasos numerados. Cada paso debe dejar el sistema en un estado **funcional y ejecutable**. Nada de "implementar la mitad y seguir mañana".

```markdown
## Implementation plan

1. Crear archivo X con un esqueleto vacío.
2. Implementar función A en X. Test manual: correr Y, ver Z.
3. Conectar X con el módulo existente W.
4. ...
```

**Reglas:**

- Cada paso debe poder commitearse por sí solo.
- Si un paso requiere más de 30–50 líneas de código, dividilo.
- El último paso del plan **no** es "testear todo" — eso son los criterios de aceptación.

---

## Sección 5 — Criterios de aceptación

Checklist booleano. Cada ítem se puede verificar con sí o no.

```markdown
## Acceptance criteria

- [ ] El juego carga sin errores en la consola.
- [ ] Romper un ladrillo suma exactamente 10 puntos.
- [ ] Recargar la página preserva los high-scores.
```

**Anti-patrones a evitar:**

- ❌ "Que funcione bien." → no verificable.
- ❌ "Buena UX." → subjetivo.
- ❌ "Sin bugs." → no operacionalizable.
- ✅ "Presionar Esc pausa el juego y muestra el menú." → verificable, booleano.

---

## Sección 6 — Decisiones tomadas y descartadas

La sección con más valor dentro de 3 meses. Capturá **qué consideraste**, no solo qué elegiste.

```markdown
## Decisions

- **Yes:** localStorage para persistencia. Entra en <5MB y no necesitamos queries.
- **No:** IndexedDB. Overengineering para este caso.
- **Yes:** key versionada (`save:v1`). Permite migrar el esquema después sin romper nada.
- **No:** sync en la nube. Va en otra spec si algún día se hace.
```

Cada decisión idealmente tiene una razón breve. Las decisiones sin razón son las primeras que se van a cuestionar después.

---

## Sección 7 — Riesgos identificados (opcional)

Solo cuando hay riesgos no obvios. Tabla simple:

```markdown
## Risks

| Riesgo                                    | Mitigación                                                                       |
| ------------------------------------------ | --------------------------------------------------------------------------------- |
| localStorage deshabilitado en modo privado | Fallback a objeto en memoria. El juego sigue funcionando, solo que no persiste.  |
| Esquema incompatible a futuro              | La key incluye `:v1`. Migración documentada en `persistence.js`.                 |
```

Para specs chicas o features muy acotadas, omitirla.

---

## Sección final — Qué NO está incluido (refuerzo)

Repetir explícitamente al final qué **no** se va a hacer en esta spec. Esta repetición es deliberada — la sección de Alcance ya lo dice, pero al final del documento sirve como recordatorio cuando alguien lee solo las últimas líneas.

```markdown
## What is **not** in this spec

- Editor visual (otra spec si algún día se hace).
- Multiplayer.
- Versión mobile.

Cada una de esas, si se hace, va en su propia spec.
```

---

## Reglas globales sobre todo el documento

- **Una oración por idea.** Si una oración tiene dos comas y un punto y coma, dividila.
- **Nombres concretos.** Si decís "el módulo de niveles", decí `src/levels.js`. Si decís "una key", dá el string exacto.
- **Sin TODOs.** Un TODO en una spec significa que la decisión no se tomó. Tomala o anotala como decisión pendiente con una razón.
- **Sin código largo ejecutable.** La spec describe; el código se escribe después. Snippets cortos para ilustrar estructuras de datos están bien; funciones completas no.
- **Markdown estándar.** Sin extensiones raras. Debe renderizar en GitHub sin sorpresas.
