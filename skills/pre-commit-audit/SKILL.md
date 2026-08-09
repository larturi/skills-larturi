---
name: pre-commit-audit
description: Auditoria de cambios staged antes de commitear, con metodologia 4R (Risk, Readability, Reliability, Resilience) mas chequeos de higiene de commit. Usar cuando se pida auditar, revisar o validar cambios antes de un commit
---

# Pre-Commit Audit

Actua como senior software engineer auditando los cambios a punto de commitearse.
Analiza con `git diff --staged` (si no hay nada staged, usa `git diff` y avisa que no hay cambios en stage).

La auditoria se organiza en **cuatro dimensiones independientes (4R)** mas un bloque de **higiene de commit**. Recorrelas una por una, sin mezclarlas.

## Alcance

- Audita SOLO el codigo tocado en el diff, mas bugs pre-existentes visibles en el codigo que se modifica.
- Lee el archivo completo afectado antes de opinar, no solo las lineas del diff.
- NO reportes issues especulativos o de baja confianza. Si no estas seguro, dilo como duda, no como hallazgo.

## Higiene de commit (bloqueo previo a las 4R)

Chequeos rapidos que bloquean el commit por si solos:

- Secretos o credenciales: tokens, API keys, passwords, archivos `.env` staged.
- Codigo de debug: `console.log`, `debugger`, `print()` olvidados.
- Codigo comentado sin justificacion o `TODO`/`FIXME` nuevos sin ticket.
- Archivos que no deberian commitearse: binarios grandes, `node_modules`, builds, archivos de IDE.
- Archivos staged por accidente (cambios ajenos al trabajo actual).
- Marcadores de conflicto de merge sin resolver (`<<<<<<<`, `=======`, `>>>>>>>`).

## Las 4 dimensiones

### R1 — Risk (Seguridad y riesgo)

Foco: seguridad, limites de privilegios, exposicion de datos y vulnerabilidades que bloquean el commit.

- Vulnerabilidades OWASP Top 10 (inyeccion, XSS, SSRF, etc.).
- Manejo de secretos: nada hardcodeado ni logueado.
- Validacion de inputs en bordes de confianza (server actions, endpoints, handlers).
- Exposicion de datos sensibles hacia el cliente.
- Dependencias nuevas: version pinneada, paquete conocido, sin typosquatting.

### R2 — Readability (Legibilidad y mantenibilidad)

Foco: naming, complejidad, intencion y convenciones del proyecto.

- Naming consistente con el proyecto (ej. `camelCase` variables/funciones, `PascalCase` componentes/tipos).
- Sin `any` nuevo sin justificar (si el repo usa TypeScript strict).
- Complejidad innecesaria, funciones muy largas, logica duplicada.
- Convenciones del proyecto: leer `AGENTS.md` si existe y aplicarlo como regla dura.
- Claridad de intencion: ¿se entiende QUE hace el codigo sin adivinar?
- Tamano del cambio: si el diff es muy grande, sugerir dividir en commits mas chicos.

### R3 — Reliability (Confiabilidad y comportamiento)

Foco: logica correcta, edge cases, contratos y regresiones.

- Errores de logica y comportamiento incorrecto.
- Edge cases no manejados y problemas de `null`/`undefined`.
- Contratos entre capas: formas de retorno consistentes.
- Regresiones sobre comportamiento existente.
- Determinismo: nada de dependencias ocultas de orden, tiempo o estado global.
- Si hay tests cercanos al cambio: ¿siguen validos? ¿Falta cubrir comportamiento nuevo o bugfix?

### R4 — Resilience (Resiliencia y operacion)

Foco: fallbacks, degradacion elegante, observabilidad y riesgos operativos.

- Integraciones externas: timeouts, reintentos, manejo de fallo.
- Degradacion elegante: si el servicio externo falla o tarda, ¿la app queda usable?
- Cache y revalidacion usados correctamente (si aplica al stack).
- Observabilidad: errores logueados con contexto util, sin filtrar secretos.

## Formato de salida

Reporta en espanol, agrupado por bloque (Higiene, luego R1 a R4). Para cada hallazgo:

- **Severidad**: `CRITICO` (bloquea commit) · `ADVERTENCIA` (deberia arreglarse) · `SUGERENCIA` (mejora opcional).
- **Ubicacion**: archivo y linea.
- **Que**: el problema concreto.
- **Por que**: el impacto o el riesgo.
- **Como**: la correccion propuesta (con ejemplo si aplica).

Cerra con un **veredicto**:

- `LISTO PARA COMMIT` — sin criticos ni advertencias de higiene.
- `COMMIT CON CAMBIOS MENORES` — solo advertencias/sugerencias.
- `NO COMMITEAR` — al menos un `CRITICO`.

Si un bloque no tiene hallazgos, dilo explicitamente (`R2 — Readability: sin observaciones`). No inventes hallazgos para llenar.

## Notas

- Si el veredicto es `NO COMMITEAR`, ofrece arreglar los hallazgos criticos antes de commitear.
- Si hay cambios unstaged mezclados con staged, avisa: el commit solo incluye lo staged.
- Esta skill audita, no commitea. El commit lo decide y lo ejecuta el usuario.
