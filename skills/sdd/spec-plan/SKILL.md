---
name: spec-plan
description: "Planifica un proyecto o sistema grande desde cero, descomponiéndolo en una hoja de ruta de specs secuenciadas. Un nivel arriba de spec-init: en vez de una spec lista para implementar, produce visión, alcance y arquitectura de alto nivel más la lista ordenada de specs que después se van implementando de a una con spec-init + spec-impl. Úsalo al arrancar un sistema nuevo desde cero, no para features puntuales en un proyecto existente."
disable-model-invocation: true
argument-hint: 'descripción breve del proyecto o sistema a construir'
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion, Bash(ls:*), Bash(cat:*), Bash(date:*), Bash(mkdir:*), mcp__plugin_engram_engram__mem_current_project, mcp__plugin_engram_engram__mem_search, mcp__plugin_engram_engram__mem_save, mcp__plugin_engram_engram__mem_session_summary
---

# /spec-plan - Planificador de alto nivel para proyectos nuevos

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

### Fase 0 - Detectar si hay memoria persistente (Engram)

Antes de entender el contexto, fijate si en esta sesión tenés disponible el protocolo de Engram (herramientas `mem_search`, `mem_save`, `mem_session_summary` - se anuncian como "core tools" al arrancar la sesión cuando el plugin está activo).

- **Si Engram está disponible:** vas a usarlo en la Fase 1 para traer decisiones de arquitectura o restricciones de sesiones anteriores relacionadas con este proyecto, y para guardar en la Fase 3 las decisiones de arquitectura que valga la pena recordar más allá de lo que ya queda escrito en el roadmap.
- **Si Engram NO está disponible:** avisá una sola vez, sin bloquear, y seguí:

  ```
  ℹ️ Engram no está disponible en esta sesión: las decisiones van a quedar
  solo en specs/00-roadmap.md, sin memoria persistente.
  ```

  No lo vuelvas a mencionar en el resto de la ejecución.

### Fase 1 - Entender el contexto

1. Leé todos los archivos de instrucciones aplicables que existan (`AGENTS.md`, `CLAUDE.md`, `GEMINI.md`), respetando su alcance y precedencia. Después leé `README.md`, si existe, como contexto del producto - no como reemplazo de las instrucciones.
2. Revisá el contexto de sesión de arriba. Si ya existe `specs/00-roadmap.md`, esto es una **revisión** de un roadmap existente, no una planificación desde cero - leelo completo antes de preguntar nada, y en la Fase 2 enfocate en qué cambió respecto de lo que ya está escrito.
3. Si además de (o en vez de) un roadmap ya hay specs numeradas (`01-`, `02-`, ...), vinculalas con los ítems por la referencia explícita del campo `Spec` o, como respaldo, por un slug inequívoco. Leé el estado dentro de cada spec: la existencia del archivo no significa que ya esté implementada.
4. **Si Engram está disponible** (ver Fase 0): llamá `mem_search` con palabras clave del proyecto o sistema a construir para ver si hay decisiones de arquitectura o restricciones de sesiones anteriores relevantes. Si aparece algo, traelo a la Fase 2 en vez de volver a preguntarlo.

Si `$ARGUMENTS` llega vacío, pedile al usuario una descripción inicial en **una o dos oraciones** de qué sistema quiere construir.

### Fase 2 - Clarificar mediante preguntas de alto nivel

Esta fase clarifica el **proyecto entero**, no una feature. Preguntá solo lo necesario, en bloques de 1 a 5 preguntas relacionadas, esperando respuesta antes de seguir. No completes un cupo fijo si el contexto ya alcanza.

**Categorías de preguntas que siempre deberías considerar:**

- **Visión y usuarios:** ¿Qué problema resuelve el sistema y para quién? ¿Cuál es el caso de uso principal que tiene que andar primero?
- **Alcance de la v1:** ¿Qué es indispensable para la primera versión funcional? ¿Qué queda explícitamente para después?
- **Restricciones técnicas:** ¿Hay stack, infra o integraciones ya decididas? ¿Alguna restricción no negociable (proveedor, lenguaje, hosting)?
- **Integraciones externas:** ¿El sistema depende de APIs, servicios o datos de terceros? ¿Alguno de esos es un riesgo (rate limits, costos, disponibilidad)?
- **Orden de valor:** ¿Qué parte del sistema, si funcionara sola, ya sería útil? Esa suele ser la primera spec.
- **Riesgos de arquitectura:** ¿Qué decisión, si se toma mal ahora, es cara de revertir después (modelo de datos, elección de proveedor, límites de escala)?

**Cómo formular las preguntas:**

- Concretas, no abiertas. Si tu agente expone `AskUserQuestion`, usala para estos bloques en vez de prosa - con tu recomendación marcada primero.
- Cuando algo suene a una feature completa dentro de la respuesta (ej.: "y también necesito un dashboard de analytics"), anotalo como candidato a ítem propio del roadmap, no como detalle de otro ítem.
- Clasificá cada capacidad mencionada fuera de la v1 como una de estas tres cosas: **exclusión explícita**, **idea para una etapa futura** o **spec comprometida del roadmap**. No mezcles ideas valiosas todavía no comprometidas con cosas que se decidió no construir.

**Cuándo dejar de preguntar:**

Parar cuando puedas responder sin asumir nada:

1. ¿Cuál es el primer ítem del roadmap y por qué va primero?
2. ¿Qué decisiones de arquitectura hacen falta antes de escribir la primera spec?
3. ¿Qué queda explícitamente fuera de la v1?

### Fase 3 - Diseñar la arquitectura de alto nivel y descomponer en specs

**Arquitectura de alto nivel:** un párrafo o lista corta con los componentes principales y las decisiones estructurales que ya se tomaron en la Fase 2 (stack, servicios, almacenamiento). Suficiente para justificar el orden del roadmap - no para implementar. Si una decisión de arquitectura todavía está abierta y bloquea el orden del roadmap, resolvela con una pregunta antes de seguir; no la dejes como TODO.

**Si Engram está disponible** (ver Fase 0): guardá con `mem_save` las decisiones de arquitectura no obvias que se cierren acá (una elección de stack o proveedor, un trade-off descartado y por qué, una restricción no negociable que impuso el usuario). No dupliques ahí todo el roadmap - el archivo `specs/00-roadmap.md` ya es la fuente de verdad; guardá solo lo que le ahorre repreguntar a una sesión futura.

**Descomposición en specs:** convertí el trabajo en una lista ordenada de ítems. Cada ítem es un candidato a spec futura, no la spec en sí.

Reglas de descomposición:

- **Tamaño de spec, no de epic.** Cada ítem tiene que poder resolverse en una sola spec de `/spec-init` - el usuario los describió como "unas horas" de trabajo. Si un ítem se siente más grande que eso, dividilo en dos o más ítems ya en esta lista, no lo dejes grande para que `/spec-init` lo descubra después.
- **Resultado verificable.** Preferí verticales delgadas y demostrables (una funcionalidad de punta a punta) por sobre capas horizontales. Permití una spec habilitante de infraestructura, migración o arquitectura solo cuando sea inevitable, produzca un resultado verificable y diga qué ítems desbloquea.
- **Identidad estable.** Cada ítem tiene un slug único. El número expresa el orden actual; las dependencias siempre se escriben con slugs, nunca con números ordinales, porque el roadmap puede reordenarse.
- **Dependencias explícitas.** Si `billing` necesita lo que construye `organizations`, decilo como `Depende de: organizations`. El orden final del roadmap debe respetar esas dependencias.
- **El primer ítem es el de mayor valor con menor dependencia.** Preferí arrancar por algo que, una vez implementado, ya sea demostrable.
- **Tamaño relativo, con límite.** Usá S/M/L solo para comparar ítems de este roadmap. Incluso un ítem L debe caber en una única spec y, previsiblemente, en no más de una jornada; si no, dividilo.
- **Estado trazable.** Cada ítem incluye `Estado` y `Spec`. Usá `Pendiente` si la spec todavía no existe, `Especificada` si existe pero no está implementada, `En progreso` solo cuando haya evidencia de implementación activa y `Hecho` únicamente cuando la spec vinculada esté en `Implementado` o `Publicado`. `Replantear` sirve para una spec vinculada que quedó `Obsoleto`.

**Ideas para etapas futuras:** registrá aparte las ideas valiosas que no estén comprometidas en el roadmap actual. No llevan número, tamaño ni estado de ejecución. Cada una debe indicar por qué se difiere y, cuando sea útil, qué condición justificaría reconsiderarla. Si en una revisión una idea se promueve al roadmap, sacala de esta sección.

### Fase 4 - Guardar el roadmap

1. Asegurate de que exista la carpeta `specs/`. Si es la primera vez, creá `specs/00-roadmap.md` siguiendo `template.md`.
2. Si ya existía, actualizalo sin perder trazabilidad: conservá `Creado`, actualizá `Actualizado`, preservá el estado y el vínculo `Spec` de los ítems que no cambiaron, y agregá una entrada breve al historial de revisiones indicando qué se agregó, eliminó o reordenó. No dejes ítems eliminados dentro del roadmap activo solo para conservar historia: el historial cumple esa función.
3. Usá la fecha del contexto de sesión de arriba. **Nunca escribas una fecha que no hayas leído de ahí.**
4. Escribí el archivo directamente. **No pidas permiso para escribirlo** - anunciá la ruta en la confirmación final.
5. Releé el archivo escrito y validá antes de confirmar:
   - todos los slugs son únicos;
   - todas las dependencias apuntan a slugs existentes y no forman ciclos;
   - el orden respeta las dependencias;
   - cada ítem tiene objetivo, dependencias, tamaño, estado y vínculo `Spec`;
   - el primer ítem pendiente no depende de otro ítem pendiente posterior;
   - ninguna idea futura está duplicada en el roadmap comprometido;
   - el Markdown está completo y no quedaron placeholders accidentales.
6. **Si Engram está disponible** (ver Fase 0): antes de confirmar, llamá `mem_session_summary` con Goal (el roadmap creado/revisado), Discoveries (lo guardado con `mem_save` en la Fase 3), Accomplished (roadmap escrito) y Relevant Files (`specs/00-roadmap.md`).
7. Confirmale al usuario:
   - Ruta del archivo (`specs/00-roadmap.md`).
   - Cuántos ítems tiene el roadmap y cuál es el primero.
   - Cuántas ideas quedaron registradas para etapas futuras, si las hay.
   - Próximo paso: ejecutar `/spec-init` con la descripción del primer ítem pendiente para empezar a dosificar el roadmap.
   - **Parar acá.** No propongas escribir la primera spec vos mismo, ni tomar ninguna acción más allá de esta confirmación.

## Reglas duras

- **Nunca escribir código ni specs individuales durante este comando.** Solo `specs/00-roadmap.md`.
- **Nunca asumir decisiones de arquitectura que el usuario no confirmó.** Si el orden del roadmap depende de una decisión abierta, preguntá en la Fase 2.
- **Cada ítem del roadmap debe caber en una spec de `/spec-init`.** Si dudás si un ítem es demasiado grande, dividilo - es más barato dividir acá que a mitad de una spec.
- **No dupliques el detalle de `/spec-init`.** El roadmap no lleva modelo de datos, plan de implementación paso a paso, ni criterios de aceptación - eso lo escribe cada spec individual cuando le toque.
- **No uses números ordinales como identidad.** Los números pueden cambiar; los slugs y vínculos `Spec` mantienen la trazabilidad.
- **No marques un ítem como `Hecho` por encontrar un archivo con nombre parecido.** Verificá el estado de la spec vinculada.
- **Puntuación:** nunca uses el carácter de guion largo. Usá siempre `-`.

## Argumentos

`$ARGUMENTS` es la descripción del proyecto o sistema completo, no de una feature puntual. Tratalo como punto de partida de la Fase 1.

Si invocaron `/spec-plan` sin argumentos, empezá pidiendo la descripción en una o dos oraciones.
