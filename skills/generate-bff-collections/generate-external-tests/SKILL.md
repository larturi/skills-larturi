---
name: generate-external-tests
description: Genera colecciones de Postman para probar servicios externos (dependencias) de un repositorio directamente, sin pasar por el servicio bajo prueba. Orquesta el flujo completo. Usar cuando se pida crear o generar pruebas/colecciones de Postman para servicios externos, dependencias externas, providers o clients. Detecta el tipo de repositorio (microservicio de autogestión, monorepo de ecommerce o landing) y delega el descubrimiento de servicios al skill discover-external-services correspondiente.
---

# Generación de Pruebas de Dependencias Externas con Postman

## Objetivo

Crear colecciones de Postman que prueben servicios externos directamente, sin pasar por el servicio bajo prueba. Las colecciones se basan en el análisis del código para identificar endpoints, métodos HTTP, headers, payloads y respuestas esperadas de cada servicio externo consumido.

Esta skill es el orquestador del flujo. El descubrimiento de servicios externos (qué consume el repo y dónde está definido) es específico de cada tipo de repositorio y vive en skills separados.

## Flujo de trabajo

### Paso 0: Detectar el tipo de repositorio

Antes de comenzar, identificá en qué tipo de repositorio estás trabajando, porque de eso depende cómo se descubren los servicios externos:

- **Microservicio de autogestión (ag)** → usar el skill `discover-external-services-ag`
- **Monorepo de ecommerce** → usar el skill `discover-external-services-ecommerce`
- **Repo de una landing** → usar el skill `discover-external-services-landings`

Para detectarlo, revisá la estructura del proyecto (organización de carpetas, archivos de build, tecnología, manifiestos de despliegue).

**Si no podés identificar el tipo de repositorio con confianza, preguntale al usuario cuál de los tres es antes de continuar.**

### Paso 1: Descubrir los servicios externos (skill delegado)

Aplicá las instrucciones del skill de descubrimiento correspondiente al tipo de repositorio detectado en el Paso 0:

- `discover-external-services-ag`
- `discover-external-services-ecommerce`
- `discover-external-services-landings`

**Si el skill de descubrimiento necesario no está instalado en este repositorio, advertí al usuario que debe instalarlo y detené el flujo hasta que esté disponible.** No intentes adivinar la estructura por tu cuenta.

El skill de descubrimiento te dará:
- Los servicios externos que consume el repo (proveedor y operación).
- Las URLs de cada servicio.
- Las credenciales y dónde buscarlas.
- La estructura de payloads.

### Paso 2: Seleccionar endpoints a probar

Con los servicios descubiertos, mostrá al usuario los endpoints de cada servicio externo que el repo consume, indicando proveedor y operación.

Preguntá para cuál de ellos querés que se genere la colección de Postman. **Solo generá la colección para el endpoint seleccionado, no para todos los identificados.**

### Paso 3: Crear estructura de carpetas

```
external-tests/
├── <proveedor1>/
├── <proveedor2>/
└── ...
```

### Paso 4: Crear una colección por endpoint

**Convención de nombres:**
```
external-tests/
├── <proveedor1>/
│   ├── <proveedor1>-<servicio1>-<operacionA>.postman_collection.json
│   ├── <proveedor1>-<servicio1>-<operacionB>.postman_collection.json
│   ├── <proveedor1>-<servicio2>-<operacionC>.postman_collection.json
│   ├── <proveedor1>-<servicio2>-<operacionC>.md
│   └── <proveedor1>-secrets.json
└── ...
```

**Ejemplo:**
```
external-tests/
├── ice/
│   ├── ice-consultar-deuda-linea.postman_collection.json
│   ├── ice-consultar-deuda-linea.md
│   └── ice-secrets.json
└── payment/
    ├── payment-procesar-pago.postman_collection.json
    ├── payment-procesar-pago.md
    └── payment-secrets.json
```

**Nota:** Cada colección contiene TODOS los escenarios (exitosos, fallidos, casos borde).

### Paso 5: Definir escenarios de prueba

Revisá todos los payloads y respuestas esperadas del endpoint, usando como referencia las pruebas automatizadas del repo y la gestión de los payloads en el código (DTOs, builders, adapters).

Para cada endpoint, definí al menos 3 escenarios:
- **Casos exitosos**: request con datos válidos que debería retornar respuesta exitosa (200 o similar).
- **Casos fallidos**: request con datos inválidos que debería retornar error (400, 401, 403, 404, 500, etc.).
- **Casos borde**: request con datos límite o condiciones especiales que podrían causar comportamientos inesperados (campos vacíos, valores máximos, etc.).

### Paso 6: Organizar escenarios dentro de la colección

Organizá los escenarios en carpetas dentro de cada colección:

- **Carpetas con escenarios exitosos**: requests con datos válidos y tests que validan respuesta exitosa.
- **Carpetas con escenarios fallidos**: requests con datos inválidos y tests que validan manejo de errores.
- **Carpetas con otros casos borde (opcional)**: requests con datos límite y tests que validan comportamiento en bordes.

### Paso 7: Parámetros como variables

Todos los campos del request que son variables provistas por el usuario y/o que cambian entre escenarios deben definirse como variables en la colección.

Incluye:
- Path params
- Query params
- Body params
- Headers params
- URLs de servicios externos (a nivel de la colección, no como variables de environment)

No incluye:
- Campos constantes o definidos por el sistema (`Content-Type`, `Accept`, etc.).
- Campos que son parte de la estructura del payload pero no varían entre escenarios (ej: `currency` si siempre es el mismo).
- Credenciales (se referencian desde el environment, no como variables de la colección).

**Reglas para definir variables:**
- Cada caso debe tener su propio set de variables para que los tests sean independientes entre sí.
- Cada variable debe tener nombre descriptivo, valor de ejemplo y descripción de su rol en el request.

**Convención de nombres:**
```
<nombreDelEscenario>_<nombreDelCampo>
```

### Paso 8: Agregar tests

Los tests deben validar:
- Estructura de la respuesta (todos los campos mapeados/leídos por el servicio bajo prueba).
- Códigos de estado HTTP.
- Manejo de errores (mensajes y códigos de error específicos).
- Usar las pruebas automatizadas del repo como referencia para cubrir los mismos escenarios.

### Paso 9: Autenticación

Usá la sección `auth` de Postman para configurar la autenticación, en lugar de incluir tokens o credenciales directamente en los headers.

**Obtención automática de tokens:**

Si el endpoint requiere un token obtenido de otro servicio (IDP, OAuth), la obtención debe estar automatizada dentro de la colección mediante un pre-request script a nivel de colección. El script debe:

1. Verificar si ya existe un token vigente (comparando el timestamp de expiración guardado como variable de colección).
2. Si no hay token o expiró, llamar al servicio de autenticación con `pm.sendRequest()`.
3. Guardar el token obtenido como variable de colección.
4. Guardar el timestamp de expiración (con margen de 5 minutos) como variable de colección.
5. Reutilizar el token en las siguientes requests hasta que expire.

La URL del servicio de autenticación se define como variable de colección (no de environment). Las credenciales (client_id, client_secret, api_keys, etc.) se definen en el environment.

**Ejemplo de estructura del pre-request script:**
```javascript
// Verificar si el token está vigente
const tokenExpiry = pm.collectionVariables.get('token_expiry');
const currentToken = pm.collectionVariables.get('token');

if (currentToken && tokenExpiry && Date.now() < parseInt(tokenExpiry)) {
    return; // Token vigente, reutilizar
}

// Obtener nuevo token
const authUrl = pm.collectionVariables.get('auth_url');
const clientId = pm.environment.get('client_id');
const clientSecret = pm.environment.get('client_secret');

pm.sendRequest({
    url: authUrl,
    method: 'POST',
    header: { /* headers según el servicio */ },
    body: { /* body según el servicio */ }
}, function (err, res) {
    if (err || res.code !== 200) { return; }
    const token = res.json().token_field;
    const expiresIn = res.json().expires_in || 3600;
    pm.collectionVariables.set('token', token);
    pm.collectionVariables.set('token_expiry', String(Date.now() + (expiresIn - 300) * 1000));
});
```

### Paso 10: Environment de Postman

Creá un environment específico para cada proveedor, donde se definan exclusivamente las credenciales necesarias. El resto de las variables van a nivel de la colección.

**Convención de nombres:**
```
archivo: <proveedor>-secrets.json
descripción: Secrets de <proveedor>
```

### Paso 11: Documentar las pruebas generadas

Generá un archivo con el contenido para una página de Confluence que incluya: nombre del servicio, descripción, impacto del mismo e instrucciones para usar la colección.

**Convención de nombres:**
```
archivo: <proveedor1>-<servicio2>-<operacionC>.md
```

## Finalización

Una vez generada la colección, el environment y la documentación del endpoint seleccionado, el flujo termina. **No ofrezcas generar otra prueba ni preguntes si el usuario quiere continuar con otro endpoint.** La generación de cualquier otra prueba se inicia en una nueva sesión.

## Restricciones

- No generar otros archivos además de los especificados.
- Generar la colección únicamente para el endpoint seleccionado por el usuario.
- Al terminar, no ofrecer generar otra prueba; cada prueba adicional se lanza en una sesión nueva.
