# Contribuir al hub de skills

## Crear una nueva skill

### 1. Crear la carpeta

```bash
mkdir skills/mi-nueva-skill
```

O usa el CLI para generar el template:

```bash
cd skills && npx skills init mi-nueva-skill
```

### 2. Estructura del SKILL.md

```markdown
---
name: mi-nueva-skill
description: Descripcion clara de que hace esta skill y cuando deberia activarse
---

# Mi Nueva Skill

Instrucciones detalladas para el agente.

## Cuando usar

Escenarios donde esta skill aplica.

## Pasos

1. Primer paso
2. Segundo paso
```

### 3. Campos requeridos del frontmatter

| Campo | Descripcion |
|-------|-------------|
| `name` | Identificador unico (lowercase, guiones permitidos) |
| `description` | Explicacion breve de que hace y cuando se usa |

### 4. Campos opcionales

| Campo | Descripcion |
|-------|-------------|
| `metadata.internal` | `true` para ocultar la skill del discovery publico |

## Convenciones de nombres

- Usa **lowercase** con guiones: `mi-skill-nombre`
- Que sea descriptivo y breve
- Evita prefijos genericos como `skill-` o `tool-`

## Estructura recomendada del contenido

1. **Titulo** - Nombre de la skill
2. **Descripcion** - Que hace
3. **Cuando usar** - Escenarios de activacion
4. **Pasos/Instrucciones** - Lo que el agente debe seguir
5. **Ejemplos** - Casos concretos (opcional pero recomendado)

## Proceso de PR

1. Crea un branch: `skill/<nombre-de-la-skill>`
2. Agrega tu skill en `skills/<nombre>/SKILL.md`
3. Actualiza la tabla de skills en el README.md principal
4. Abre un PR con una descripcion de que resuelve la skill
5. Espera review de al menos un miembro del equipo

## Testear localmente

Podes instalar skills desde un path local para probarlas antes del PR:

```bash
npx skills add ./skills/mi-nueva-skill
```
