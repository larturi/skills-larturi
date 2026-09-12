---
name: discover-external-services-landings
description: Descubre los servicios externos (dependencias) que consume un repositorio de una landing, analizando el código para identificar providers/clients, URLs, credenciales y payloads. Usar cuando se trabaje sobre un repo de una landing y se necesite identificar y listar los endpoints de servicios externos antes de generar colecciones de Postman. Complementa al skill generate-external-tests.
---

# Descubrimiento de Servicios Externos — Landing

## Objetivo

Analizar un repositorio de una landing para identificar los servicios externos que consume, sus endpoints, métodos HTTP, headers, payloads y respuestas esperadas. El resultado alimenta la generación de colecciones de Postman (skill `generate-external-tests`).

## Identificar los servicios externos

Revisar el código del repositorio para identificar:

- **Providers/Clients**: archivos que realizan llamadas HTTP/SOAP a servicios externos.
- **URLs de servicios**: obtenerlas desde `task_definition_testing.json`, que contiene la Task Definition para desplegar el servicio sobre Amazon ECS usando AWS Fargate. Este archivo define las variables de entorno, incluyendo las URLs de los servicios externos.
- **Credenciales**: identificarlas en la sección `secrets` del `task_definition_testing.json` y dejarlas reflejadas en el environment de Postman, referenciando dónde buscarlas en AWS Secrets Manager.
- **Payloads**: estructura de datos enviados (DTOs, builders, adapters).

**Ubicaciones comunes:**
```
src/**/providers/
src/**/clients/
src/**/services/
task-definition/task-definition_testing.json
```

## Listar y seleccionar endpoints

Mostrá los endpoints de cada servicio externo que el repositorio consume, indicando proveedor y operación.

Preguntá al usuario para cuál de ellos quiere que se genere la colección de Postman. Solo se generará la colección para el endpoint seleccionado, no para todos los identificados.

## Referencia para escenarios y payloads

Usá como referencia las pruebas automatizadas desarrolladas en el repositorio y la gestión de los payloads en el código (DTOs, builders, adapters) para definir los payloads, respuestas esperadas y escenarios de cada endpoint.

## Continuación

Una vez identificados y seleccionados los endpoints, continuá con el skill `generate-external-tests` para producir la colección de Postman.
