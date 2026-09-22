# /spec-finish

Cierra una spec implementada: audita los cambios y deja listo el mensaje de commit. Nunca commitea.

## Uso

```bash
/spec-finish 03-niveles-y-highscores   # o solo 03, o solo el slug
/spec-finish                           # infiere la spec de la rama spec-NN-slug actual
```

## Qué hace

1. **Valida el estado:** solo acepta `Implementado`. Con `Implementado con observaciones` se detiene y te deriva a `/spec-impl`.
2. **Detecta el camino:** con rama (`spec-NN-slug` contra la principal) o sin rama (cambios sin commitear en la rama actual).
3. **Audita** con la metodología de [`/spec-pre-commit`](../spec-pre-commit/) (4R + higiene de commit). Si el veredicto es `NO COMMITEAR`, se detiene.
4. **Propone el mensaje de commit:** objetivo de la spec + criterios cumplidos + observaciones aceptadas (si las hay).
5. **Prepara el cierre:**
   - Con rama: `git merge --squash` en la principal (queda staged, sin commit).
   - Sin rama: no toca git; te da la lista exacta de archivos para `git add`.

## Qué hacés vos después

- `git add` (solo sin rama), revisar `git diff --staged` y `git commit`.
- Borrar la rama `spec-NN-slug` si se usó.
- Marcar la spec como `Publicado` cuando corresponda (ej. después de un deploy).

## Reglas clave

- Nunca ejecuta `git commit` ni borra ramas.
- Nunca modifica el estado de la spec.
- Nunca avanza con un veredicto `NO COMMITEAR`.

Requiere una spec que [`/spec-impl`](../spec-impl/) haya dejado en `Implementado`.
