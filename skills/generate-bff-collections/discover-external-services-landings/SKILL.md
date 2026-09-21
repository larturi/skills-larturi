---
name: discover-external-services-landings
description: Descubre los servicios externos (dependencias) que consume un repositorio de una landing, analizando el código para identificar providers/clients, URLs, credenciales y payloads. Usar cuando se trabaje sobre un repo de una landing y se necesite identificar y listar los endpoints de servicios externos antes de generar colecciones de Postman. Complementa al skill generate-external-tests.
---

# Descubrimiento de Servicios Externos - Landing

## Objetivo

Analizar un repositorio de una landing para identificar los servicios externos que consume, sus endpoints, métodos HTTP, headers, payloads y respuestas esperadas. El resultado alimenta la generación de colecciones de Postman (skill `generate-external-tests`).

## Identificar los servicios externos

Revisar el código del repositorio para identificar:

- **Providers/Clients**: archivos que realizan llamadas HTTP/SOAP a servicios externos.
- **URLs de servicios**: la arquitectura actual despliega sobre EKS y las variables de entorno se gestionan en Vault, no en un archivo versionado en el repo. Identificar en el código el nombre de la variable de entorno referenciada y pedirle el valor real al usuario si no es deducible del código.
- **Credenciales**: mismo caso - identificar en el código qué variable de entorno se usa para la credencial (token, API key, etc.) y pedirle al usuario el valor o dónde consultarlo en Vault, sin asumir un mecanismo de secrets específico.
- **Payloads**: estructura de datos enviados (DTOs, builders, adapters).

**Ubicaciones comunes:**
```
src/**/providers/
src/**/clients/
src/**/services/
```

## Listar y seleccionar endpoints

Mostrá los endpoints de cada servicio externo que el repositorio consume, indicando proveedor y operación.

Preguntá al usuario para cuál de ellos quiere que se genere la colección de Postman. Solo se generará la colección para el endpoint seleccionado, no para todos los identificados.

## Referencia para escenarios y payloads

Usá como referencia las pruebas automatizadas desarrolladas en el repositorio y la gestión de los payloads en el código (DTOs, builders, adapters) para definir los payloads, respuestas esperadas y escenarios de cada endpoint.

## Continuación

Una vez identificados y seleccionados los endpoints, continuá con el skill `generate-external-tests` para producir la colección de Postman.
