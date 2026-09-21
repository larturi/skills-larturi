---
name: discover-external-services-ecommerce
description: Descubre los servicios externos (dependencias) que consume el monorepo de ecommerce, analizando el código Go para identificar clients HTTP, URLs, credenciales y payloads. Usar cuando se necesite identificar y listar los endpoints de servicios externos antes de generar colecciones de Postman. Complementa al skill generate-external-tests.
---

# Descubrimiento de Servicios Externos - Monorepo de Ecommerce

## Objetivo

Analizar el monorepo de ecommerce para identificar los servicios externos que consume, sus endpoints, métodos HTTP, headers, payloads y respuestas esperadas. El resultado alimenta la generación de colecciones de Postman (skill `generate-external-tests`).

## Identificar los servicios externos

Revisá el código del repositorio para identificar:

- **Clients de infraestructura**: Archivos en `api/service/infrastructure/` que realizan llamadas HTTP a servicios externos mediante `BearerTokenAuthenticatedHttpClient`
- **URLs de servicios**: Obtenerlas desde `api/.env`, que contiene las variables de entorno con las URLs de los servicios externos (convención: `{SERVICIO}_{OPERACION}_URL`)
- **Credenciales**: Identificarlas en `api/.env` bajo las variables `{SERVICIO}_CLIENT_ID`, `{SERVICIO}_CLIENT_SECRET`, `{SERVICIO}_USERNAME`, `{SERVICIO}_PASSWORD` y `{SERVICIO}_TOKENS_URL`. Dejarlas reflejadas en el environment de Postman, referenciando dónde buscarlas.
- **Payloads**: Estructura de datos enviados (funciones `*RequestBodyFor`, `*RequestHeadersFor`, structs de request, mapas `map[string]any`)

**Ubicaciones del código relevante:**
```
api/service/infrastructure/{proveedor}/          # Clients y operaciones HTTP
api/service/infrastructure/{proveedor}/*_test.go # Tests unitarios con mocks
api/test/fixtures/.http-services/                # Cassettes go-vcr con requests/responses reales de UAT
api/test/mock/http_mock.go                       # Mock HTTP centralizado con URLs de referencia
api/.env                                         # URLs y credenciales de entornos
toolset/go/utils/BearerTokenAuthenticatedHttpClient.go  # Cliente HTTP base con OAuth2
toolset/go/testing/internal/http_recorder.go     # HTTP recorder (go-vcr) para tests de integración
```

## Proveedores externos conocidos

- `mulesoft` - Gateway API (pagos, órdenes, stock, shipping, eSIM, débito automático, etc.)
- `salesforce` - CRM (órdenes, offerings, promotions financieras)
- `dxp` - Plataforma DXP (stocks, comunicaciones, notificaciones)
- `personal_pay` - Pagos con Personal Pay
- `idp` - Identity Provider (OTP, tokens de sesión)
- `google_cloud_platform` - Consent policies
- `google_analytics` - Analytics
- `martech` - Productos similares (recomendaciones)
- `graphql` - Servicio OTT
- `aem` - Adobe Experience Manager (assets)
- `recaptcha` - Validación reCAPTCHA
- `appointment` - Turnos
- `lead_collector` - Captura de leads
- `threescale` - API management
- `mosa` - Servicio MOSA

## Listar y seleccionar endpoints

Mostrá los endpoints de cada servicio externo que el repositorio consume, indicando proveedor y operación.

Preguntá al usuario para cuál de ellos quiere que se genere la colección de Postman. Solo se generará la colección para el endpoint seleccionado, no para todos los identificados.

## Referencia para escenarios y payloads

Usá como referencia las siguientes fuentes (en orden de prioridad):

3. **Funciones `*RequestBodyFor` y `*RequestHeadersFor`** en el archivo del servicio (ej: `mulesoft/Payments.go`)
2. **Cassettes de go-vcr**: Archivos en `api/test/fixtures/.http-services/` que contienen requests y responses reales grabados contra UAT. Buscar cassettes que contengan la URL del endpoint. Verificar la fecha de última actualización con `git log -1 --format="%ai" -- <archivo>` para priorizar los más recientes.
3. **Tests unitarios** en `*_test.go` que validan la construcción de requests y el parseo de responses
4. **Mock HTTP centralizado** en `api/test/mock/http_mock.go`
5. **Manejo de status codes** en los `switch response.StatusCode` del código de producción

**⚠️ IMPORTANTE:** Los cassettes de go-vcr son la fuente preferida de datos reales. Los tests unitarios usan datos inventados para validar lógica interna y NO están garantizados en UAT.

## Continuación

Una vez identificados y seleccionados los endpoints, continuá con el skill `generate-external-tests` para producir la colección de Postman.
