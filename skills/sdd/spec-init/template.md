# Plantilla de spec

Referencia de `/spec-init`. Es la forma a respetar, no texto para copiar.

**Prioridad: que la spec se relea en un par de minutos.** Una línea por idea, sin prosa, sin repetir. Orientativo: 60-80 líneas. Si no entra, la feature es demasiado grande: dividirla.

---

## Estructura

```markdown
# SPEC NN - Título corto

> **Estado:** Borrador
> **Depende de:** SPEC 01, SPEC 02
> **Fecha:** YYYY-MM-DD
> **Objetivo:** Una sola oración.

## Alcance

**Entra:**

- Cosa concreta uno.

**Fuera:**

- Algo mencionado que se difiere a otra spec.

## Modelo de datos

\`\`\`js
const state = { level: 1, score: 0, highScores: [] }; // highScores: { score, level, date }
\`\`\`

## Plan

1. Crear `src/levels.js` con el esqueleto. Prueba: el juego carga igual que antes.
2. Implementar `nextLevel()`. Prueba: al romper el último ladrillo sube a nivel 2.

## Criterios de aceptación

- [ ] Romper un ladrillo suma exactamente 10 puntos.
- [ ] Recargar la página preserva los high-scores.

## Decisiones

- Sí: localStorage - entra en <5MB y no hay queries.
- No: IndexedDB - overengineering para este caso.

## Riesgos

- localStorage deshabilitado en modo privado → fallback en memoria.
```

## Reglas por sección

| Sección | Obligatoria | Regla |
|---------|-------------|-------|
| Encabezado | Sí | Objetivo en una oración. `Depende de` se omite si no hay dependencias. |
| Alcance | Sí | Bullets de una línea. `Fuera` es explícito: evita meter cosas "ya que estamos". |
| Modelo de datos | No | Solo si hay estructuras nuevas o cambiadas. Código real, corto. Si no aplica, se omite sin comentario. |
| Plan | Sí | Una línea por paso: qué se hace + cómo se prueba. Cada paso deja el sistema funcional. Si un paso pasa de ~50 líneas de código, dividirlo. El último paso no es "testear todo". |
| Criterios de aceptación | Sí | Una línea booleana cada uno. Nada de "funciona bien", "buena UX", "sin bugs". |
| Decisiones | Sí | `Sí:` / `No:` + razón en una línea. Es la sección con más valor a futuro. |
| Riesgos | No | Solo riesgos no obvios, `riesgo → mitigación` en una línea. |
| Observaciones | No | La escribe solo `/spec-impl`. Ver abajo. |

## Estados

`Borrador`, `Aprobado`, `Implementado con observaciones`, `Implementado`, `Publicado`, `Obsoleto`. Quién escribe cada uno: ver el README del grupo `sdd`.

Al leer se aceptan equivalentes en otros idiomas (`Draft`, `Approved`, `Implemented`, ...). Si el repo ya usa otro idioma en sus specs, mantener el de las specs existentes.

## Observaciones (solo `/spec-impl`)

Cuando la verificación final no pasa del todo, `/spec-impl` agrega al final:

```markdown
## Observaciones

- [pendiente] Recargar la página preserva los high-scores - falla: la key no se lee al iniciar.
- [aceptada] El menú se ve bien en mobile - no verificable, el usuario lo acepta así.
```

`[pendiente]`: falló o falta confirmar. `[aceptada]`: el usuario decidió cerrarlo así. Una línea cada una, sin logs.

## Reglas globales

- Nombres concretos: `src/levels.js`, no "el módulo de niveles".
- Sin TODOs: una decisión se toma o se registra como pendiente con su razón.
- Sin código ejecutable largo: la spec describe, el código viene después.
- Markdown estándar que renderice en GitHub.
