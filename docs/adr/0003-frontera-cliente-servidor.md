# ADR 0003: Frontera cliente/servidor — FastAPI como única fuente de verdad, Next.js como BFF

## Estado

Aceptado

## Contexto

El stack declarado en `AGENTS.md` separa frontend (Next.js 15) y backend
(FastAPI). Hace falta decidir quién accede a PostgreSQL y cómo se
comunican Next.js, FastAPI y n8n, evitando duplicar lógica de negocio
(validaciones, reglas de categorías, cálculo de períodos) en dos
lenguajes.

## Decisión

- FastAPI es la única capa que accede a PostgreSQL y concentra toda la
  lógica de negocio (autenticación, CRUD de registros, categorías,
  estadísticas, endpoint que consume n8n para registros automáticos de
  Telegram).
- Next.js nunca llama a FastAPI directamente desde el browser ni desde
  Server Components: usa sus propios Route Handlers (`app/api/*`) como
  capa BFF (Backend-for-Frontend). Estos Route Handlers reenvían la
  petición a FastAPI, manejan la cookie de sesión, y son el único punto
  desde donde se conoce la URL interna del backend.
- n8n llama a un endpoint dedicado de FastAPI (autenticado con API key
  estática) para crear registros automáticos clasificados desde
  Telegram.

## Consecuencias

- Una sola fuente de verdad para reglas de negocio (Python/FastAPI);
  Next.js queda como capa de presentación + intermediario de sesión.
- La URL y los detalles de FastAPI nunca se exponen al navegador, lo
  que facilita mover o proteger el backend en el VPS sin tocar el
  cliente.
- Costo: una capa extra de indirección (Route Handlers) que hay que
  mantener sincronizada con los endpoints de FastAPI; cada request de
  mutación pasa por dos saltos de red (browser → Next → FastAPI).
- El endpoint usado por n8n requiere su propio mecanismo de
  autenticación (API key), separado del de sesiones de usuario.
