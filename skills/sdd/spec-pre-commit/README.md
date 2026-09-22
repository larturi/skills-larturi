# /spec-pre-commit

Audita los cambios staged antes de commitear y da un veredicto de bloqueo. No commitea ni modifica código.

## Uso

```bash
/spec-pre-commit
```

Audita `git diff --staged`. Si no hay nada staged, audita `git diff` y lo avisa.

## Qué hace

1. **Higiene de commit:** secretos, código de debug, TODOs sin ticket, archivos que no deberían commitearse, marcadores de conflicto.
2. **4R**, una dimensión por vez:
   - **Risk:** seguridad, inputs, exposición de datos, dependencias nuevas.
   - **Readability:** naming, complejidad, duplicación, convenciones del repo (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`).
   - **Reliability:** lógica, edge cases, contratos, regresiones, tests.
   - **Resilience:** timeouts, degradación, cache, observabilidad.
3. **Veredicto:** `LISTO PARA COMMIT`, `COMMIT CON CAMBIOS MENORES` o `NO COMMITEAR`.

Cada hallazgo trae severidad (`CRITICO` / `ADVERTENCIA` / `SUGERENCIA`), ubicación, qué, por qué y cómo corregirlo.

## Dentro y fuera del flujo

- **Dentro del flujo:** [`/spec-finish`](../spec-finish/) aplica esta misma metodología como gate. No hace falta correrla aparte.
- **Fuera del flujo:** se puede usar sola sobre cualquier commit. Es la única skill del grupo que el agente puede invocar por su cuenta.

## Reglas clave

- Solo audita lo tocado en el diff (más bugs preexistentes visibles ahí).
- No reporta hallazgos especulativos: las dudas se marcan como dudas.
- No toca specs ni roadmap.
