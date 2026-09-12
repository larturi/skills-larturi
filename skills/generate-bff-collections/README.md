# Grupo `generate-bff-collections`

Genera colecciones de Postman a partir del análisis del código, para dos necesidades relacionadas:

- **Documentar un BFF NestJS completo** (`generate-bff-collection`): todos sus endpoints propios agrupados por módulo, con referencia cruzada al provider externo que llama cada uno. Es la skill principal del grupo, pensada para pararte sobre un BFF propio y exportar toda su API de una.
- **Probar un servicio externo puntual** que consume un repo, directamente contra el proveedor, sin pasar por el servicio bajo prueba (`generate-external-tests` + su discovery correspondiente).

Las cinco skills están agrupadas porque colaboran en (o reutilizan) un mismo flujo de descubrimiento de dependencias externas.

## Skills del grupo

| Skill | Rol |
|-------|-----|
| `generate-bff-collection` | **Skill principal.** Documenta la API completa de un BFF NestJS: todos los endpoints propios por módulo, más una carpeta "Legacy Providers" con los servicios externos que llama cada uno (reutiliza la lógica de descubrimiento de `discover-external-services-ag`). |
| `generate-external-tests` | Orquestador del flujo de testing puntual. Detecta el tipo de repositorio, delega el descubrimiento, y genera la colección, el environment y la documentación de un endpoint externo elegido por el usuario. |
| `discover-external-services-ag` | Descubrimiento para **microservicios de autogestión**: providers/clients, URLs y secrets desde el `task-definition_testing.json` (ECS/Fargate). |
| `discover-external-services-ecommerce` | Descubrimiento para el **monorepo de ecommerce** (Go): clients en `api/service/infrastructure/`, URLs y credenciales en `api/.env`, cassettes de go-vcr como fuente de datos reales. |
| `discover-external-services-landings` | Descubrimiento para **landings**: misma mecánica que `ag`, sobre la estructura de una landing. |

## Qué instalar

Depende de qué necesitás:

| Necesidad | Skills a instalar |
|-----------|-------------------|
| Documentar un BFF NestJS completo | `generate-bff-collection` (standalone, no necesita las otras) |
| Probar un servicio externo puntual (microservicio de autogestión) | `generate-external-tests` + `discover-external-services-ag` |
| Probar un servicio externo puntual (monorepo de ecommerce) | `generate-external-tests` + `discover-external-services-ecommerce` |
| Probar un servicio externo puntual (landing) | `generate-external-tests` + `discover-external-services-landings` |

`generate-external-tests` sin su `discover-external-services-*` correspondiente no arranca: detecta el tipo de repo, no encuentra el skill de descubrimiento y detiene el flujo.

## Flujos

### Documentar un BFF completo (`generate-bff-collection`)

1. Detecta los endpoints propios del BFF (`src/**/*.controller.ts`), agrupados por módulo.
2. Rastrea, por cada endpoint, qué provider(s) externo llama (controller → service → provider/client).
3. Te muestra los módulos/endpoints detectados y preguntás si generar la colección completa o un subconjunto (por default, todo el BFF).
4. Genera la colección con todos los endpoints propios organizados por módulo, más una carpeta "Legacy Providers" con los servicios externos referenciados.

A diferencia de `generate-external-tests`, esto es documentación completa del BFF, no un test exhaustivo de un provider puntual: los escenarios de éxito/error/borde de cada provider siguen siendo responsabilidad de `generate-external-tests`.

**Qué produce en el repo destino:**

```
<bff-name>.postman_collection.json
<bff-name>-local.postman_environment.json
<bff-name>-uat.postman_environment.json
<bff-name>-prod.postman_environment.json
```

### Probar un servicio externo puntual (`generate-external-tests`)

1. Detecta el tipo de repo (autogestión / ecommerce / landing). Si no puede con confianza, pregunta.
2. Delega en el `discover-external-services-*` correspondiente, que analiza el código y lista los servicios externos con proveedor, operación, URLs, credenciales (dónde buscarlas) y estructura de payloads.
3. Te muestra los endpoints encontrados y elegís **uno**.
4. Genera, solo para ese endpoint, la colección con todos los escenarios (exitosos, fallidos y de borde), el environment con las credenciales y la documentación.

Cada prueba adicional se pide en una sesión nueva: al terminar, el flujo no ofrece seguir con otro endpoint.

**Qué produce en el repo destino:**

```
external-tests/
└── <proveedor>/
    ├── <proveedor>-<servicio>-<operacion>.postman_collection.json   # escenarios ok / error / borde
    ├── <proveedor>-<servicio>-<operacion>.md                        # doc para Confluence
    └── <proveedor>-secrets.json                                     # environment: solo credenciales
```

## Convenciones comunes

- **Variables de colección** para todo lo que cambia entre escenarios (path, query, body y headers), con el nombre `<nombreDelEscenario>_<nombreDelCampo>` y un set propio por caso, para que los tests sean independientes.
- **Environment solo para credenciales**; las URLs de los servicios van como variables de la colección.
- **Autenticación** por la sección `auth` de Postman. Si el token se obtiene de otro servicio (IDP/OAuth), la colección lo resuelve sola con un pre-request script que cachea el token y lo renueva al vencer.
- **Fuente de los datos**: los payloads y respuestas salen del código del repo (DTOs, builders, adapters) y de sus pruebas automatizadas. En ecommerce, los cassettes de go-vcr tienen prioridad por ser tráfico real contra UAT.

## Contribuir

Para agregar una skill nueva o modificar una existente en este grupo, ver [docs/agregar-o-modificar-una-skill.md](docs/agregar-o-modificar-una-skill.md).
