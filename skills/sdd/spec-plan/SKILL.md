---
name: spec-plan
description: "Planifica un proyecto o sistema grande desde cero, descomponiéndolo en una hoja de ruta de specs secuenciadas. Un nivel arriba de spec-init: en vez de una spec lista para implementar, produce visión, alcance y arquitectura de alto nivel más la lista ordenada de specs que después se van implementando de a una con spec-init + spec-impl. Úsalo al arrancar un sistema nuevo desde cero, no para features puntuales en un proyecto existente."
disable-model-invocation: true
argument-hint: 'descripción breve del proyecto o sistema a construir'
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion, Bash(ls:*), Bash(cat:*), Bash(date:*), mcp__plugin_engram_engram__mem_current_project, mcp__plugin_engram_engram__mem_search, mcp__plugin_engram_engram__mem_save, mcp__plugin_engram_engram__mem_session_summary
---

# /spec-plan — Planificador de alto nivel para proyectos nuevos

## Contexto de sesión

Fecha de hoy (usar esta para el encabezado del roadmap, nunca adivinarla):
!`date +%F`

Specs y roadmap que ya existen:
!`ls specs/ 2>/dev/null || echo "La carpeta specs/ todavía no existe"`

---

## Cuándo usar esto (y cuándo no)

- **Usá `/spec-plan`** cuando vas a construir un sistema de cero: hay decisiones de arquitectura por tomar, el trabajo no entra en una sola spec, y necesitás ver el mapa completo antes de arrancar a dosificarlo.
- **Usá `/spec-init` directo** (sin pasar por acá) cuando la feature es puntual sobre un proyecto que ya existe y se resuelve en una spec, típicamente unas horas de trabajo.
- **Este comando nunca escribe specs individuales ni código.** Su única salida es el archivo `specs/00-roadmap.md`. Las specs de cada ítem del roadmap las genera `/spec-init` cuando el usuario decide arrancar con ese ítem.

## Filosofía

Un roadmap no es una spec gigante. Es el mapa que permite decidir **en qué orden** y **con qué alcance** se va a dosificar el trabajo en specs individuales. Por eso acá la profundidad es la inversa de `/spec-init`: **poca profundidad en cada ítem, pero cobertura completa del proyecto.** El detalle de implementación de cada ítem se decide después, cuando ese ítem se convierta en su propia spec.

Leé `template.md` (en el mismo directorio que este skill) para ver la estructura completa que va a seguir el roadmap. Apoyate en ella en cada paso.

## Flujo del comando

- Seguí las cinco fases en orden.
- Tus respuestas deben estar en el mismo idioma que el prompt inicial.

### Fase 0 — Detectar si hay memoria persistente (Engram)

Antes de entender el contexto, fijate si en esta sesión tenés disponible el protocolo de Engram (herramientas `mem_search`, `mem_save`, `mem_session_summary` — se anuncian como "core tools" al arrancar la sesión cuando el plugin está activo).

- **Si Engram está disponible:** vas a usarlo en la Fase 1 para traer decisiones de arquitectura o restricciones de sesiones anteriores relacionadas con este proyecto, y para guardar en la Fase 3 las decisiones de arquitectura que valga la pena recordar más allá de lo que ya queda escrito en el roadmap.
- **Si Engram NO está disponible:** avisale al usuario en una sola línea, bloquear el flujo una unica vez y proponerle ayuda para configurarlo:

  ```
  ℹ️ No tenés Engram configurado en esta sesión, queres configurarlo? 
  De lo contrario las decisiones de arquitectura de este roadmap van a quedar 
  solo en specs/00-roadmap.md, sin memoria persistente entre sesiones. 
  Si querés que las próximas specs de este proyecto arranquen con ese contexto, 
  activamos el plugin engram ahora.
  ```

  No lo vuelvas a mencionar en el resto de la ejecución.

### Fase 1 — Entender el contexto

1. Leé el archivo de memoria del proyecto, si existe. Probá en orden y detenete en el primero que encuentres: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `README.md`.
2. Revisá el contexto de sesión de arriba. Si ya existe `specs/00-roadmap.md`, esto es una **revisión** de un roadmap existente, no una planificación desde cero — leelo completo antes de preguntar nada, y en la Fase 2 enfocate en qué cambió respecto de lo que ya está escrito.
3. Si además de (o en vez de) un roadmap ya hay specs numeradas (`01-`, `02-`, ...), notá cuáles ítems del roadmap ya se implementaron para no volver a plantearlos como pendientes.
4. **Si Engram está disponible** (ver Fase 0): llamá `mem_search` con palabras clave del proyecto o sistema a construir para ver si hay decisiones de arquitectura o restricciones de sesiones anteriores relevantes. Si aparece algo, traelo a la Fase 2 en vez de volver a preguntarlo.

Si `$ARGUMENTS` llega vacío, pedile al usuario una descripción inicial en **una o dos oraciones** de qué sistema quiere construir.

### Fase 2 — Clarificar mediante preguntas de alto nivel

Esta fase clarifica el **proyecto entero**, no una feature. Preguntá en bloques de 3 a 5, esperando respuesta antes de seguir.

**Categorías de preguntas que siempre deberías considerar:**

- **Visión y usuarios:** ¿Qué problema resuelve el sistema y para quién? ¿Cuál es el caso de uso principal que tiene que andar primero?
- **Alcance de la v1:** ¿Qué es indispensable para la primera versión funcional? ¿Qué queda explícitamente para después?
- **Restricciones técnicas:** ¿Hay stack, infra o integraciones ya decididas? ¿Alguna restricción no negociable (proveedor, lenguaje, hosting)?
- **Integraciones externas:** ¿El sistema depende de APIs, servicios o datos de terceros? ¿Alguno de esos es un riesgo (rate limits, costos, disponibilidad)?
- **Orden de valor:** ¿Qué parte del sistema, si funcionara sola, ya sería útil? Esa suele ser la primera spec.
- **Riesgos de arquitectura:** ¿Qué decisión, si se toma mal ahora, es cara de revertir después (modelo de datos, elección de proveedor, límites de escala)?

**Cómo formular las preguntas:**

- Concretas, no abiertas. Si tu agente expone `AskUserQuestion`, usala para estos bloques en vez de prosa — con tu recomendación marcada primero.
- Cuando algo suene a una feature completa dentro de la respuesta (ej.: "y también necesito un dashboard de analytics"), anotalo como candidato a ítem propio del roadmap, no como detalle de otro ítem.

**Cuándo dejar de preguntar:**

Parar cuando puedas responder sin asumir nada:

1. ¿Cuál es el primer ítem del roadmap y por qué va primero?
2. ¿Qué decisiones de arquitectura hacen falta antes de escribir la primera spec?
3. ¿Qué queda explícitamente fuera de la v1?

### Fase 3 — Diseñar la arquitectura de alto nivel y descomponer en specs

**Arquitectura de alto nivel:** un párrafo o lista corta con los componentes principales y las decisiones estructurales que ya se tomaron en la Fase 2 (stack, servicios, almacenamiento). Suficiente para justificar el orden del roadmap — no para implementar. Si una decisión de arquitectura todavía está abierta y bloquea el orden del roadmap, resolvela con una pregunta antes de seguir; no la dejes como TODO.

**Si Engram está disponible** (ver Fase 0): guardá con `mem_save` las decisiones de arquitectura no obvias que se cierren acá (una elección de stack o proveedor, un trade-off descartado y por qué, una restricción no negociable que impuso el usuario). No dupliques ahí todo el roadmap — el archivo `specs/00-roadmap.md` ya es la fuente de verdad; guardá solo lo que le ahorre repreguntar a una sesión futura.

**Descomposición en specs:** convertí el trabajo en una lista ordenada de ítems. Cada ítem es un candidato a spec futura, no la spec en sí.

Reglas de descomposición:

- **Tamaño de spec, no de epic.** Cada ítem tiene que poder resolverse en una sola spec de `/spec-init` — el usuario los describió como "unas horas" de trabajo. Si un ítem se siente más grande que eso, dividilo en dos o más ítems ya en esta lista, no lo dejes grande para que `/spec-init` lo descubra después.
- **Cada ítem deja el sistema funcional.** Igual que los pasos de una spec: nada de "medio backend sin frontend que lo use". Preferí verticales delgadas (una funcionalidad de punta a punta) por sobre capas horizontales (todo el modelo de datos primero, después todo el backend, después todo el frontend), salvo que el usuario pida explícitamente lo segundo.
- **Dependencias explícitas.** Si el ítem 4 necesita lo que construye el ítem 2, decilo. El orden final del roadmap debe respetar esas dependencias.
- **El primer ítem es el de mayor valor con menor dependencia.** Preferí arrancar por algo que, una vez implementado, ya sea demostrable.

## Fase 4 — Guardar el roadmap

1. Si es la primera vez, creá `specs/00-roadmap.md` siguiendo `template.md`. Si ya existía (revisión), actualizalo: conservá el estado (`Pendiente` / `En progreso` / `Hecho`) de los ítems que no cambiaron, y marcá con claridad qué se agregó, se sacó o se reordenó.
2. Usá la fecha del contexto de sesión de arriba. **Nunca escribas una fecha que no hayas leído de ahí.**
3. Escribí el archivo directamente. **No pidas permiso para escribirlo** — anunciá la ruta en la confirmación final.
4. **Si Engram está disponible** (ver Fase 0): antes de confirmar, llamá `mem_session_summary` con Goal (el roadmap creado/revisado), Discoveries (lo guardado con `mem_save` en la Fase 3), Accomplished (roadmap escrito) y Relevant Files (`specs/00-roadmap.md`).
5. Confirmale al usuario:
   - Ruta del archivo (`specs/00-roadmap.md`).
   - Cuántos ítems tiene el roadmap y cuál es el primero.
   - Próximo paso: ejecutar `/spec-init` con la descripción del primer ítem pendiente para empezar a dosificar el roadmap.
   - **Parar acá.** No propongas escribir la primera spec vos mismo, ni tomar ninguna acción más allá de esta confirmación.

## Reglas duras

- **Nunca escribir código ni specs individuales durante este comando.** Solo `specs/00-roadmap.md`.
- **Nunca asumir decisiones de arquitectura que el usuario no confirmó.** Si el orden del roadmap depende de una decisión abierta, preguntá en la Fase 2.
- **Cada ítem del roadmap debe caber en una spec de `/spec-init`.** Si dudás si un ítem es demasiado grande, dividilo — es más barato dividir acá que a mitad de una spec.
- **No dupliques el detalle de `/spec-init`.** El roadmap no lleva modelo de datos, plan de implementación paso a paso, ni criterios de aceptación — eso lo escribe cada spec individual cuando le toque.

## Argumentos

`$ARGUMENTS` es la descripción del proyecto o sistema completo, no de una feature puntual. Tratalo como punto de partida de la Fase 1.

Si invocaron `/spec-plan` sin argumentos, empezá pidiendo la descripción en una o dos oraciones.
