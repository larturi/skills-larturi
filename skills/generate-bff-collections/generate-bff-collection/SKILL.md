---
name: generate-bff-collection
description: Genera una colección de Postman con TODOS los endpoints propios de un BFF NestJS, agrupados por módulo, con environments local/uat/prod y una carpeta "Legacy Providers" con los servicios externos que consume cada endpoint. Usar cuando se pida documentar, exportar o mapear la API completa de un BFF a Postman, o cuando se necesite poder probar directo el provider que falla detrás de un endpoint del BFF.
---

# Generación de Colección Postman de un BFF (NestJS)

## Objetivo

A diferencia de `generate-external-tests` (que prueba un provider externo puntual seleccionado por el usuario), esta skill documenta el BFF completo: todos sus endpoints propios organizados por módulo, más una referencia cruzada al provider que llama cada uno, para poder saltar directo a probarlo si el endpoint falla.

## Paso 0: Detectar endpoints propios del BFF

Buscar controllers en `src/**/*.controller.ts`. Por cada método con `@Get/@Post/@Put/@Patch/@Delete`, extraer: método HTTP, path completo (prefix de `@Controller()` + path del método), DTOs de request/response y guards de auth. Agrupar por módulo, usando la misma organización de `src/**/*.module.ts`.

## Paso 1: Detectar el provider que llama cada endpoint

Rastrear controller → service → provider/client inyectado, aplicando la misma lógica de `discover-external-services-ag` (providers locales en `src/**/providers|clients`, y providers de librería compartida vía imports `@pv-commons-provider/*` en los módulos). Anotar por endpoint qué provider(s) invoca (puede ser ninguno, uno o varios).

URLs y credenciales de providers: `task_definition_testing.json`. URL base propia del BFF por ambiente: buscar `task-definition/task-definition_{local|testing|production}.json`; si no existe, preguntar al usuario.

## Paso 2: Confirmar alcance

Mostrar los módulos/endpoints detectados y preguntar si se genera la colección completa o un subconjunto. Por default, proponé generar todo el BFF (a diferencia de `generate-external-tests`, acá el objetivo es documentación completa, no un test puntual).

## Paso 3: Archivos a generar

```
<bff-name>.postman_collection.json
<bff-name>-local.postman_environment.json
<bff-name>-uat.postman_environment.json
<bff-name>-prod.postman_environment.json
```

## Paso 4: Estructura interna de la colección

```
<BFF Name>
├── <Módulo A>
│   ├── GET /moduloA/recurso
│   └── POST /moduloA/recurso
├── <Módulo B>/...
└── Legacy Providers
    ├── <provider1>/<operación>
    └── <provider2>/<operación>
```

- Cada request de un endpoint propio incluye en su `description` a qué provider(s) llama, citando la carpeta exacta dentro de "Legacy Providers" (ej: "Llama a: Legacy Providers > ice > consultar-deuda-linea"). Es el requisito clave: navegar directo al provider real ante un error.
- Cada request de "Legacy Providers" apunta directo a la URL del provider. Si requiere token, aplicar el pre-request script de `generate-external-tests` (Paso 9).
- Variables de path/query/body: mismo criterio que `generate-external-tests` (Paso 7).

## Paso 5: Environments

Uno por ambiente (`local`, `uat`, `prod`) con `base_url` del BFF y credenciales de providers si aplica. El resto de las variables van a nivel de colección.

## Finalización

Al terminar, listar módulos/endpoints incluidos y providers referenciados. No ofrecer regenerar ni preguntar por otro alcance.

## Restricciones

- No generar escenarios exhaustivos de éxito/error/borde para los providers acá — esa profundidad es responsabilidad de `generate-external-tests`; esta colección es para navegación y debugging rápido.
- No hardcodear credenciales: siempre vía environment.
- No generar archivos fuera de los especificados en el Paso 3.
