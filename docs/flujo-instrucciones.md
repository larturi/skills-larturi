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

## Contenido completo (copy-paste si cambio de máquina)

Este es el bloque de instrucciones propias — estilo de código + criterio de alcance —,
idéntico hoy en los tres proveedores. No incluye el protocolo de `engram`: ese ya está
wireado por proveedor (ver arriba) y en Codex vive agregado a continuación de este
bloque dentro de `general-instructions.md`; no se duplica acá para no tener que
mantenerlo en dos lugares.

### `~/.claude/CLAUDE.md`
### `~/.config/opencode/AGENTS.md`
### `~/.config/opencode/instructions.md`
### `~/.codex/general-instructions.md` (más el protocolo de `engram` a continuación, no reproducido acá)

```markdown
# Estilo de código

Eres un agente de codificación experto en clean code, con 15 años de experiencia en desarrollo fullstack, arquitectura, liderazgo y habilidades de comunicación superlativas.

- Responde de manera corta y concisa.
- Código en inglés (nombres de variables, funciones, clases, todo).
- Comentarios en español, solo cuando aporten valor.
- Prioriza simplicidad: código fácil de leer, funciones pequeñas, sin duplicación (DRY).
- Sigue principios SOLID donde aplique.
- Sin comentarios superfluos — el código debe ser auto-documentado en lo posible.
- Sin magia: evita expresiones crípticas, prefiere claridad.
- Buenas prácticas de seguridad cuando corresponda.

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

- **2026-09-21**: se simplifica la sección de contenido copy-paste a un solo bloque
  (antes había uno repetido por proveedor). El protocolo de `engram` que Codex ya
  tenía agregado en `general-instructions.md` se deja de duplicar acá — se asume
  preexistente y se documenta solo su wireo, no su texto completo.
- **2026-09-21**: se agrega el bloque de estilo de código (clean code) a
  `~/.claude/CLAUDE.md`, que ya lo tenía opencode (`instructions.md`) y Codex
  (`general-instructions.md`). Los tres proveedores quedan alineados en ese
  criterio.
- **2026-09-21**: primera versión. Se alinea el criterio de alcance en Claude y
  opencode para que apunte al `sdd` real (antes era un fallback genérico). Codex pasa
  de wirear solo `engram-instructions.md` a un `general-instructions.md` que también
  suma el criterio de alcance y las reglas de clean code que ya tenían Claude/opencode.
  Falta terminar de instalar `sdd` como prompts en Codex.
