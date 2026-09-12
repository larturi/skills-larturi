# Agregar o modificar una skill

Guía para contribuir al catálogo. Para instalarlas y usarlas desde Kiro, ver el [README](../README.md).

## Dónde va el archivo

Por defecto, la skill va suelta en la raíz del catálogo:

```
skills/<nombre-de-la-skill>/SKILL.md
```

Agrupala en una carpeta intermedia **solo si colabora con otras skills en un mismo flujo** (por ejemplo, una que orquesta y otras que se delegan entre sí):

```
skills/<grupo>/<nombre-de-la-skill>/SKILL.md
```

El grupo no es una categoría temática ni un requisito: es una forma de dejar juntas las skills que se necesitan entre sí, y de poder documentar el flujo en un `README.md` propio. Si la skill se usa sola, dejala suelta.

**No anides más niveles.** La CLI de `skills` descubre el layout plano (`skills/<skill>/`) y un nivel extra para el de catálogo (`skills/<grupo>/<skill>/`); una skill más adentro no se instala.

## Frontmatter

Mínimo requerido:

```yaml
---
name: <igual al nombre de la carpeta, en kebab-case>
description: Qué hace y **cuándo usarla**.
---
```

La `description` es lo único que el agente ve para decidir si aplica la skill: escribila pensando en el disparador ("usar cuando se pida…", "usar cuando se trabaje sobre…"), no solo en lo que hace.

## Cuerpo

Escribilo en español rioplatense, con:

- **Objetivo**: qué produce la skill y para qué sirve.
- **Flujo paso a paso**: qué mira el agente, qué le pregunta al usuario y en qué orden.
- **Convenciones**: nombres de archivos y carpetas que genera, con ejemplos.
- **Restricciones**: qué **no** debe generar ni ofrecer el agente. Esta sección es la que evita que la skill se desmadre; sé explícito (por ejemplo: "al terminar, no ofrecer generar otra prueba").

Si la skill participa de un flujo con otras, dejá escrito **con cuál continúa** y **qué hacer si no está instalada** (lo esperable: avisar y detener el flujo, no adivinar).

## Probarla antes de integrar

Instalala en Kiro sobre un repo real desde tu copia local, y recién después integrá a `master`:

```bash
npx skills add /ruta/a/qa-skills -a kiro-cli -s <nombre>
```

Verificá que el agente la active sola a partir de la `description`, sin que tengas que nombrarla.
