# CONTINUAR.md — Cómo sostener el build

Requisito previo: todas las casillas de `BUILD-READY.md` en sí.

## Primer comando

```
git checkout develop && git pull && git checkout -b feat/tarea-01-nextjs-shadcn
```

## Loop por tarea (repetir para cada tarea de `plan.md`, en orden)

1. `git checkout develop && git pull`
2. `git checkout -b feat/tarea-NN-nombre-corto` (una tarea = una rama).
3. Invocar a `implementer` con el número de tarea; lee `AGENTS.md`,
   `spec.md`, los ADR citados, `docs/diseño.md` y `CONTEXT.md`.
4. `implementer` implementa y propone el commit (no lo ejecuta).
5. Verificar a mano el **criterio de hecho** exacto de la tarea.
6. Invocar a `reviewer` sobre el diff propuesto.
7. Si `reviewer` no marca nada bloqueante: ejecutar el commit
   (`tipo: descripción corta`) — una tarea = un commit.
8. `git checkout develop && git merge --no-ff feat/tarea-NN-... &&
   git branch -d feat/tarea-NN-...`.
9. Si la tarea tiene prueba manual asociada en
   `docs/pruebas-manuales.md`, ejecutarla y actualizar su estado.

## Si `implementer` se atora 2 veces en la misma tarea

No hay tercer intento automatizado. Edición manual: vos resolvés la
tarea a mano, y antes de seguir documentás en `CONTEXT.md` (fecha,
tarea, qué intentó `implementer`, qué se hizo a mano, por qué). Después
seguís el loop normal desde el paso 5 (verificar criterio de hecho).

## `release/` y `hotfix/`

- `release/`: se crea desde `develop` solo al preparar un despliegue al
  VPS propio (contenedores Docker, según `AGENTS.md`). Congela lo que
  se publica, se valida ahí, se mergea a `main` (y de vuelta a
  `develop`), y se tagea. No se agregan tareas nuevas dentro de una
  rama `release/`.
- `hotfix/`: se crea desde `main` solo para un bug encontrado en
  producción después de un release. Se mergea a `main` (nuevo tag) y a
  `develop`, para que el fix no se pierda en la siguiente release.
