# Plantilla para un roadmap útil

Este archivo es la referencia que consulta el skill `/spec-plan` al generar el roadmap. Cada sección incluye su propósito. **No es texto para copiar textualmente** — es la forma que el skill debe respetar.

---

## Encabezado

```markdown
# Roadmap — Nombre corto del proyecto

> **Status:** Planning
> **Date:** YYYY-MM-DD
> **Vision:** Una o dos oraciones. Qué problema resuelve el sistema y para quién.
```

---

## Alcance de la v1

Dos sub-bloques explícitos, igual que el Alcance de una spec individual.

```markdown
## Scope de la v1

**In:**

- Capacidad concreta uno.
- Capacidad concreta dos.

**Out of scope (para versiones futuras):**

- Algo que se mencionó pero se decidió diferir.
```

---

## Arquitectura de alto nivel

Componentes principales y decisiones estructurales ya tomadas. Nivel de detalle: el necesario para justificar el orden del roadmap, no para implementar.

```markdown
## Arquitectura

- Stack: ...
- Componentes principales: ...
- Decisiones ya tomadas: ...
```

Si una decisión sigue abierta y no bloquea el orden del roadmap, anotala como pendiente explícitamente — no la escondas.

---

## Roadmap de specs

La sección central. Lista ordenada, no tabla larga con detalle de implementación — eso vive en cada spec.

```markdown
## Roadmap de specs

1. **slug-corto-uno** — Objetivo en una oración. Depende de: nada. Tamaño: S. Estado: Pendiente.
2. **slug-corto-dos** — Objetivo en una oración. Depende de: (1). Tamaño: M. Estado: Pendiente.
3. **slug-corto-tres** — Objetivo en una oración. Depende de: (1), (2). Tamaño: S. Estado: Pendiente.
```

**Reglas:**

- Cada ítem tiene que caber en una sola spec de `/spec-init` (algo resoluble en horas, no días).
- Tamaño es relativo entre ítems de este mismo roadmap: S/M/L.
- Estado arranca en `Pendiente`. Se actualiza a `En progreso` cuando se corre `/spec-init` sobre ese ítem, y a `Hecho` cuando la spec resultante llega a `Implemented`.
- El slug de cada ítem es una sugerencia de punto de partida para `/spec-init`, no un compromiso — puede ajustarse al escribir la spec real.

---

## Riesgos identificados

Solo riesgos de nivel proyecto (no de una feature puntual): elección de proveedor, límites de escala, dependencias externas críticas.

```markdown
## Riesgos

| Riesgo | Mitigación / cuándo se decide |
| --- | --- |
| ... | ... |
```

Omitir si no hay riesgos no obvios.

---

## Cómo seguir

Cierre fijo del documento:

```markdown
## Cómo seguir

Ejecutá `/spec-init` con la descripción del próximo ítem pendiente (en orden, salvo que el usuario prefiera otro). Cuando esa spec llegue a `Approved`, corré `/spec-impl` para implementarla. Al terminar, volvé a este archivo y marcá la fila como `Hecho` antes de pasar a la siguiente.
```

---

## Reglas globales sobre todo el documento

- **Sin plan de implementación paso a paso, sin modelo de datos, sin criterios de aceptación.** Eso es contenido de spec individual, no de roadmap.
- **Una oración por objetivo de ítem.** Si necesita más, es una señal de que el ítem es en realidad dos.
- **Markdown estándar.** Debe renderizar en GitHub sin sorpresas.
