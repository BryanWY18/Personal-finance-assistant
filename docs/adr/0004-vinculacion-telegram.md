# ADR 0004: Vinculación de Telegram — deep link con código de un solo uso

## Estado

Aceptado

## Contexto

El criterio #6 exige que un usuario autenticado entre a la sección de
vinculación en la web, luego contacte al bot con `/start` y que el proceso
de verificación asocie su `telegram_user_id` a su cuenta. El ADR 0001 ya
define la entidad `TelegramLinkCode`, pero no estaba decidido el mecanismo
de handshake ni quién genera el código.

## Decisión

- Al entrar a la sección de vinculación, FastAPI genera un código de un
  solo uso (`TelegramLinkCode`: `code`, `user_id`, `expires_at`, `used_at`)
  con TTL corto (10 minutos).
- La web muestra un botón/link de deep link de Telegram:
  `https://t.me/<bot_username>?start=<code>`. Al tocarlo, Telegram abre el
  chat con el bot y envía `/start <code>` automáticamente — sin tipeo
  manual del usuario.
- n8n recibe el mensaje `/start <code>`, llama al endpoint dedicado de
  FastAPI (mismo patrón de autenticación por API key del ADR 0003) con el
  código y el `telegram_user_id` del remitente.
- FastAPI valida: código existe, no usado, no expirado. Si es válido,
  asocia `telegram_user_id` al `user_id` del código y marca `used_at`. Si
  no es válido (expirado, ya usado, inexistente), responde error y el bot
  le indica al usuario que vuelva a la web a generar un nuevo link.
- Un código usado o expirado no es reutilizable; el usuario debe generar
  uno nuevo desde la web (sin límite de reintentos en este alcance).

## Consecuencias

- Cero fricción de tipeo para el usuario: un solo tap desde la web abre
  Telegram con el código ya cargado.
- El acoplamiento entre code y `user_id` vive enteramente en
  `TelegramLinkCode`, sin que la web necesite comunicarse directamente con
  Telegram.
- Requiere que la web conozca el `bot_username` público para construir el
  deep link (config, no secreto).
- Si el usuario abre el link desde un dispositivo sin Telegram instalado o
  con otra cuenta de Telegram activa, el flujo falla en el lado de
  Telegram, no en el backend — fuera de control del sistema, comportamiento
  esperado.
