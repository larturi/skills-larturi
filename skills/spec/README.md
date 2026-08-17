# /spec

Diseña el documento de una feature haciendo preguntas de clarificación, siguiendo el método spec-driven. No escribe código.

## Uso

```bash
/spec niveles-y-highscores
```

## Qué hace

1. **Contexto** — lee `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` / `README.md` (el primero que exista) y los specs previos en `specs/`.
2. **Preguntas** — clarifica alcance, datos, integración, persistencia y riesgos en bloques de 3-5 preguntas.
3. **Escritura** — genera el spec completo (o sección por sección, si falta información).
4. **Guardado** — escribe `specs/NN-slug.md` con estado `Draft`.

## Salida

Un spec con: objetivo en una frase, alcance (qué entra / qué no), modelo de datos, plan de implementación paso a paso, criterios de aceptación verificables, y decisiones tomadas/descartadas.

El estado arranca en `Draft`. Cambiarlo a `Approved` es un paso manual del humano — de ahí lo toma [`/spec-impl`](../spec-impl/).
