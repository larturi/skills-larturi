---
name: spec-pre-commit
description: Auditoría de cambios staged antes de commitear, con metodología 4R (Risk, Readability, Reliability, Resilience) más chequeos de higiene de commit. Usar cuando se pida auditar, revisar o validar cambios antes de un commit. Es la única skill del grupo sdd auto-invocable a propósito, porque sirve también fuera del flujo.
---

# Pre-Commit Audit

Actuá como senior software engineer auditando los cambios a punto de commitearse.
Analizá `git diff --staged` (si no hay nada staged, usá `git diff` y avisá que no hay cambios en stage).

La auditoría se organiza en **cuatro dimensiones independientes (4R)** más un bloque de **higiene de commit**. Recorrelas una por una, sin mezclarlas.

## Alcance

- Auditá SOLO el código tocado en el diff, más bugs preexistentes visibles en el código que se modifica.
- Leé el archivo completo afectado antes de opinar, no solo las líneas del diff.
- Leé las instrucciones del repo que existan (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`) y aplicá sus convenciones como regla dura.
- NO reportes issues especulativos o de baja confianza. Si no estás seguro, decilo como duda, no como hallazgo.

## Higiene de commit (bloqueo previo a las 4R)

Chequeos rápidos que bloquean el commit por sí solos:

- Secretos o credenciales: tokens, API keys, passwords, archivos `.env` staged.
- Código de debug: `console.log`, `debugger`, `print()` olvidados.
- Código comentado sin justificación o `TODO`/`FIXME` nuevos sin ticket.
- Archivos que no deberían commitearse: binarios grandes, `node_modules`, builds, archivos de IDE.
- Archivos staged por accidente (cambios ajenos al trabajo actual).
- Marcadores de conflicto de merge sin resolver (`<<<<<<<`, `=======`, `>>>>>>>`).

## Las 4 dimensiones

### R1 - Risk (Seguridad y riesgo)

Foco: seguridad, límites de privilegios, exposición de datos y vulnerabilidades que bloquean el commit.

- Vulnerabilidades OWASP Top 10 (inyeccion, XSS, SSRF, etc.).
- Manejo de secretos: nada hardcodeado ni logueado.
- Validación de inputs en bordes de confianza (server actions, endpoints, handlers).
- Exposición de datos sensibles hacia el cliente.
- Dependencias nuevas: versión pinneada, paquete conocido, sin typosquatting.

### R2 - Readability (Legibilidad y mantenibilidad)

Foco: naming, complejidad, intención y convenciones del proyecto.

- Naming consistente con el proyecto (ej. `camelCase` variables/funciones, `PascalCase` componentes/tipos).
- Sin `any` nuevo sin justificar (si el repo usa TypeScript strict).
- Complejidad innecesaria, funciones muy largas, lógica duplicada.
- Claridad de intención: ¿se entiende QUÉ hace el código sin adivinar?
- Tamaño del cambio: si el diff es muy grande, sugerir dividir en commits más chicos.

### R3 - Reliability (Confiabilidad y comportamiento)

Foco: lógica correcta, edge cases, contratos y regresiones.

- Errores de lógica y comportamiento incorrecto.
- Edge cases no manejados y problemas de `null`/`undefined`.
- Contratos entre capas: formas de retorno consistentes.
- Regresiones sobre comportamiento existente.
- Determinismo: nada de dependencias ocultas de orden, tiempo o estado global.
- Si hay tests cercanos al cambio: ¿siguen válidos? ¿Falta cubrir comportamiento nuevo o bugfix?

### R4 - Resilience (Resiliencia y operacion)

Foco: fallbacks, degradación elegante, observabilidad y riesgos operativos.

- Integraciones externas: timeouts, reintentos, manejo de fallo.
- Degradación elegante: si el servicio externo falla o tarda, ¿la app queda usable?
- Cache y revalidación usados correctamente (si aplica al stack).
- Observabilidad: errores logueados con contexto útil, sin filtrar secretos.

## Formato de salida

Reportá en español, agrupado por bloque (Higiene, luego R1 a R4). Para cada hallazgo:

- **Severidad**: `CRITICO` (bloquea commit) · `ADVERTENCIA` (deberia arreglarse) · `SUGERENCIA` (mejora opcional).
- **Ubicación**: archivo y línea.
- **Que**: el problema concreto.
- **Por que**: el impacto o el riesgo.
- **Cómo**: la corrección propuesta (con ejemplo si aplica).

Cerrá con un **veredicto**:

- `LISTO PARA COMMIT` - sin críticos ni advertencias de higiene.
- `COMMIT CON CAMBIOS MENORES` - solo advertencias/sugerencias.
- `NO COMMITEAR` - al menos un `CRITICO`.

Si un bloque no tiene hallazgos, decilo explícitamente (`R2 - Readability: sin observaciones`). No inventes hallazgos para llenar.

## Notas

- Si el veredicto es `NO COMMITEAR`, ofrecé arreglar los hallazgos críticos antes de commitear (salvo que te corran como auditoría de solo lectura, ej. desde `/spec-finish`).
- Si hay cambios unstaged mezclados con staged, avisá: el commit solo incluye lo staged.
- Esta skill audita, no commitea. El commit lo decide y lo ejecuta el usuario.
- No toques specs ni roadmap: los estados los escriben `/spec-init`, `/spec-impl` y el usuario.
- Nunca uses el carácter de guion largo. Usá siempre `-`.
