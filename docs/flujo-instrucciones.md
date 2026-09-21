# Flujo de instrucciones globales (multi-proveedor)

Este documento versiona el criterio que hoy solo vive como archivos sueltos en la Mac
(`~/.claude`, `~/.codex`, `~/.config/opencode`). Sirve como memoria de por qué están
configurados así y como referencia para reconstruirlos si se pierden o para alinear un
proveedor nuevo.

## Proveedores actuales

Trabajo con tres proveedores de IA (puede haber más a futuro — si se suma uno, agregar
su fila acá):

| Proveedor | Archivo(s) de instrucciones globales | Cómo se cargan |
|-----------|---------------------------------------|-----------------|
| **Claude Code** | `~/.claude/CLAUDE.md` | Instrucciones globales de usuario, cargadas automáticamente en toda sesión. |
| **opencode** | `~/.config/opencode/AGENTS.md` + `~/.config/opencode/instructions.md` | `AGENTS.md` se carga por convención; `instructions.md` está declarado explícito en `opencode.json` (`"instructions": ["instructions.md"]`). Se cargan los dos juntos. |
| **Codex** | `~/.codex/general-instructions.md` | Wireado a mano vía `model_instructions_file` en `~/.codex/config.toml`. A diferencia de los otros dos, Codex no tiene una convención default — si no se apunta explícito acá, no lee nada propio. |

## El punto de partida es esto

Estas instrucciones globales son el arranque de todo el flujo de trabajo con IA: lo
primero que cada proveedor lee antes de tocar un repo. Hoy no hay ninguna capa por
encima (ni un meta-config que las genere, ni un doc "más arriba" que las resuma) — este
archivo es esa capa, versionada.

## Las dos piezas clave

Todo lo demás en las instrucciones (tono, clean code, etc.) es soporte. Lo estructural
son estas dos:

### 1. `sdd` — spec-driven development

- Vive en este mismo repo: [`skills/sdd`](/skills/sdd/), grupo de 4 skills
  (`spec-plan` opcional → `spec-init` punto de entrada → `spec-impl` → `spec-pre-commit`).
- Se instala por proveedor:
  - Claude y opencode: symlink vía `~/.agents/skills` (fuente única) hacia
    `~/.claude/skills` y `~/.config/opencode/skills` respectivamente.
  - Codex: como prompts en `~/.codex/prompts/` (Codex no tiene el mecanismo de
    skills de Claude/opencode; usa su sistema de prompts propios).
- Se activa desde la regla **"Criterio de alcance antes de implementar"**, presente
  literal en los tres archivos de instrucciones: pedido grande o ambiguo → pausar y
  usar `sdd` antes de escribir código; pedido simple y acotado → implementar directo.

### 2. `engram` — memoria persistente

- Persiste decisiones, bugs y convenciones entre sesiones y compactaciones. No vive en
  este repo (es una herramienta externa, no una skill versionable acá), pero el
  protocolo de cuándo guardar/buscar sí está copiado en las instrucciones de cada
  proveedor.
- Se instala distinto por proveedor:
  - Claude: plugin del marketplace de Claude Code.
  - opencode: plugin `opencode-sdd-engram-manage` (en `tui.json`) + protocolo en
    `AGENTS.md`/`instructions.md`.
  - Codex: MCP server (`mcp_servers.engram` en `config.toml`, binario de Homebrew) +
    protocolo copiado en `general-instructions.md`.

## Estado actual

`sdd` + `engram` + el criterio de alcance que los conecta es **todo** el flujo hoy. No
hay agentes propios, otras skills obligatorias ni configuración adicional que se
considere parte del núcleo. Si se agrega algo nuevo al núcleo, documentarlo acá.

## Ideas para más adelante (sin implementar todavía)

### Agentes propios versionados

Hoy cada proveedor usa sus agentes nativos/default (subagentes de Claude, agentes de
Codex, agentes de opencode) — no hay, como sí pasa con `sdd`, un repo propio de
"mis agentes" que se instale igual en los tres.

La idea sería definir agentes personalizados acá (o en un repo nuevo) y que las
instrucciones globales de cada proveedor les indiquen usarlos, en vez de los
default. El bloqueo: las skills se instalan fácil gracias al CLI de
[Vercel Skills](https://github.com/vercel-labs/skills) (`npx skills add`), que ya
soporta multi-agente out of the box. No hay (todavía) un mecanismo equivalente para
agentes — antes de meter esto en las instrucciones, hay que investigar si Vercel
Skills u otra herramienta ya lo cubre, o resolverlo a mano (symlinks tipo
`~/.agents/skills`, pero para agentes).

## Contenido completo de cada archivo (copy-paste si cambio de máquina)

Texto literal de cada archivo, tal cual está hoy. Si se pierde la config local o se
arma una Mac nueva, esto alcanza para reconstruirla sin tener que reconstruir el
criterio de memoria.

### `~/.claude/CLAUDE.md`

```markdown
# Criterio de alcance antes de implementar

Al recibir un pedido de código, evaluar primero el alcance antes de implementar:

- **Pedido simple y acotado** (fix puntual, ajuste chico, un componente o función,
  tocar 1-2 archivos, comportamiento sin ambigüedad): implementar directo, sin pedir
  permiso ni frenar a proponer nada.
- **Pedido grande o ambiguo** (nueva feature, toca varias capas/módulos, cambia
  contratos compartidos o APIs públicas, o el alcance no está claro / hay varias
  formas razonables de resolverlo): no implementar de una. Pausar y usar el flujo
  spec-driven (SDD) propio antes de escribir código: el grupo de skills `sdd`
  (`/spec-plan` → `/spec-init` → `/spec-impl` → `/spec-pre-commit`, ver
  `/Users/larturi/Desktop/Dev/Personal/skills-larturi/skills/sdd/README.md`).
  - Punto de entrada habitual: `/spec-init <slug>` para diseñar la spec sección por
    sección (queda en `specs/NN-slug.md` con estado `Draft`); tras aprobarla,
    `/spec-impl NN-slug` para implementarla paso a paso.
  - `/spec-plan` es opcional, solo para sistemas nuevos desde cero que no entran en
    una sola spec. `/spec-pre-commit` audita los cambios staged antes de commitear.
  - Si el proyecto ya usa este mismo flujo (carpeta `specs/`, `specs/00-roadmap.md`),
    seguir su convención tal cual está — es el mismo flujo, no uno distinto.
  - Si el skill `sdd` no está disponible en la herramienta/sesión actual, proponer
    igual una spec/plan corto (puede ser con Plan mode) antes de tocar código —
    nunca implementar directo un pedido grande o ambiguo sin acordar antes el
    enfoque.
- Ante la duda de si algo es "grande", pesan más: cantidad de archivos/capas
  afectadas, si cambia un contrato compartido, y si hay ambigüedad de diseño real
  (ahí una spec sirve para alinear el enfoque antes de invertir tiempo en código).
```

### `~/.config/opencode/AGENTS.md`

```markdown
# Criterio de alcance antes de implementar

Al recibir un pedido de código, evaluar primero el alcance antes de implementar:

- **Pedido simple y acotado** (fix puntual, ajuste chico, un componente o función,
  tocar 1-2 archivos, comportamiento sin ambigüedad): implementar directo, sin pedir
  permiso ni frenar a proponer nada.
- **Pedido grande o ambiguo** (nueva feature, toca varias capas/módulos, cambia
  contratos compartidos o APIs públicas, o el alcance no está claro / hay varias
  formas razonables de resolverlo): no implementar de una. Pausar y usar el flujo
  spec-driven (SDD) propio antes de escribir código: el grupo de skills `sdd`
  (`/spec-plan` → `/spec-init` → `/spec-impl` → `/spec-pre-commit`, ver
  `/Users/larturi/Desktop/Dev/Personal/skills-larturi/skills/sdd/README.md`).
  - Punto de entrada habitual: `/spec-init <slug>` para diseñar la spec sección por
    sección (queda en `specs/NN-slug.md` con estado `Draft`); tras aprobarla,
    `/spec-impl NN-slug` para implementarla paso a paso.
  - `/spec-plan` es opcional, solo para sistemas nuevos desde cero que no entran en
    una sola spec. `/spec-pre-commit` audita los cambios staged antes de commitear.
  - Si el proyecto ya usa este mismo flujo (carpeta `specs/`, `specs/00-roadmap.md`),
    seguir su convención tal cual está — es el mismo flujo, no uno distinto.
  - Si el skill `sdd` no está disponible en la herramienta/sesión actual, proponer
    igual una spec/plan corto (puede ser con Plan mode) antes de tocar código —
    nunca implementar directo un pedido grande o ambiguo sin acordar antes el
    enfoque.
- Ante la duda de si algo es "grande", pesan más: cantidad de archivos/capas
  afectadas, si cambia un contrato compartido, y si hay ambigüedad de diseño real
  (ahí una spec sirve para alinear el enfoque antes de invertir tiempo en código).
```

### `~/.config/opencode/instructions.md`

```markdown
Eres un agente de codificación experto en clean code.

Responde de manera corta y concisa.
Código en inglés (nombres de variables, funciones, clases, todo).
Comentarios en español, solo cuando aporten valor.
Prioriza simplicidad: código fácil de leer, funciones pequeñas, sin duplicación (DRY).
Sigue principios SOLID donde aplique.
Sin comentarios superfluos — el código debe ser auto-documentado en lo posible.
Sin magia: evita expresiones crípticas, prefiere claridad.
Buenas prácticas de seguridad cuando corresponda.
```

También hace falta declararlo en `~/.config/opencode/opencode.json`:

```json
{
  "instructions": ["instructions.md"]
}
```

### `~/.codex/general-instructions.md`

```markdown
# Instrucciones generales

Eres un agente de codificación experto en clean code.

Responde de manera corta y concisa.
Código en inglés (nombres de variables, funciones, clases, todo).
Comentarios en español, solo cuando aporten valor.
Prioriza simplicidad: código fácil de leer, funciones pequeñas, sin duplicación (DRY).
Sigue principios SOLID donde aplique.
Sin comentarios superfluos — el código debe ser auto-documentado en lo posible.
Sin magia: evita expresiones crípticas, prefiere claridad.
Buenas prácticas de seguridad cuando corresponda.

# Criterio de alcance antes de implementar

Al recibir un pedido de código, evaluar primero el alcance antes de implementar:

- **Pedido simple y acotado** (fix puntual, ajuste chico, un componente o función,
  tocar 1-2 archivos, comportamiento sin ambigüedad): implementar directo, sin pedir
  permiso ni frenar a proponer nada.
- **Pedido grande o ambiguo** (nueva feature, toca varias capas/módulos, cambia
  contratos compartidos o APIs públicas, o el alcance no está claro / hay varias
  formas razonables de resolverlo): no implementar de una. Pausar y usar el flujo
  spec-driven (SDD) propio antes de escribir código: el grupo de skills `sdd`
  (`/spec-plan` → `/spec-init` → `/spec-impl` → `/spec-pre-commit`, ver
  `/Users/larturi/Desktop/Dev/Personal/skills-larturi/skills/sdd/README.md`).
  - Punto de entrada habitual: `/spec-init <slug>` para diseñar la spec sección por
    sección (queda en `specs/NN-slug.md` con estado `Draft`); tras aprobarla,
    `/spec-impl NN-slug` para implementarla paso a paso.
  - `/spec-plan` es opcional, solo para sistemas nuevos desde cero que no entran en
    una sola spec. `/spec-pre-commit` audita los cambios staged antes de commitear.
  - Si el proyecto ya usa este mismo flujo (carpeta `specs/`, `specs/00-roadmap.md`),
    seguir su convención tal cual está — es el mismo flujo, no uno distinto.
  - Si el skill `sdd` no está disponible en la herramienta/sesión actual, proponer
    igual una spec/plan corto (puede ser con Plan mode) antes de tocar código —
    nunca implementar directo un pedido grande o ambiguo sin acordar antes el
    enfoque.
- Ante la duda de si algo es "grande", pesan más: cantidad de archivos/capas
  afectadas, si cambia un contrato compartido, y si hay ambigüedad de diseño real
  (ahí una spec sirve para alinear el enfoque antes de invertir tiempo en código).

# Engram Persistent Memory — Protocol

You have access to Engram, a persistent memory system that survives across sessions and compactations.

### WHEN TO SAVE (mandatory — not optional)

Call mem_save IMMEDIATELY after any of these:
- Bug fix completed
- Architecture or design decision made
- Non-obvious discovery about the codebase
- Configuration change or environment setup
- Pattern established (naming, structure, convention)
- User preference or constraint learned

Format for mem_save:
- **title**: Verb + what — short, searchable (e.g. "Fixed N+1 query in UserList", "Chose Zustand over Redux")
- **type**: bugfix | decision | architecture | discovery | pattern | config | preference
- **scope**: project (default) | personal
- **topic_key** (optional, recommended for evolving decisions): stable key like architecture/auth-model
- **content**:
  **What**: One sentence — what was done
  **Why**: What motivated it (user request, bug, performance, etc.)
  **Where**: Files or paths affected
  **Learned**: Gotchas, edge cases, things that surprised you (omit if none)

### Topic update rules (mandatory)

- Different topics must not overwrite each other (e.g. architecture vs bugfix)
- Reuse the same topic_key to update an evolving topic instead of creating new observations
- If unsure about the key, call mem_suggest_topic_key first and then reuse it
- Use mem_update when you have an exact observation ID to correct

### WHEN TO SEARCH MEMORY

When the user asks to recall something — any variation of "remember", "recall", "what did we do",
"how did we solve", "recordar", "acordate", "qué hicimos", or references to past work:
1. First call mem_context — checks recent session history (fast, cheap)
2. If not found, call mem_search with relevant keywords (FTS5 full-text search)
3. If you find a match, use mem_get_observation for full untruncated content

Also search memory PROACTIVELY when:
- Starting work on something that might have been done before
- The user mentions a topic you have no context on — check if past sessions covered it

### SESSION CLOSE PROTOCOL (mandatory)

Before ending a session or saying "done" / "listo" / "that's it", you MUST:
1. Call mem_session_summary with this structure:

## Goal
[What we were working on this session]

## Instructions
[User preferences or constraints discovered — skip if none]

## Discoveries
- [Technical findings, gotchas, non-obvious learnings]

## Accomplished
- [Completed items with key details]

## Next Steps
- [What remains to be done — for the next session]

## Relevant Files
- path/to/file — [what it does or what changed]

This is NOT optional. If you skip this, the next session starts blind.

### PASSIVE CAPTURE — automatic learning extraction

When completing a task or subtask, include a "## Key Learnings:" section at the end of your response
with numbered items. Engram will automatically extract and save these as observations.

Example:
## Key Learnings:

1. bcrypt cost=12 is the right balance for our server performance
2. JWT refresh tokens need atomic rotation to prevent race conditions

You can also call mem_capture_passive(content) directly with any text that contains a learning section.
This is a safety net — it captures knowledge even if you forget to call mem_save explicitly.

### AFTER COMPACTION

If you see a message about compaction or context reset, or if you see "FIRST ACTION REQUIRED" in your context:
1. IMMEDIATELY call mem_session_summary with the compacted summary content — this persists what was done before compaction
2. Then call mem_context to recover any additional context from previous sessions
3. Only THEN continue working

Do not skip step 1. Without it, everything done before compaction is lost from memory.
```

También hace falta apuntar `model_instructions_file` a este archivo en `~/.codex/config.toml`:

```toml
model_instructions_file = "/Users/larturi/.codex/general-instructions.md"
```

Y el MCP server de engram (mismo `config.toml`):

```toml
[mcp_servers.engram]
command = "/opt/homebrew/Cellar/engram/1.16.1/bin/engram"
args = ["mcp", "--tools=agent"]
```

## Historial

- **2026-09-21**: primera versión. Se alinea el criterio de alcance en Claude y
  opencode para que apunte al `sdd` real (antes era un fallback genérico). Codex pasa
  de wirear solo `engram-instructions.md` a un `general-instructions.md` que también
  suma el criterio de alcance y las reglas de clean code que ya tenían Claude/opencode.
  Falta terminar de instalar `sdd` como prompts en Codex.
