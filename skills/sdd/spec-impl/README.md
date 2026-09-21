# /spec-impl

Implementa una spec aprobada, paso a paso, con pausas para revisar cada diff.

## Uso

```bash
/spec-impl 03-niveles-y-highscores
```

## Qué hace

1. **Identifica** el archivo en `specs/` (por nombre completo, número o slug).
2. **Valida** que el estado signifique `Approved` (en cualquier idioma) - si no, se detiene sin tocar código.
3. **Crea la rama** `spec-NN-slug` y se mueve a ella. Controlado por `AutoCreateBranch` en `specs/.spec-config.yml` (default `true`).
4. **Implementa** un paso del plan a la vez, mostrando el diff y esperando confirmación antes de seguir.
5. **Cierra** la spec como `Implemented` después de verificar todos los criterios de aceptación y sincroniza un ítem de roadmap solamente si ya estaba vinculado de forma exacta.

## Reglas clave

- Nunca commitea automáticamente - eso lo decide el humano.
- Si encuentra una ambigüedad que la spec no resuelve, para y pregunta.
- No exige roadmap: los planes completos, pausados o sin vínculo exacto quedan intactos.
- El historial del roadmap se reserva para cambios estructurales; las transiciones `En progreso` y `Hecho` no agregan entradas.

Requiere una spec generada por [`/spec-init`](../spec-init/) con estado `Approved`.
