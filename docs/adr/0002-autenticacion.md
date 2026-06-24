# ADR 0002: Autenticación — sesiones server-side

## Estado

Aceptado

## Contexto

El criterio #3 exige que el logout invalide la sesión de inmediato y que
las rutas protegidas dejen de ser accesibles en el acto. Un JWT
stateless no puede revocarse antes de su expiración natural sin mantener
una blocklist, lo que contradice la simplicidad buscada para el alcance
del proyecto (un usuario por cuenta, sin requisitos de escala).

## Decisión

- Autenticación por email + contraseña (hash con bcrypt o argon2).
- Sesiones server-side: tabla `Session` en PostgreSQL (token opaco,
  `user_id`, `expires_at`). La cookie del navegador es httpOnly y solo
  contiene el identificador de sesión, nunca datos del usuario.
- Logout = borrar la fila de `Session`. Cualquier request posterior con
  esa cookie deja de resolver a un usuario válido.
- Sin password reset ni verificación de email en este alcance (no-goals,
  confirmado).

## Consecuencias

- Cada request autenticado implica una consulta a `Session` en
  Postgres — costo aceptable dado el volumen esperado (un usuario, bajo
  tráfico).
- Revocación de sesión es inmediata y total, cumpliendo el criterio #3
  sin lógica adicional de blocklist.
- Si el proyecto escala a mayor tráfico, esta decisión debería
  revisarse (cache de sesión, JWT con blocklist, etc.) — fuera del
  alcance actual.
