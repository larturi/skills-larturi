# Grupo `sdd`

Flujo de spec-driven development: planificar (proyectos grandes), diseñar una spec, implementarla paso a paso, y auditar los cambios antes de commitear.

Las cinco skills están agrupadas porque cubren fases consecutivas de un mismo flujo y comparten la carpeta `specs/` como fuente de verdad.

## Skills del grupo

| Skill | Rol |
|-------|-----|
| `spec-plan` | **Opcional, para proyectos nuevos desde cero.** Planificación de alto nivel: visión, alcance y arquitectura, descompuestos en una lista ordenada de specs candidatas. Guarda `specs/00-roadmap.md`. No escribe specs individuales ni código. |
| `spec-init` | **Punto de entrada habitual.** Diseña una spec guiado por preguntas de clarificación, sección por sección, y la guarda en `specs/NN-slug.md` con estado `Draft`. |
| `spec-impl` | Implementa una spec en estado `Approved`: corre todos los pasos del plan seguidos por defecto (con una confirmación explícita antes de arrancar), y cierra con una verificación final obligatoria contra los criterios de aceptación. Por defecto trabaja directo en `main` (`AutoCreateBranch: false`); opcionalmente crea una rama propia para la spec. |
| `spec-pre-commit` | Auditoria 4R (Risk, Readability, Reliability, Resilience) de los cambios staged antes de commitear, con veredicto de bloqueo. `spec-finish` la invoca automáticamente como gate al cerrar una spec; también se puede usar sola, sobre cualquier commit. |
| `spec-finish` | Cierra una spec en estado `Implemented` o `Implementado con observaciones`: audita los cambios (con rama o sin rama, según cómo corrió `spec-impl`) con la misma auditoria de `spec-pre-commit` como gate obligatorio, y si pasa, prepara el cierre con un mensaje de commit que resume la spec. Nunca ejecuta el commit - eso lo hace siempre el usuario. |

## Flujo típico

**Feature puntual sobre un proyecto existente** (se resuelve en una spec, horas de trabajo):

```bash
/spec-init niveles-y-highscores   # 1. Diseñar la spec (queda en Draft)
#  ... revisar specs/NN-slug.md y cambiar el estado a Approved ...
/spec-impl NN-slug                # 2. Implementar todos los pasos, con verificación final
/spec-finish NN-slug              # 3. Auditar (invoca spec-pre-commit como gate) y preparar el cierre
#  ... revisar el diff staged y commitear a mano ...
```

**Sistema nuevo desde cero** (no entra en una sola spec):

```bash
/spec-plan sistema de reservas para un gimnasio   # 0. Planificar (queda specs/00-roadmap.md)
#  ... por cada ítem del roadmap, en orden, repetir el loop de arriba ...
/spec-init <primer ítem del roadmap>
/spec-impl NN-slug
/spec-finish NN-slug
#  ... spec-impl sincroniza el ítem vinculado; pasar al siguiente ...
```

`spec-impl` requiere una spec generada por `spec-init` con estado `Approved`. `spec-finish` requiere que `spec-impl` haya dejado la spec en `Implemented` o `Implementado con observaciones`. `spec-pre-commit` sigue siendo independiente y puede usarse sola sobre cualquier commit, dentro o fuera de este flujo. `spec-plan` es opcional y solo aporta valor cuando el trabajo no entra en una sola spec.

Cuando hay un roadmap activo y un vínculo inequívoco, `spec-init` y `spec-impl` sincronizan el archivo de spec y el estado del ítem. Sin roadmap, con un roadmap ya completo o para una feature puntual que no pertenece al plan, ambas skills funcionan de forma independiente y no alteran el roadmap histórico.

El estado `Released` de una spec es siempre una decisión manual del usuario - ninguna skill del grupo lo escribe. `spec-impl` llega hasta `Implemented` o `Implementado con observaciones`, y `spec-finish` prepara el commit de cierre pero tampoco toca ese campo.
