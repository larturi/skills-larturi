# Grupo `sdd`

Flujo de spec-driven development: diseñar una spec, implementarla paso a paso, y auditar los cambios antes de commitear.

Las tres skills están agrupadas porque cubren fases consecutivas de un mismo flujo y comparten la carpeta `specs/` como fuente de verdad.

## Skills del grupo

| Skill | Rol |
|-------|-----|
| `spec-init` | **Punto de entrada.** Diseña una spec guiado por preguntas de clarificación, sección por sección, y la guarda en `specs/NN-slug.md` con estado `Draft`. |
| `spec-impl` | Implementa una spec en estado `Approved`, paso a paso, con pausas para revisar cada diff. Opcionalmente crea la rama de git de la spec. |
| `spec-pre-commit` | Auditoria 4R (Risk, Readability, Reliability, Resilience) de los cambios staged antes de commitear, con veredicto de bloqueo. |

## Flujo típico

```bash
/spec-init niveles-y-highscores   # 1. Diseñar la spec (queda en Draft)
#  ... revisar specs/NN-slug.md y cambiar el estado a Approved ...
/spec-impl NN-slug                # 2. Implementar paso a paso
#  ... antes de cada commit ...
/spec-pre-commit                  # 3. Auditar los cambios staged
```

`spec-impl` requiere una spec generada por `spec-init` con estado `Approved`. `spec-pre-commit` es independiente y puede usarse en cualquier commit, spec o no.
