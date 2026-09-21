---
name: discover-external-services-ag
description: Descubre los servicios externos (dependencias) que consume un microservicio de autogestión (ag), analizando el código para identificar providers/clients, URLs, credenciales y payloads. Usar cuando se trabaje sobre un microservicio de autogestión y se necesite identificar y listar los endpoints de servicios externos antes de generar colecciones de Postman. Complementa al skill generate-external-tests.
---

# Descubrimiento de Servicios Externos - Microservicio de Autogestión (ag)

## Objetivo

Analizar un microservicio de autogestión para identificar los servicios externos que consume, sus endpoints, métodos HTTP, headers, payloads y respuestas esperadas. El resultado alimenta la generación de colecciones de Postman (skill `generate-external-tests`).

## Identificar los servicios externos

Hay dos categorías de providers a detectar: **locales** (código fuente en el repo) y **de librerías compartidas** (paquetes npm sin código local). Ambos deben aparecer en el listado final.

### Providers locales

Revisar el código del microservicio para identificar:

- **Providers/Clients**: archivos que realizan llamadas HTTP/SOAP a servicios externos.
- **URLs de servicios**: la arquitectura actual despliega sobre EKS y las variables de entorno se gestionan en Vault, no en un archivo versionado en el repo. Identificar en el código el nombre de la variable de entorno referenciada (ej: `process.env.PI_BUSINESS_URL`) y pedirle el valor real al usuario si no es deducible del código.
- **Credenciales**: mismo caso - identificar en el código qué variable de entorno se usa para la credencial (token, API key, etc.) y pedirle al usuario el valor o dónde consultarlo en Vault, sin asumir un mecanismo de secrets específico.
- **Payloads**: estructura de datos enviados (DTOs, builders, adapters).

**Ubicaciones comunes:**
```
src/**/providers/
src/**/clients/
src/**/services/
```

### Providers de librerías compartidas

Algunos servicios externos se consumen a través de librerías npm compartidas (ej: `@pv-commons-provider/*`) que encapsulan el provider. Estos no tienen código fuente local - se importan como dependencia y se registran en los módulos NestJS.

**Cómo detectarlos:**

1. **Buscar imports en módulos** (`src/**/*.module.ts`): líneas con patrón `from '@pv-commons-provider/*'` u otras convenciones de naming de providers compartidos. Cada import identifica un provider y el servicio externo asociado.

2. **Identificar la variable de entorno** que da la URL base del servicio (no está en un archivo del repo: se gestiona en Vault). Las variables suelen seguir una convención derivada del nombre del paquete (ej: `@pv-commons-provider/pibusiness` → `PI_BUSINESS_URL`; `@pv-commons-provider/dxp-customer-bill` → `DXP_CUSTOMER_BILL_URL`). Pedirle el valor al usuario si no puede deducirse del código.

3. **Buscar paths de endpoints** en archivos de constantes (`src/**/utils/constant.ts`, `src/**/constants.ts` o similares). Los paths suelen definirse como constantes exportadas (ej: `DXP_CUSTOMER_BILL_INVOICES_PATH = '/v2/customerBill?...'`).

4. **Buscar grabaciones en tapes** (`tapes/`): los archivos `.json5` contienen requests reales grabados contra el servicio, que proveen: path exacto, método HTTP, headers, estructura del request body y response body. Son la fuente más confiable de la interfaz real.

5. **Revisar uso en services** (`src/**/*.service.ts`): qué métodos del provider se invocan y con qué parámetros, para entender la interfaz que el microservicio utiliza.

**En el listado final, indicar que estos servicios provienen de una librería compartida** y mostrar toda la información inferida (URL, path, auth, response structure) al mismo nivel que los providers locales.

## Listar y seleccionar endpoints

Mostrá los endpoints de cada servicio externo que el microservicio consume, indicando proveedor y operación.

Preguntá al usuario para cuál de ellos quiere que se genere la colección de Postman. Solo se generará la colección para el endpoint seleccionado, no para todos los identificados.

## Referencia para escenarios y payloads

Usá como referencia las pruebas automatizadas desarrolladas en el microservicio y la gestión de los payloads en el código (DTOs, builders, adapters) para definir los payloads, respuestas esperadas y escenarios de cada endpoint.

## Continuación

Una vez identificados y seleccionados los endpoints, continuá con el skill `generate-external-tests` para producir la colección de Postman.
