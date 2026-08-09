---
name: feature-flags-cleanup
description: Detecta feature flags en el codebase, lista su estado actual (incluyendo .env local), y genera reportes estilo Jira para planificar su eliminacion
---

# Feature Flags Cleanup

Skill para identificar, auditar y planificar la eliminacion de feature flags en un proyecto.

## Cuando usar

- Cuando se quiera hacer limpieza de feature flags que ya no son necesarias
- Cuando se necesite un inventario de todas las feature flags activas en el proyecto
- Cuando se quiera generar tickets de trabajo para remover flags obsoletas
- Periodicamente como higiene del codebase

## Instrucciones

### Paso 1: Detectar feature flags en el codebase

Busca feature flags en el proyecto usando los siguientes patrones comunes:

1. **Variables de entorno con prefijos tipicos:**
   - `FEATURE_*`, `FF_*`, `FLAG_*`, `ENABLE_*`, `TOGGLE_*`
   - `NEXT_PUBLIC_FEATURE_*`, `REACT_APP_FEATURE_*`, `VITE_FEATURE_*`

2. **Archivos de configuracion de flags:**
   - Archivos como `flags.ts`, `flags.js`, `features.ts`, `featureFlags.ts`, `toggles.ts`
   - Configuraciones en JSON/YAML que definan flags

3. **Patrones en codigo:**
   - Llamadas a funciones como `isFeatureEnabled()`, `useFeatureFlag()`, `getFlag()`, `hasFeature()`
   - Condicionales que evaluen flags: `if (features.X)`, `if (process.env.FEATURE_X)`
   - SDKs de feature flags (LaunchDarkly, Unleash, Split, Flagsmith, etc.)

4. **Archivos .env:**
   - Lee `.env`, `.env.local`, `.env.development`, `.env.example` para encontrar flags definidas

### Paso 2: Leer el estado local desde .env

Lee los archivos `.env` y `.env.local` (u otros variantes presentes) para determinar el estado actual de cada flag en el entorno local del desarrollador:

- `true` / `1` / `"enabled"` → Activada localmente
- `false` / `0` / `"disabled"` / no definida → Desactivada localmente
- Si no existe en .env pero si en el codigo → Marcar como "sin valor local definido"

### Paso 3: Presentar el inventario

Presenta las flags encontradas en formato tabla con las siguientes columnas:

| Flag | Estado Local | Archivos donde se usa | Contexto |
|------|-------------|----------------------|----------|
| `FEATURE_NUEVO_CHECKOUT` | Activada | `src/checkout/index.ts`, `src/api/routes.ts` | Controla el flujo de checkout v2 |
| `FF_DARK_MODE` | Desactivada | `src/theme/provider.tsx` | Habilita el modo oscuro experimental |

Para cada flag incluir:
- **Nombre** de la flag
- **Estado local** segun .env
- **Archivos** donde se referencia (listar los principales, no mas de 5)
- **Contexto** breve de que controla (inferido del codigo circundante)

### Paso 4: Seleccion de flags a eliminar

Pregunta al usuario cuales flags desea eliminar. Puede elegir una o varias.

Para ayudar en la decision, sugiere cuales son candidatas a eliminar basandote en:
- Flags que estan activadas en todos los entornos (ya son el comportamiento default)
- Flags que referencian codigo muy antiguo (basarse en git blame si es posible)
- Flags desactivadas que no han cambiado en mucho tiempo

### Paso 5: Generar reporte de eliminacion

Por cada flag seleccionada, genera un reporte con el siguiente formato:

---

## Reporte de Eliminacion de Feature Flag

**Titulo:** [CLEANUP] Eliminar feature flag `NOMBRE_DE_LA_FLAG`

**Descripcion:**

La feature flag `NOMBRE_DE_LA_FLAG` fue introducida para [contexto inferido del codigo]. Actualmente se encuentra [activada/desactivada] en el entorno local y se utiliza en los siguientes archivos:

- `ruta/al/archivo1.ts` (linea X): [breve descripcion del uso]
- `ruta/al/archivo2.ts` (linea Y): [breve descripcion del uso]

**Objetivo:**

Eliminar la feature flag `NOMBRE_DE_LA_FLAG` del codebase, consolidando el comportamiento [activo/inactivo] como permanente. Esto implica:

1. Eliminar la variable de entorno de todos los archivos `.env*`
2. Eliminar las condiciones/branching asociados a la flag en:
   - [lista de archivos afectados]
3. Eliminar la definicion de la flag en archivos de configuracion
4. [Si la flag esta activada] Mantener solo el codigo del path "activado" y eliminar el path alternativo
5. [Si la flag esta desactivada] Eliminar el codigo del path "activado" que nunca se ejecuta
6. Actualizar tests que mockeen o evaluen la flag

**Criterios de aceptacion:**

- [ ] No quedan referencias a `NOMBRE_DE_LA_FLAG` en el codebase
- [ ] Los tests pasan sin modificaciones adicionales
- [ ] El comportamiento de la aplicacion no cambia (la flag ya estaba [activada/desactivada])
- [ ] Se removio de la documentacion interna si aplica

---

## Ejemplo

Si el agente encuentra la flag `FEATURE_NEW_ONBOARDING` activada en `.env` y usada en 3 archivos, el reporte seria:

**Titulo:** [CLEANUP] Eliminar feature flag `FEATURE_NEW_ONBOARDING`

**Descripcion:**
La feature flag `FEATURE_NEW_ONBOARDING` fue introducida para controlar el nuevo flujo de onboarding de usuarios. Actualmente se encuentra activada en el entorno local y se utiliza en:

- `src/onboarding/flow.tsx` (linea 12): Renderiza condicionalmente el nuevo wizard
- `src/api/users.ts` (linea 45): Envia evento analytics diferenciado
- `src/config/features.ts` (linea 8): Definicion de la flag

**Objetivo:**
Eliminar la feature flag `FEATURE_NEW_ONBOARDING` del codebase, consolidando el nuevo flujo de onboarding como comportamiento permanente. Esto implica:

1. Eliminar `FEATURE_NEW_ONBOARDING` de `.env`, `.env.example`, `.env.development`
2. En `src/onboarding/flow.tsx`: remover el condicional y dejar solo el nuevo wizard
3. En `src/api/users.ts`: mantener el evento analytics del nuevo flujo
4. En `src/config/features.ts`: eliminar la entrada de la flag
5. Actualizar tests en `src/onboarding/__tests__/` que mockeen la flag

## Notas

- Si el proyecto usa un SDK de feature flags (LaunchDarkly, Unleash, etc.), mencionar que tambien debe eliminarse del dashboard del servicio
- Si hay flags que dependen de otras flags, indicar la dependencia en el reporte
- Siempre verificar que no haya logica de rollback atada a la flag antes de recomendar su eliminacion
