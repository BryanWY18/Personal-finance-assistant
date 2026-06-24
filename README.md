# Personal Finance Assistant

Aplicación web con chatbot de Telegram integrado que permite a un usuario registrar, categorizar y visualizar sus gastos e ingresos personales desde cualquier dispositivo.

> 📄 Estado del proyecto: en fase de especificación. Ver [`spec.md`](./spec.md) para los criterios de aceptación completos.

---

## Qué hace

- Autenticación por email y contraseña (registro, login, logout).
- Vinculación de la cuenta web con un usuario de Telegram para registrar gastos por chat.
- CRUD manual de registros financieros (categoría, descripción, monto) desde la web.
- Registro automático de gastos enviando un mensaje al bot de Telegram, clasificado mediante un workflow de n8n.
- Atajos de chatbot (`/comida`, `/transporte`, `/vivienda`, `/entretenimiento`, `/ingresos`, `/sin-categoria`) para registrar gastos frecuentes sin escribir texto largo.
- Categorías fijas del sistema más categorías personalizadas por usuario.
- Vista de estadísticas con gastos e ingresos separados, balance neto, y selector de período quincenal o mensual.

## Qué no hace (por diseño)

- No usa autenticación con proveedores externos (Google, GitHub, etc.).
- No permite registrar gastos con fecha u hora retroactiva.
- No mantiene historial ni auditoría de ediciones.
- No admite múltiples categorías por registro.
- No soporta múltiples usuarios por cuenta ni cuentas compartidas.
- No incluye presupuestos, metas de ahorro ni alertas de gasto.
- No importa extractos bancarios ni se conecta a APIs de bancos.
- No envía notificaciones por canales fuera de Telegram.
- No tiene aplicación móvil nativa.
- No exporta datos (CSV, PDF, etc.).

La especificación completa, incluyendo los 25 criterios de aceptación en formato Given-When-Then, está en [`spec.md`](./spec.md).

## Stack

- Next.js
- Python
- PostgreSQL
- n8n

## Licencia

[MIT](./LICENSE)
