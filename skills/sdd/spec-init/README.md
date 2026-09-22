# /spec-init

Diseña el documento de una feature haciendo preguntas de clarificación, siguiendo el método spec-driven. No escribe código.

## Uso

```bash
/spec-init niveles-y-highscores
```

## Qué hace

1. **Contexto** - lee `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` / `README.md`, las specs previas y, si existe, el roadmap activo.
2. **Preguntas** - clarifica alcance, datos, integración, persistencia y riesgos en bloques de 3-5 preguntas.
3. **Escritura** - genera una spec breve (una línea por idea), completa o sección por sección si falta información.
4. **Guardado** - escribe `specs/NN-slug.md` con estado `Borrador` y, si corresponde inequívocamente a un ítem del roadmap activo, lo vincula como `Especificada`.

## Salida

Una spec corta (orientativo: 60-80 líneas) con: objetivo en una oración, alcance (entra / fuera), modelo de datos si aplica, plan de una línea por paso, criterios de aceptación booleanos y decisiones con su razón.

El estado arranca en `Borrador`. Cambiarlo a `Aprobado` es un paso manual - de ahí lo toma [`/spec-impl`](../spec-impl/).

El roadmap es opcional. Si no existe, está completo o la feature no coincide inequívocamente con un ítem comprometido, la spec se crea de forma independiente y el roadmap no se modifica.

¿Planificando un sistema desde cero y no una feature puntual? Empezá por [`/spec-plan`](../spec-plan/) para armar el roadmap y después dosificá cada ítem con este comando.
