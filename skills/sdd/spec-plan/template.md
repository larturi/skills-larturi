# Plantilla para un roadmap útil

Este archivo es la referencia que consulta el skill `/spec-plan` al generar el roadmap. Cada sección incluye su propósito. **No es texto para copiar textualmente** - es la forma que el skill debe respetar.

---

## Encabezado

```markdown
# Roadmap - Nombre corto del proyecto

> **Status:** Planning
> **Created:** YYYY-MM-DD
> **Updated:** YYYY-MM-DD
> **Vision:** Una o dos oraciones. Qué problema resuelve el sistema y para quién.
```

Estados válidos del roadmap: `Planning` mientras se define, `Active` cuando ya se están ejecutando sus specs, `Paused` si se detuvo deliberadamente y `Complete` cuando todos los ítems comprometidos están hechos. El estado del roadmap es independiente del estado de cada spec.

---

## Alcance

Dos sub-bloques explícitos, igual que el Alcance de una spec individual.

```markdown
## Scope

**In:**

- Capacidad concreta uno.
- Capacidad concreta dos.

**Out:**

- Algo que se decidió explícitamente no construir dentro de este roadmap.
```

`Out` contiene exclusiones, no ideas candidatas. Las capacidades valiosas diferidas van en **Ideas para etapas futuras**.

---

## Arquitectura de alto nivel

Componentes principales y decisiones estructurales ya tomadas. Nivel de detalle: el necesario para justificar el orden del roadmap, no para implementar.

```markdown
## Arquitectura

- Stack: ...
- Componentes principales: ...
- Decisiones confirmadas: ...
- Decisiones abiertas no bloqueantes: ...
```

Si una decisión sigue abierta y no bloquea el orden del roadmap, anotala como pendiente explícitamente - no la escondas.

---

## Roadmap de specs

La sección central. Lista ordenada, no tabla larga con detalle de implementación - eso vive en cada spec.

```markdown
## Roadmap de specs

1. **slug-corto-uno**
   - Objetivo: Una oración.
   - Depende de: nada.
   - Tamaño: S.
   - Estado: Pendiente.
   - Spec: todavía no creada.

2. **slug-corto-dos**
   - Objetivo: Una oración.
   - Depende de: `slug-corto-uno`.
   - Tamaño: M.
   - Estado: Especificada.
   - Spec: `specs/01-slug-corto-dos.md`.
```

**Reglas:**

- Cada ítem tiene que caber en una sola spec de `/spec-init` (algo resoluble en horas, no días).
- El número representa el orden actual. El slug es la identidad estable y debe ser único.
- Las dependencias se expresan con slugs, nunca con números ordinales.
- Tamaño es relativo entre ítems de este mismo roadmap: S/M/L. Incluso L debe caber en una única spec y previsiblemente en no más de una jornada; si no, se divide.
- Estado usa `Pendiente`, `Especificada`, `En progreso`, `Hecho` o `Replantear`.
- `Pendiente`: todavía no hay spec. `Especificada`: la spec existe pero no está implementada. `En progreso`: hay evidencia de implementación activa. `Hecho`: la spec vinculada está en `Implemented` o equivalente. `Replantear`: la spec quedó obsoleta o su alcance debe redefinirse.
- `Spec` contiene la ruta real una vez creada; hasta entonces dice `todavía no creada`.
- El slug de cada ítem es una sugerencia de punto de partida para `/spec-init`, no un compromiso - puede ajustarse al escribir la spec real.

---

## Ideas para etapas futuras

Capacidades valiosas registradas pero todavía no comprometidas. No llevan número, tamaño ni estado de ejecución, para que no se confundan con el roadmap activo.

```markdown
## Ideas para etapas futuras

- **idea-slug** - Descripción breve de la capacidad.
  - Motivo para diferir: No es necesaria para validar la v1.
  - Reconsiderar cuando: Exista suficiente uso o evidencia que la justifique.
```

`Reconsiderar cuando` es opcional si no existe un disparador concreto. Si una idea se promueve al roadmap, eliminarla de esta sección. Omitir la sección si no surgieron ideas futuras.

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

## Historial de revisiones

Registro breve para que una revisión no borre silenciosamente la evolución del roadmap.

```markdown
## Historial de revisiones

| Fecha | Cambio |
| --- | --- |
| YYYY-MM-DD | Roadmap inicial. |
| YYYY-MM-DD | Se agregó `slug-nuevo` y se movió `billing` después de `organizations`. |
```

Registrar cambios de alcance u orden, no correcciones editoriales menores.

---

## Cómo seguir

Cierre fijo del documento:

```markdown
## Cómo seguir

Ejecutá `/spec-init` nombrando el slug exacto del próximo ítem pendiente (en orden, salvo que el usuario prefiera otro). Si el vínculo es inequívoco, `/spec-init` lo registra como `Especificada` y activa el roadmap. Cuando esa spec llegue a `Approved`, corré `/spec-impl`: al comenzar la marca `En progreso` y, después de verificar todos los criterios de aceptación, deja la spec en `Implemented` y el ítem en `Hecho`. Cuando todos los ítems comprometidos estén hechos, el roadmap pasa a `Complete`.
```

---

## Reglas globales sobre todo el documento

- **Sin plan de implementación paso a paso, sin modelo de datos, sin criterios de aceptación.** Eso es contenido de spec individual, no de roadmap.
- **Una oración por objetivo de ítem.** Si necesita más, es una señal de que el ítem es en realidad dos.
- **Slugs únicos y dependencias estables.** Reordenar ítems no debe romper referencias.
- **Sin duplicados entre roadmap e ideas futuras.** Una capacidad pertenece a una sola sección.
- **Markdown estándar.** Debe renderizar en GitHub sin sorpresas.
