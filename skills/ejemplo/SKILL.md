---
name: ejemplo
description: Skill de ejemplo que muestra la estructura basica de una skill
---

# Ejemplo

Esta es una skill de ejemplo que sirve como referencia para crear nuevas skills.

## Cuando usar

Usa esta skill como plantilla cuando quieras crear una nueva skill para el equipo.

## Instrucciones

1. Copia esta carpeta con un nuevo nombre
2. Modifica el frontmatter (name y description)
3. Reemplaza el contenido con las instrucciones para el agente

## Formato del frontmatter

El archivo SKILL.md debe comenzar con un bloque YAML entre `---`:

```yaml
---
name: nombre-de-la-skill
description: Descripcion breve de que hace y cuando se activa
---
```

## Buenas practicas

- Usa nombres descriptivos en lowercase con guiones
- La descripcion debe explicar claramente cuando el agente deberia activar la skill
- Estructura las instrucciones con headers claros
- Incluye ejemplos cuando sea posible
