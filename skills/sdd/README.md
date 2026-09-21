# Grupo `sdd`

Flujo de spec-driven development: planificar (proyectos grandes), diseñar una spec, implementarla paso a paso, y auditar los cambios antes de commitear.

Las cuatro skills están agrupadas porque cubren fases consecutivas de un mismo flujo y comparten la carpeta `specs/` como fuente de verdad.

## Skills del grupo

| Skill | Rol |
|-------|-----|
| `spec-plan` | **Opcional, para proyectos nuevos desde cero.** Planificación de alto nivel: visión, alcance y arquitectura, descompuestos en una lista ordenada de specs candidatas. Guarda `specs/00-roadmap.md`. No escribe specs individuales ni código. |
| `spec-init` | **Punto de entrada habitual.** Diseña una spec guiado por preguntas de clarificación, sección por sección, y la guarda en `specs/NN-slug.md` con estado `Draft`. |
| `spec-impl` | Implementa una spec en estado `Approved`, paso a paso, con pausas para revisar cada diff. Opcionalmente crea la rama de git de la spec. |
| `spec-pre-commit` | Auditoria 4R (Risk, Readability, Reliability, Resilience) de los cambios staged antes de commitear, con veredicto de bloqueo. |

## Flujo típico

**Feature puntual sobre un proyecto existente** (se resuelve en una spec, horas de trabajo):

```bash
/spec-init niveles-y-highscores   # 1. Diseñar la spec (queda en Draft)
#  ... revisar specs/NN-slug.md y cambiar el estado a Approved ...
/spec-impl NN-slug                # 2. Implementar paso a paso
#  ... antes de cada commit ...
/spec-pre-commit                  # 3. Auditar los cambios staged
```

**Sistema nuevo desde cero** (no entra en una sola spec):

```bash
/spec-plan sistema de reservas para un gimnasio   # 0. Planificar (queda specs/00-roadmap.md)
#  ... por cada ítem del roadmap, en orden, repetir el loop de arriba ...
/spec-init <primer ítem del roadmap>
/spec-impl NN-slug
/spec-pre-commit
#  ... spec-impl sincroniza el ítem vinculado; pasar al siguiente ...
```

`spec-impl` requiere una spec generada por `spec-init` con estado `Approved`. `spec-pre-commit` es independiente y puede usarse en cualquier commit, spec o no. `spec-plan` es opcional y solo aporta valor cuando el trabajo no entra en una sola spec.

Cuando hay un roadmap activo y un vínculo inequívoco, `spec-init` y `spec-impl` sincronizan el archivo de spec y el estado del ítem. Sin roadmap, con un roadmap ya completo o para una feature puntual que no pertenece al plan, ambas skills funcionan de forma independiente y no alteran el roadmap histórico.
