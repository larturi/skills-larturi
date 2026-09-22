# /spec-plan

Planifica un proyecto o sistema desde cero: visión, alcance de la v1, arquitectura de alto nivel, y una lista ordenada de specs candidatas. No escribe specs individuales ni código.

## Uso

```bash
/spec-plan sistema de reservas para un gimnasio, multi-sede
```

## Qué hace

1. **Contexto** - lee `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` / `README.md` y detecta si ya existe `specs/00-roadmap.md` (revisión) o si es la primera vez (planificación desde cero).
2. **Preguntas** - clarifica visión, alcance de la v1, restricciones técnicas, integraciones externas, orden de valor, riesgos e ideas para etapas futuras.
3. **Descomposición** - parte el proyecto en ítems del tamaño de una spec (resolubles en horas, no días), con slugs estables, dependencias y orden explícitos.
4. **Guardado** - escribe o revisa `specs/00-roadmap.md`, vincula las specs existentes y conserva un historial breve de cambios.

## Cuándo usarlo

- Sistema nuevo desde cero, donde el trabajo no entra en una sola spec.
- Para una feature puntual sobre un proyecto existente, saltealo y andá directo a [`/spec-init`](../spec-init/).

## Salida

Un roadmap con: visión en una o dos oraciones, alcance in/out de la v1, arquitectura de alto nivel, lista ordenada de specs candidatas, ideas no comprometidas para etapas futuras e historial de revisiones. Cada ítem comprometido incluye objetivo, dependencias por slug, tamaño relativo, estado y vínculo a su spec cuando exista.

Cada ítem del roadmap se convierte después en su propia spec vía [`/spec-init`](../spec-init/), se implementa con [`/spec-impl`](../spec-impl/). Los estados del ítem los actualizan esas skills: `/spec-init` lo pasa a `Especificada` y `/spec-impl` a `En progreso` y luego a `Hecho`.
