# ADR 0002: Autenticación — JWT validado en middleware de Next.js

## Estado

Aceptado

## Contexto

La spec exige login con email/contraseña con reglas de complejidad
(#1), y que el logout invalide la sesión de inmediato y bloquee rutas
protegidas en el acto (#3). El stack (`AGENTS.md`) separa Next.js 15
(App Router) como frontend y FastAPI como backend.

## Decisión

- FastAPI emite, al login exitoso, un JWT firmado con id de usuario,
  email y expiración corta.
- Next.js guarda el JWT en una cookie httpOnly y usa middleware (Edge)
  para verificar firma y expiración en cada request a ruta protegida,
  sin llamar a FastAPI para esa verificación.
- FastAPI vuelve a verificar la firma del JWT en cada endpoint de su
  API.
- Para cumplir el criterio #3 (logout invalida de inmediato), la
  expiración del JWT debe ser corta y debe existir un mecanismo de
  invalidación (denylist mínima o refresh de corta vida); el diseño
  concreto de ese mecanismo es detalle de implementación, pero la
  arquitectura exige que exista.

## Consecuencias

- Valida sesión en el edge sin round-trip a FastAPI en cada
  navegación protegida, reduciendo latencia percibida.
- Requiere mantener sincronizada la clave de firma (o par de llaves)
  entre Next.js y FastAPI.
- El logout inmediato no es gratis con JWT puro: exige expiración
  corta y/o lista de revocación, lo que reintroduce parte del estado
  que un JWT busca evitar — aceptado como costo de esta decisión.
- Mayor curva de aprendizaje inicial (firma, expiración, revocación)
  frente a una sesión server-side simple.
