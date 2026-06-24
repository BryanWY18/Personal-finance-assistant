# Personal Finance Assistant — Spec

## Objetivo

Aplicación web con chatbot de Telegram integrado que permite a un usuario registrar, categorizar y visualizar sus gastos e ingresos personales desde cualquier dispositivo.

---

## Scope

### Qué SÍ entra en este proyecto

- Autenticación por email y contraseña (registro, login, logout)
- Vinculación id de usuario de Telegram con el chatbot
- CRUD manual de registros financieros desde la web (categoría, descripción, monto)
- Registro automático de gastos vía mensaje de Telegram procesado por n8n
- Atajos (hábitos) en el chatbot para registrar gastos frecuentes sin tipear texto largo
- Clasificación automática de mensajes en categorías; 
- Categorías fijas por defecto más categorías personalizadas creadas por el usuario
- Vista de estadísticas que muestra gastos e ingresos por separado, con balance neto
- Selector de período en estadísticas: modo quincenal y modo mensual
- Borrado físico permanente de registros
- Edición de registros que modifica el original sin auditoría del valor anterior

### Qué NO entra (no-goals explícitos)

- Autenticación con proveedores externos (Google, GitHub, etc.)
- Registro de gastos con fecha u hora retroactiva
- Historial de cambios o auditoría de ediciones
- Múltiples categorías por registro
- Soporte para múltiples usuarios en una misma cuenta o cuentas compartidas
- Presupuestos, metas de ahorro o alertas de límite de gasto
- Importación de extractos bancarios o conexión con APIs bancarias
- Notificaciones push, email o cualquier canal fuera de Telegram
- Aplicación móvil nativa
- Exportación de datos (CSV, PDF, etc.)

---

## Criterios de aceptación

### Autenticación

1. Dado que un visitante no registrado accede al formulario de registro, cuando completa email y contraseña válidos y confirma, entonces se crea su cuenta y queda autenticado en la sesión activa, la contraseña en mínimo de 6 caracteres, necesitando al menos 1 mayúscula, un número y un carácter especial.
2. Dado que un usuario registrado accede al formulario de login, cuando ingresa sus credenciales correctas, entonces accede a su dashboard personal.
3. Dado que un usuario autenticado ejecuta la acción de logout, entonces su sesión se invalida y es redirigido a la pantalla de login; rutas protegidas dejan de ser accesibles.
4. Dado que un visitante intenta registrarse con un email ya existente, entonces recibe un mensaje de error y no se crea una cuenta duplicada.
5. Dado que un usuario ingresa credenciales incorrectas en el login, entonces recibe un mensaje de error y no obtiene acceso.

### Vinculación de Telegram

6. Dado que un usuario autenticado accede a la sección de vinculación de telegram, entra en contacto con el chatbot que inicia interacción con /start, se completa el proceso de verificación y su cuenta queda asociada a ese user_id y el bot puede identificarlo en conversaciones futuras.
7. Dado que un usuario no ha vinculado su Telegram, cuando envía un mensaje al bot, entonces el bot le indica que debe vincular su cuenta antes de registrar gastos, incluyendo un link a la sección de vinculación de la web.

### CRUD de registros (web)

8. Dado que un usuario autenticado accede al formulario de nuevo registro, cuando ingresa categoría, descripción y monto, y guarda, entonces el registro aparece en su listado con los datos ingresados.
9. Dado que un usuario autenticado accede al formulario de nuevo registro cuando no hay descripción, esta queda con texto "Sin descripción" y se guarda.
10. Dado que un usuario autenticado accede al formulario de nuevo registro, si no hay monto, el registro de anula y marca error "Se debe ingresar monto para poder hacer registro", no puede existir registro si monto, el monto debe ser mínimo $1, no negativos, se permiten decimales.
11. Dado que un usuario accede a un registro existente y modifica cualquiera de sus campos, cuando guarda los cambios, entonces el registro refleja los nuevos valores; los valores anteriores no quedan almacenados.
12. Dado que un usuario selecciona un registro y confirma su borrado, entonces el registro desaparece del listado y no puede recuperarse.
13. Dado que se envía una petición de creación de registro sin categoría al servidor, entonces el servidor guarda la petición con categoría "Sin categoría".
14. Dado que el servicio está caído (Telegram o n8n) no se puede guardar el registro hasta que vuelva a estar en servicio, mostrando un mensaje de error en el chatbot donde solicita intentar el registro más tarde.
15. Dado que el servicio estaba caído y se volvió a levantar, el chatbot envía un mensaje donde aclara que ya está disponible el servicio.

### Registro automático vía Telegram

16. Dado que un usuario vinculado envía un mensaje de gasto al bot, cuando n8n clasifica el mensaje (todo campo debidamente completado), entonces el bot responde con un resumen del registro (categoría, descripción, monto). 
17. Dado que el usario quiere ver los atajos disponibles, presiona "/" y se despligan los comandos disponibles: /comida, /transporte, /vivienda, /entretenimiento, /ingresos, /sin-categoria

### Categorías

18. Dado que un usuario autenticado accede a la gestión de categorías, entonces ve las categorías fijas del sistema disponibles para todos los usuarios. Las categorías base son: "Comida, Transporte, Vivienda, Entretenimiento y Sin-Categoría, Ingresos. Estas no son editables por el usario desde la interfaz web
19. Dado que un usuario crea una categoría personalizada con nombre único, entonces esa categoría queda disponible para ese usuario al crear o editar registros.
20. Dado que un usuario intenta crear una categoría personalizada con un nombre que ya existe (fija o propia), entonces recibe un error y no se crea un duplicado, la validación al crear es case-insensitive.
21. Dado que un usuario borra una categoría personalizada que tiene registros asociados, entonces esos registros quedan sin categoría asignada y aparecen agrupados bajo "Sin categoría" en la vista de estadísticas y la DB se actualiza, cambiando el campo categoria_id a "Sin categoría".

### Estadísticas

22. Dado que un usuario autenticado accede a la vista de estadísticas, entonces ve sus gastos agrupados por categoría para el período seleccionado.
23. Dado que el usuario selecciona modo quincenal, entonces las estadísticas muestran los datos de la quincena en curso (1–15 o 16–fin de mes según la fecha actual) con timezone del usuario.
24. Dado que el usuario selecciona modo mensual, entonces las estadísticas muestran los datos del mes calendario en curso con timezone del usuario.
25. Dado que la vista de estadísticas está activa, entonces se muestran el total de gastos, el total de ingresos y el balance neto (ingresos − gastos) para el período seleccionado, separados en dos gráficos continuos.
26. Dado que el usuario no tiene registros en el período seleccionado, entonces la vista muestra montos en cero sin errores ni datos de otros períodos.

---

## No-goals

- **Sin fecha retroactiva:** no se puede crear un registro con una fecha u hora distinta al momento del envío; todos los registros toman timestamp del servidor.
- **Sin auditoría de ediciones:** editar un registro sobreescribe el original; no existe historial de valores anteriores ni log de cambios.
- **Sin borrado reversible:** el borrado es físico e inmediato; no hay papelera, confirmación de segunda instancia ni período de gracia.
- **Sin múltiples categorías por registro:** cada registro pertenece a exactamente una categoría; no se admiten etiquetas múltiples ni jerarquía de subcategorías.
- **Sin integraciones bancarias:** el sistema no importa datos de bancos, tarjetas ni ninguna fuente externa; todos los registros se ingresan manualmente o vía Telegram.
- **Sin presupuestos ni alertas:** el sistema no evalúa límites de gasto ni envía notificaciones proactivas de ningún tipo.
