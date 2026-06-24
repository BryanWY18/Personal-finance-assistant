# ADR 0005: Detección de caída de servicio — reactivo por usuario

## Estado

Aceptado

## Contexto

Los criterios #14 y #15 exigen que, si el backend no puede guardar un
registro vía Telegram, el bot avise que el servicio no está disponible y
pida reintentar más tarde; y que, cuando el servicio vuelve a estar
disponible, el bot lo aclare. No estaba decidido qué componente detecta la
caída ni cómo se dispara el aviso de recuperación.

## Decisión

- Sin monitoreo activo ni health-check programado. La detección es
  reactiva: ocurre únicamente cuando un usuario vinculado envía un mensaje
  de gasto al bot.
- El workflow de n8n envuelve la llamada al endpoint de FastAPI (ADR 0003)
  en manejo de error. Si la llamada falla (timeout, 5xx, conexión
  rechazada), n8n responde en el chat: "Servicio no disponible, intentá
  más tarde" y marca un flag `last_attempt_failed = true` para ese
  `telegram_user_id` (campo en `User` o tabla simple en Postgres).
- Cuando ese mismo usuario vuelve a enviar un mensaje y la llamada a
  FastAPI ahora sí responde con éxito, el bot guarda el registro
  normalmente y agrega a la respuesta una aclaración de que el servicio ya
  está disponible. Luego limpia el flag (`last_attempt_failed = false`).
- No hay broadcast ni aviso a usuarios que no volvieron a intentar tras la
  caída: el sistema nunca les debe una notificación si ellos no
  preguntaron.

## Consecuencias

- Cero infraestructura nueva: no hay trigger programado, no hay lista de
  usuarios a notificar, no hay job de health-check que mantener.
- El aviso de "servicio disponible de nuevo" solo lo recibe quien
  efectivamente reintentó tras la caída — un usuario que no volvió a
  escribir simplemente no se entera, lo cual es consistente con el alcance
  del proyecto (sin notificaciones proactivas fuera de la interacción del
  usuario).
- Si la caída es prolongada, cada intento fallido durante ese lapso repite
  el mismo mensaje de error — comportamiento esperado, no requiere
  backoff ni deduplicación en este alcance.
