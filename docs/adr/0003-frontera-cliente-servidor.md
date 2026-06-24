# ADR 0003: Frontera cliente/servidor — FastAPI como única puerta de escritura

## Estado

Aceptado

## Contexto

Existen dos caminos de escritura de registros financieros: el CRUD
manual desde la web (#8-13) y el registro automático vía Telegram
clasificado por n8n (#16). Ambos deben cumplir las mismas reglas de
validación (monto mínimo, categoría por defecto, descripción por
defecto — #9, #10, #13). La spec también exige detectar caída de
servicio y avisar su recuperación (#14, #15).

## Decisión

- FastAPI es la única capa con credenciales de escritura a Postgres.
- n8n nunca escribe directo a Postgres: llama a un endpoint dedicado
  de FastAPI (ej. `POST /records`), autenticado con una API key
  estática propia del canal de Telegram, distinta de la sesión de
  usuario web.
- FastAPI valida el `user_id` de Telegram contra el usuario vinculado,
  aplica las mismas reglas de validación que el CRUD web, y escribe el
  registro.
- El manejo de caída de servicio (#14, #15) se implementa como un
  health-check de n8n contra el endpoint de FastAPI, sin que n8n
  necesite conocer el estado de Postgres directamente.

## Consecuencias

- Una sola fuente de verdad para las reglas de negocio; evita
  duplicación y divergencia entre el flujo web y el flujo de Telegram.
- Agrega un salto de red (n8n → FastAPI) en el flujo de Telegram,
  marginal dentro de la red Docker interna.
- Reduce la superficie de credenciales con acceso a Postgres a un solo
  componente (FastAPI), facilitando auditoría y rotación de secretos.
- El endpoint usado por n8n requiere su propio mecanismo de
  autenticación (API key), separado del de sesiones de usuario.
