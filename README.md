# Skills

Hub de skills compartidos. Compatible con el ecosistema [Vercel Skills](https://github.com/vercel-labs/skills).

> Ver [docs/flujo-instrucciones.md](/docs/flujo-instrucciones.md) para el flujo completo
> de instrucciones globales multi-proveedor (Claude, Codex, opencode) en el que encajan
> estas skills.

## Instalacion

### Todas las skills del repo

```bash
npx skills add larturi/skills-larturi
```

### Una skill especifica

```bash
npx skills add larturi/skills-larturi --skill <nombre-de-la-skill>
```

### Para un agente especifico

```bash
npx skills add larturi/skills-larturi --agent kiro-cli
npx skills add larturi/skills-larturi --agent claude-code
npx skills add larturi/skills-larturi --agent cursor
```

## Skills disponibles

| Skill | Descripcion |
|-------|-------------|
| [feature-flags-cleanup](/skills/feature-flags-cleanup/) | Detecta feature flags y genera reportes para planificar su eliminacion |
| [generate-bff-collections](/skills/generate-bff-collections/) *(grupo)* | Genera colecciones de Postman: documenta la API completa de un BFF NestJS o prueba puntualmente un servicio externo que consume el repo |
| [sdd](/skills/sdd/) *(grupo)* | Metodo spec-driven: planifica sistemas grandes en un roadmap de specs, disena cada spec guiado por preguntas, la implementa paso a paso, y audita los cambios antes de commitear |

## Listar skills disponibles

```bash
npx skills add larturi/skills-larturi --list
```

## Como actualizar

Las skills no se actualizan solas: cuando se mergea un cambio al repo, hay que actualizar manualmente:

```bash
# Todas las skills instaladas
npx skills update

# Una skill especifica
npx skills update spec-init
```

Si instalaste con symlink (metodo default), un solo `update` refresca todos tus agentes a la vez.

## Estructura del repositorio

```
skills-larturi/
├── README.md
├── AGENTS.md
├── docs/
│   └── flujo-instrucciones.md
├── skills/
│   ├── mi-skill/
│   │   └── SKILL.md
│   ├── otro-grupo/
│   │   ├── README.md
│   │   ├── sub-skill-a/
│   │   │   └── SKILL.md
│   │   └── sub-skill-b/
│   │       └── SKILL.md
│   └── ...
└── CONTRIBUTING.md
```

## Crear una nueva skill

1. Crea una carpeta dentro de `skills/` con el nombre de tu skill (lowercase, guiones para separar palabras)
2. Agrega un archivo `SKILL.md` con el frontmatter YAML requerido
3. Abre un PR para revision del equipo

Consulta [CONTRIBUTING.md](./CONTRIBUTING.md) para mas detalles.

## Auditoria de prompts

Despues de crear o modificar skills, y en cada release de un modelo nuevo, correr en Claude Code:

```bash
/claude-api prompt-audit
```

Detecta instrucciones desactualizadas: referencias a campos, rutas o pasos que ya no existen, reglas que se contradicen entre skills y patrones escritos para modelos anteriores. Propone un diff; no aplica nada sin confirmacion.

## Agentes soportados

El CLI de skills detecta automaticamente los agentes instalados. Algunos de los mas usados:

- **Kiro CLI** (`kiro-cli`)
- **Claude Code** (`claude-code`)
- **Cursor** (`cursor`)
- **GitHub Copilot** (`github-copilot`)
- **Windsurf** (`windsurf`)

Ver la [lista completa de agentes soportados](https://github.com/vercel-labs/skills#supported-agents).
