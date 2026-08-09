# Skills

Hub de skills compartidos. Compatible con el ecosistema [Vercel Skills](https://github.com/vercel-labs/skills).

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
| [ejemplo](/skills/ejemplo/) | Skill de ejemplo para referencia |
| [feature-flags-cleanup](/skills/feature-flags-cleanup/) | Detecta feature flags y genera reportes para planificar su eliminacion |
| [pre-commit-audit](/skills/pre-commit-audit/) | Auditoria 4R de cambios staged antes de commitear, con veredicto de bloqueo |

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
npx skills update pre-commit-audit
```

Si instalaste con symlink (metodo default), un solo `update` refresca todos tus agentes a la vez.

## Estructura del repositorio

```
skills-larturi/
├── README.md
├── skills/
│   ├── ejemplo/
│   │   └── SKILL.md
│   ├── otra-skill/
│   │   └── SKILL.md
│   └── ...
└── CONTRIBUTING.md
```

## Crear una nueva skill

1. Crea una carpeta dentro de `skills/` con el nombre de tu skill (lowercase, guiones para separar palabras)
2. Agrega un archivo `SKILL.md` con el frontmatter YAML requerido
3. Abre un PR para revision del equipo

Consulta [CONTRIBUTING.md](./CONTRIBUTING.md) para mas detalles.

## Agentes soportados

El CLI de skills detecta automaticamente los agentes instalados. Algunos de los mas usados:

- **Kiro CLI** (`kiro-cli`)
- **Claude Code** (`claude-code`)
- **Cursor** (`cursor`)
- **GitHub Copilot** (`github-copilot`)
- **Windsurf** (`windsurf`)

Ver la [lista completa de agentes soportados](https://github.com/vercel-labs/skills#supported-agents).
