# /spec-impl

Implementa una spec aprobada de punta a punta y la verifica contra sus criterios de aceptación.

## Uso

```bash
/spec-impl 03-niveles-y-highscores   # o solo 03, o solo el slug
```

## Qué hace

1. **Valida el estado:** `Aprobado` → implementa; `Implementado con observaciones` → retoma; cualquier otro → se detiene sin tocar nada.
2. **Rama:** por defecto trabaja en la rama principal (si estás en otra, pregunta). Con `AutoCreateBranch: true` en `specs/.spec-config.yml` usa `spec-NN-slug`.
3. **Confirma una sola vez** cómo va a correr, y después implementa todos los pasos seguidos, un fork por paso. Solo para ante una ambigüedad o un fallo que no cede.
4. **Verifica** todos los criterios de aceptación al final.
5. **Cierra:** `Implementado` si todo pasa; si no, `Implementado con observaciones` con los pendientes en `## Observaciones`.

Al volver a correrla sobre una spec con observaciones, ofrece **resolverlas** (corrige y re-verifica) u **omitirlas** (quedan como `[aceptada]` y la spec pasa a `Implementado`).

## Reglas clave

- Nunca commitea.
- Nunca toca la Base de Datos.
- Nunca escribe `Publicado`: es manual.
- En la spec solo toca el estado y `## Observaciones`.
- Roadmap opcional: solo sincroniza un ítem vinculado por ruta exacta (`En progreso` → `Hecho`).

Siguiente paso cuando queda `Implementado`: [`/spec-finish`](../spec-finish/).
