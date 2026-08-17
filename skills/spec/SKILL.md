---
name: spec
description: Diseña y desarrolla specs siguiendo el método spec-driven. Hace preguntas de aclaración antes de proponer una estructura, y construye la spec sección por sección. Úsalo al comenzar una feature grande, antes de escribir código.
disable-model-invocation: true
argument-hint: 'descripción corta de la feature o requerimiento'
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion, Bash(ls:*), Bash(cat:*), Bash(date:*)
---

# /spec — Diseñador guiado de specs

## Contexto de sesión

Fecha de hoy (usar esta para el encabezado de la spec, nunca adivinarla):
!`date +%F`

Specs que ya existen:
!`ls specs/ 2>/dev/null || echo "La carpeta specs/ todavía no existe"`

---

Este skill te ayuda a producir una spec útil siguiendo el método spec-driven. **Acá no escribís código.** Tu trabajo es ayudar al usuario a clarificar qué quiere construir, hacer preguntas cuando algo no está lo suficientemente bien definido, y desarrollar la spec sección por sección hasta que esté lista para guardarse en `specs/`.

## Filosofía

Una spec no es documentación decorativa. Es el contrato que guía la ejecución posterior. Si la spec es vaga, el código va a improvisar. Por eso este flujo es **deliberadamente lento durante la fase de definición** y **rápido durante la fase de escritura**.

Leé `template.md` (en el mismo directorio que este skill) para ver la estructura completa que va a seguir la spec. Apoyate en ella en cada paso.

## Flujo del comando

- Seguí las cuatro fases en orden. **Nunca te saltees la Fase 2** — las preguntas son el punto central. Si el usuario quiere ir más rápido, recordale que el costo de una mala spec se paga después en el código. (La Fase 3 sí tiene un camino rápido una vez que la Fase 2 está genuinamente completa; ver abajo.)
- Tus respuestas deben estar en el mismo idioma que el prompt inicial. Ej.: si el prompt inicial está en español, tus respuestas deben estar en español; si está en inglés, tus respuestas deben estar en inglés.

### Fase 1 — Entender el contexto

Antes de preguntar sobre la feature, asegurate de tener contexto del proyecto:

1. Leé el archivo de memoria del proyecto, si existe. Probá en orden y detenete en el primero que encuentres: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `README.md`. Esto adapta el skill al agente que lo esté ejecutando (Claude Code, Codex, Gemini CLI, etc.).
2. Mirá el listado de `specs/` en el contexto de sesión de arriba para ver qué specs ya existen y cómo están numeradas.
3. Si existen specs previas, leé al menos las dos más recientes para captar las convenciones del proyecto — incluyendo el **idioma** en que están escritas y la redacción exacta que usan para los estados y los títulos de sección. Una spec nueva debe coincidir con las existentes.

Si el argumento `$ARGUMENTS` llega vacío, pedile al usuario una descripción inicial en **una sola oración** de qué quiere construir. Si la descripción no entra en una oración, esa es la primera señal de que la feature es demasiado grande — sugerí dividirla antes de continuar.

### Fase 2 — Clarificar mediante preguntas

Esta es la fase más importante del comando. Tu trabajo acá es **detectar ambigüedades y preguntar**, no asumir.

Hacé las preguntas en bloques de 3 a 5 por vez (no una sola pregunta seguida de otra sola pregunta — eso es agotador). Después de cada bloque, esperá una respuesta antes de continuar.

**Categorías de preguntas que siempre deberías considerar:**

- **Alcance:** ¿Qué está adentro y qué NO? ¿Qué partes de la feature se difieren a otra spec?
- **Datos:** ¿Qué estructuras nuevas se introducen? ¿Cómo se llaman? ¿Dónde viven?
- **Integración:** ¿Esta feature depende de specs anteriores? ¿Modifica algo existente o solo agrega?
- **Persistencia:** ¿Se guarda algo entre sesiones? ¿Dónde? ¿Con qué versionado?
- **UX y estados:** ¿Cómo se ve cuando funciona? ¿Cómo se ve cuando falla? ¿Hay estados intermedios?
- **Riesgos:** ¿Qué puede romper esto? ¿Qué pasa en el caso degradado?
- **Decisiones cerradas:** ¿Hay alguna decisión que el usuario ya tomó y no quiere reabrir?

**Cómo formular las preguntas:**

- Usá preguntas concretas, no abiertas. ❌ "¿Cómo te imaginás la persistencia?" → ✅ "¿La persistencia es localStorage, IndexedDB, o un archivo JSON en disco?"
- Cuando ofrezcas opciones, dá entre 2 y 4, marcando cuál es tu recomendación y por qué.
- Si tu agente expone una herramienta nativa de preguntas de opción múltiple (en Claude Code: `AskUserQuestion`), usala para estos bloques en vez de escribir las opciones como prosa — el usuario elige en lugar de tipear. Poné tu recomendación primero y etiquetala. Si no existe esa herramienta, usá una lista markdown numerada.
- Si detectás una respuesta que abriría la caja de Pandora (ej.: "y también queremos multiplayer"), señalá que eso merece su propia spec y preguntá si lo dejamos fuera del alcance de esta.

**Cuándo dejar de preguntar:**

Parar cuando puedas responder estas tres preguntas sin asumir nada:

1. ¿Qué archivos van a aparecer o cambiar?
2. ¿Cuál es el primer paso ejecutable y cuál el último?
3. ¿Cómo verifico que la feature está terminada?

Si todavía no podés responder alguna, seguí preguntando.

### Fase 3 — Escribir la spec

Una vez cerrada la Fase 2, decidí cómo escribirla:

**Si ya tenés toda la información que necesitás** — es decir, podés responder las tres preguntas de la Fase 2 (qué archivos cambian, cuáles son el primer y último paso ejecutable, cómo verificar que está terminada) **sin asumir nada** — entonces **no vayas sección por sección**. Escribí la spec completa y saltá directo a la Fase 4 para guardar el archivo. No pidas confirmación sección por sección, ni muestres un borrador para aprobación primero: el usuario ya respondió todo en la Fase 2, y volver a preguntar es fricción. El usuario revisa el archivo guardado y pide cambios si hace falta.

**Solo si falta información** (el usuario cortó la Fase 2 antes de tiempo, una respuesta fue vaga, o alguna sección no se puede escribir sin inventar algo), desarrollá las secciones **una por una**, mostrando cada una y esperando confirmación antes de pasar a la siguiente.

En ambos casos el contenido sigue el mismo orden:

1. **Encabezado** (estado, dependencias, fecha, objetivo en una oración). El objetivo en una oración es crítico — si no entra en una oración, volvé a la Fase 2.
2. **Alcance** (qué está adentro y qué NO). El "no está" debe ser explícito.
3. **Modelo de datos** (estructuras concretas con nombres reales). Si la feature no introduce datos nuevos, omití esta sección y decilo explícitamente.
4. **Plan de implementación** (pasos numerados, cada uno dejando el sistema funcional).
5. **Criterios de aceptación** (checklist booleano, no aspiracional).
6. **Decisiones tomadas y descartadas** (con justificación breve).
7. **Riesgos identificados** (solo si aplica — si no hay riesgos relevantes, omití esta sección).

**Después de cada sección (solo en el modo sección por sección):**

- Mostrala formateada en markdown.
- Preguntá: "¿Esta sección queda así o querés ajustar algo?".
- Si el usuario pide cambios, aplicalos y mostrala de nuevo.
- Pasá a la siguiente sección recién cuando el usuario confirme.

**Errores comunes a evitar:**

- Generar criterios de aceptación que no son verificables ("que funcione bien").
- Poner en el plan de implementación cosas que no están en el alcance.
- Asumir nombres de archivos o estructuras que el usuario no confirmó.
- Saltearse la sección de decisiones — es la que tiene más valor a largo plazo.

### Fase 4 — Guardar la spec

Cuando el contenido esté listo (ya sea porque tenías todo, o porque todas las secciones fueron confirmadas):

1. Determiná el siguiente número secuencial a partir del listado de `specs/` en el contexto de sesión. Tomá el número más alto existente y sumale uno, con cero a la izquierda hasta dos dígitos. Si el último es `02-powerups.md`, este será `03-`. Si `specs/` está vacío o no existe, empezá en `01-`.
2. Generá un slug corto en kebab-case a partir del objetivo (ej.: `levels-and-highscores`). Ver **Argumentos** más abajo para cuando `$ARGUMENTS` es el slug en lugar de la descripción.
3. Usá la fecha del contexto de sesión de arriba para el campo `**Date:**`. **Nunca escribas una fecha que no hayas leído de ahí.**
4. Escribí el archivo directamente en `specs/NN-slug.md` con todas las secciones. **No pidas permiso para escribirlo ni preguntes si el nombre del archivo está bien** — anunciá la ruta en la confirmación final. Solo preguntá si el archivo destino ya existe.
5. Marcá el estado como `Draft` por defecto (o la palabra equivalente usada por las specs existentes en este repo). **No lo marques como `Approved` automáticamente** — eso lo hace el usuario una vez que la haya releído.
6. Si el encabezado lista dependencias (`**Depends on:** SPEC 01`), verificá que cada spec referenciada exista realmente en `specs/`. Si alguna no existe, decilo en vez de escribir una referencia colgante.
7. **Sembrá el archivo de configuración si no existe.** Verificá `specs/.spec-config.yml`. Si **falta**, creálo con el contenido por defecto de abajo. Si **ya existe, dejalo intacto** — nunca sobrescribas la configuración del usuario.

   ```yaml
   # configuración del flujo de spec
   #
   # AutoCreateBranch — controla si /spec-impl crea la rama de git automáticamente.
   #   true  (default) → /spec-impl crea y cambia a spec-NN-slug sin preguntar
   #   false           → /spec-impl pide confirmación [y/N] antes de crear la rama
   AutoCreateBranch: false
   ```

8. Confirmale al usuario:
   - Ruta del archivo creado.
   - Recordatorio: la spec está en estado `Draft`. Cambiala a `Approved` una vez que la hayas releído.
   - Si acabás de crear `specs/.spec-config.yml`, mencioná que existe y que `AutoCreateBranch` tiene por defecto `true` (poné `false` si querés controlar vos mismo la creación de ramas).
   - Próximo paso: una vez revisada y aprobada, ejecutar `/spec-impl NN-slug` para implementarla.
   - **Parar acá.** No propongas implementar la spec, escribir código, ni tomar ninguna acción más allá de esta confirmación.

## Reglas duras

- **Nunca escribir código durante este comando.** Solo el archivo `.md` de la spec al final.
- **Nunca proponer implementar la spec después de guardarla.** Tu trabajo termina cuando el archivo está escrito. El usuario ejecuta `/spec-impl` cuando esté listo.
- **Nunca asumir decisiones que el usuario no confirmó.** Si te falta información, preguntá — en la Fase 2, que es donde pertenecen las preguntas.
- **No vuelvas a preguntar en la Fase 3 lo que ya se respondió en la Fase 2.** Si la información está completa, escribí toda la spec y guardala. La confirmación sección por sección es el respaldo para información incompleta, no el default.
- **Si el usuario quiere acelerar y saltearse la Fase 2**, recordale: "Las preguntas ahora ahorran horas después. ¿Estás seguro de que querés saltearlas?". Si insiste, respetá su decisión pero registralo en la sección de decisiones de la spec ("Definición rápida sin clarificación detallada").
- **Si la feature es demasiado grande** (no entra en una oración, toca más de tres áreas del sistema, requiere decisiones en cuatro o más dominios), proponé dividirla en dos o más specs antes de continuar.

## Tono al hacer preguntas

Sé directo y específico. No pidas disculpas por preguntar. No uses frases como "si no te molesta..." o "¿podrías tal vez...?". El usuario invocó este skill precisamente porque quiere que preguntes. Usá preguntas concretas, una por línea cuando haya varias, y numeralas para que sean fáciles de responder.

Ejemplo de un bloque bien formado:

> Antes de escribir el modelo de datos necesito clarificar tres cosas:
>
> 1. **Persistencia.** ¿localStorage, IndexedDB, o un archivo JSON en disco? Recomendación: localStorage si los datos entran en <5MB y no necesitan queries.
> 2. **Versionado de esquema.** ¿Qué pasa cuando cambia el formato? Opciones: (a) prefijo de versión en la key, (b) ignorar y reconstruir, (c) migrar al cargar.
> 3. **Privacidad.** ¿Los datos son sensibles? Si sí, ¿están encriptados? ¿Se borran al cerrar sesión?

## Argumentos

`$ARGUMENTS` es **la descripción de la feature**, no el nombre del archivo. Tratalo como el punto de partida para la Fase 1 y derivá el slug del objetivo en la Fase 4.

La única excepción: si `$ARGUMENTS` ya es un único token en kebab-case sin espacios (ej.: `/spec levels-and-highscores`), es ambiguo entre una descripción y un slug — usalo como slug **y** como semilla de la descripción, sin pedir confirmación.

Si invocaron `/spec` sin argumentos, empezá pidiendo la descripción en una oración.
