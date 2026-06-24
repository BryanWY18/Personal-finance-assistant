# Plan de pruebas manuales — Personal Finance Assistant

Este documento contiene una prueba manual por cada criterio de aceptación
numerado en `spec.md`. Está pensado para ser ejecutado paso a paso por una
persona sin contexto previo del proyecto. No reemplaza tests automatizados
(el proyecto no los tiene, según `AGENTS.md`): es la única validación antes
de que un humano use el sistema en producción.

Cada bloque corresponde a un criterio de `spec.md`, en el mismo orden y con
el mismo número y texto original. Si un criterio no tiene entrada concreta,
acción concreta y resultado observable, se reporta como no verificable en
lugar de inventar datos.

---

## Autenticación

### Criterio 1

> Dado que un visitante no registrado accede al formulario de registro,
> cuando completa email y contraseña válidos y confirma, entonces se crea
> su cuenta y queda autenticado en la sesión activa, la contraseña en
> mínimo de 6 caracteres, necesitando al menos 1 mayúscula, un número y un
> carácter especial.

**Precondición:** No existe ninguna cuenta registrada con el email
`prueba.alta1@example.com`. No hay sesión activa (navegador sin cookies del
sitio o en ventana de incógnito).

**Pasos:**

1. Abrir la URL del formulario de registro de la aplicación.
2. Completar el campo "Email" con `prueba.alta1@example.com`.
3. Completar el campo "Contraseña" con `Clave1!`.
4. Confirmar la contraseña (si el formulario tiene un segundo campo) con
   `Clave1!`.
5. Presionar el botón de enviar/registrarse.

**Resultado esperado:** La aplicación redirige al dashboard personal del
usuario (no a la pantalla de login) sin pedir credenciales de nuevo. No se
muestra ningún mensaje de error de validación de contraseña, dado que
`Clave1!` tiene 7 caracteres, una mayúscula (`C`), un número (`1`) y un
carácter especial (`!`).

---

### Criterio 2

> Dado que un usuario registrado accede al formulario de login, cuando
> ingresa sus credenciales correctas, entonces accede a su dashboard
> personal.

**Precondición:** Existe una cuenta registrada con email
`prueba.alta1@example.com` y contraseña `Clave1!` (la creada en el
Criterio 1). No hay sesión activa.

**Pasos:**

1. Abrir la URL del formulario de login de la aplicación.
2. Completar el campo "Email" con `prueba.alta1@example.com`.
3. Completar el campo "Contraseña" con `Clave1!`.
4. Presionar el botón de iniciar sesión.

**Resultado esperado:** La aplicación redirige a la pantalla del dashboard
personal del usuario. La URL deja de ser la del login.

---

### Criterio 3

> Dado que un usuario autenticado ejecuta la acción de logout, entonces su
> sesión se invalida y es redirigido a la pantalla de login; rutas
> protegidas dejan de ser accesibles.

**Precondición:** El usuario `prueba.alta1@example.com` tiene una sesión
activa (sigue logueado desde el Criterio 2). Se conoce la URL de una ruta
protegida, por ejemplo la del dashboard (`/dashboard` o equivalente
mostrado en la barra de direcciones tras el login).

**Pasos:**

1. Con la sesión activa, localizar y presionar el botón/enlace de
   "Cerrar sesión" (logout).
2. Observar a qué pantalla redirige la aplicación.
3. Copiar la URL exacta del dashboard que se usaba antes del logout y
   pegarla directamente en la barra de direcciones del navegador.

**Resultado esperado:** En el paso 2, la aplicación redirige a la pantalla
de login. En el paso 3, según ADR-0002, la aplicación no muestra el
dashboard: redirige nuevamente a la pantalla de login porque el middleware
de Next.js verifica el JWT en cada request a ruta protegida y la sesión ya
fue invalidada.

---

### Criterio 4

> Dado que un visitante intenta registrarse con un email ya existente,
> entonces recibe un mensaje de error y no se crea una cuenta duplicada.

**Precondición:** Existe una cuenta registrada con email
`prueba.alta1@example.com` (creada en el Criterio 1). El visitante no tiene
sesión activa.

**Pasos:**

1. Abrir la URL del formulario de registro.
2. Completar el campo "Email" con `prueba.alta1@example.com`.
3. Completar el campo "Contraseña" con `Clave2!` (distinta a la original,
   para confirmar que el rechazo es por email duplicado y no por
   contraseña inválida).
4. Presionar el botón de enviar/registrarse.

**Resultado esperado:** La aplicación muestra un mensaje de error
indicando que el email ya está registrado. El visitante permanece en la
pantalla de registro (no es redirigido al dashboard). Al intentar luego
hacer login con `prueba.alta1@example.com` / `Clave2!` (la contraseña
nueva), el acceso es rechazado, confirmando que no se sobrescribió la
cuenta original.

---

### Criterio 5

> Dado que un usuario ingresa credenciales incorrectas en el login,
> entonces recibe un mensaje de error y no obtiene acceso.

**Precondición:** Existe una cuenta registrada con email
`prueba.alta1@example.com` y contraseña `Clave1!`. No hay sesión activa.

**Pasos:**

1. Abrir la URL del formulario de login.
2. Completar el campo "Email" con `prueba.alta1@example.com`.
3. Completar el campo "Contraseña" con `ClaveIncorrecta9!`.
4. Presionar el botón de iniciar sesión.

**Resultado esperado:** La aplicación muestra un mensaje de error de
credenciales incorrectas. La pantalla sigue siendo la de login; no se
accede al dashboard.

---

## Vinculación de Telegram

### Criterio 6

> Dado que un usuario autenticado accede a la sección de vinculación de
> telegram, entra en contacto con el chatbot que inicia interacción con
> /start, se completa el proceso de verificación y su cuenta queda
> asociada a ese user_id y el bot puede identificarlo en conversaciones
> futuras.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa (vuelve a loguearse según Criterio 2 si fue necesario). El usuario
tiene la app de Telegram instalada y una cuenta de Telegram activa, sin
vincular previamente a esta cuenta de Personal Finance Assistant.

**Pasos:**

1. Con la sesión activa, navegar a la sección de vinculación de Telegram
   en la web.
2. Presionar el botón/enlace de vinculación (deep link de Telegram,
   según ADR-0004: `https://t.me/<bot_username>?start=<code>`).
3. Confirmar en Telegram la apertura del chat con el bot cuando el
   sistema operativo pregunte con qué aplicación abrir el enlace.
4. Observar que Telegram envía automáticamente `/start <code>` al bot (no
   requiere tipear nada).
5. Esperar la respuesta del bot en el chat.
6. Volver a la pestaña de la web y recargar la sección de vinculación.
7. Enviar un nuevo mensaje de texto cualquiera al bot, por ejemplo
   `hola`.

**Resultado esperado:** En el paso 5, el bot responde confirmando que la
vinculación fue exitosa. En el paso 6, la web muestra el estado "vinculado"
(ya no ofrece generar un nuevo código de vinculación, o muestra el
identificador de Telegram asociado). En el paso 7, el bot responde
tratando al remitente como usuario ya identificado (no le vuelve a pedir
vincular su cuenta, a diferencia del comportamiento descrito en el
Criterio 7).

---

### Criterio 7

> Dado que un usuario no ha vinculado su Telegram, cuando envía un mensaje
> al bot, entonces el bot le indica que debe vincular su cuenta antes de
> registrar gastos, incluyendo un link a la sección de vinculación de la
> web.

**Precondición:** Existe una cuenta de Telegram que nunca vinculó ninguna
cuenta de Personal Finance Assistant (un `telegram_user_id` sin registro
en `TelegramLinkCode` usado, según ADR-0004).

**Pasos:**

1. Desde esa cuenta de Telegram sin vincular, abrir el chat con el bot.
2. Enviar el mensaje `Gasté $10 en comida`.

**Resultado esperado:** El bot responde indicando que la cuenta no está
vinculada y que debe completar la vinculación antes de poder registrar
gastos. La respuesta incluye un enlace a la sección de vinculación de la
web (URL clicable o visible como texto). El bot no registra ningún gasto.

---

## CRUD de registros (web)

### Criterio 8

> Dado que un usuario autenticado accede al formulario de nuevo registro,
> cuando ingresa categoría, descripción y monto, y guarda, entonces el
> registro aparece en su listado con los datos ingresados.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa. Su listado de registros está vacío o en cualquier estado (no
afecta la verificación).

**Pasos:**

1. Navegar al formulario de nuevo registro.
2. Seleccionar la categoría `Comida`.
3. Completar el campo "Descripción" con `Café`.
4. Completar el campo "Monto" con `5.50`.
5. Presionar el botón de guardar.
6. Ir al listado de registros.

**Resultado esperado:** El listado de registros muestra una fila nueva con
categoría `Comida`, descripción `Café` y monto `5.50` (o `$5.50` según el
formato de presentación de la app).

---

### Criterio 9

> Dado que un usuario autenticado accede al formulario de nuevo registro
> cuando no hay descripción, esta queda con texto "Sin descripción" y se
> guarda.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa.

**Pasos:**

1. Navegar al formulario de nuevo registro.
2. Seleccionar la categoría `Transporte`.
3. Dejar el campo "Descripción" vacío (no escribir nada).
4. Completar el campo "Monto" con `12`.
5. Presionar el botón de guardar.
6. Ir al listado de registros.

**Resultado esperado:** El listado muestra una fila nueva con categoría
`Transporte`, monto `12` y el texto literal `Sin descripción` en el campo
de descripción.

---

### Criterio 10

> Dado que un usuario autenticado accede al formulario de nuevo registro,
> si no hay monto, el registro de anula y marca error "Se debe ingresar
> monto para poder hacer registro", no puede existir registro si monto, el
> monto debe ser mínimo $1, no negativos, se permiten decimales.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa.

**Pasos (caso A — monto vacío):**

1. Navegar al formulario de nuevo registro.
2. Seleccionar la categoría `Vivienda`.
3. Completar el campo "Descripción" con `Prueba sin monto`.
4. Dejar el campo "Monto" vacío.
5. Presionar el botón de guardar.

**Resultado esperado (caso A):** La aplicación muestra el mensaje de error
`Se debe ingresar monto para poder hacer registro`. El registro no aparece
en el listado.

**Pasos (caso B — monto menor al mínimo):**

6. Repetir los pasos 1 a 3 con descripción `Prueba monto bajo`.
7. Completar el campo "Monto" con `0.50`.
8. Presionar el botón de guardar.

**Resultado esperado (caso B):** La aplicación rechaza el registro (no
permite guardar un monto menor a $1) y no aparece en el listado.

**Pasos (caso C — monto negativo):**

9. Repetir los pasos 1 a 3 con descripción `Prueba monto negativo`.
10. Completar el campo "Monto" con `-5`.
11. Presionar el botón de guardar.

**Resultado esperado (caso C):** La aplicación rechaza el registro y no
aparece en el listado.

**Pasos (caso D — monto decimal válido):**

12. Repetir los pasos 1 a 3 con descripción `Prueba monto decimal`.
13. Completar el campo "Monto" con `1.25`.
14. Presionar el botón de guardar.
15. Ir al listado de registros.

**Resultado esperado (caso D):** El listado muestra una fila nueva con
categoría `Vivienda`, descripción `Prueba monto decimal` y monto `1.25`.

---

### Criterio 11

> Dado que un usuario accede a un registro existente y modifica cualquiera
> de sus campos, cuando guarda los cambios, entonces el registro refleja
> los nuevos valores; los valores anteriores no quedan almacenados.

**Precondición:** Existe un registro guardado con categoría `Comida`,
descripción `Café` y monto `5.50` (el creado en el Criterio 8).

**Pasos:**

1. En el listado de registros, localizar el registro con descripción
   `Café` y monto `5.50`.
2. Presionar la acción de editar sobre ese registro.
3. Cambiar la descripción a `Café con leche`.
4. Cambiar el monto a `6.00`.
5. Presionar el botón de guardar cambios.
6. Ir al listado de registros y volver a abrir ese mismo registro en modo
   edición o detalle.

**Resultado esperado:** El listado muestra el registro con descripción
`Café con leche` y monto `6.00`. Al volver a abrir el registro en el paso
6, no se observa en ningún lugar de la interfaz (listado, detalle, ni
formulario de edición) el valor anterior `Café` / `5.50`; según la spec
(no-goals: sin auditoría de ediciones), no existe vista de historial
donde buscarlo.

---

### Criterio 12

> Dado que un usuario selecciona un registro y confirma su borrado,
> entonces el registro desaparece del listado y no puede recuperarse.

**Precondición:** Existe un registro guardado con descripción
`Café con leche` y monto `6.00` (el editado en el Criterio 11).

**Pasos:**

1. En el listado de registros, localizar el registro con descripción
   `Café con leche`.
2. Presionar la acción de borrar sobre ese registro.
3. Confirmar el borrado si la interfaz pide confirmación.
4. Recargar la página del listado de registros (F5 o equivalente).

**Resultado esperado:** Tras el paso 3, el registro desaparece
inmediatamente del listado. Tras recargar la página (paso 4), el registro
sigue sin aparecer. No existe en la interfaz ninguna opción de
"papelera", "deshacer" o "restaurar" para recuperarlo (consistente con el
no-goal "sin borrado reversible" de `spec.md`).

---

### Criterio 13

> Dado que se envía una petición de creación de registro sin categoría al
> servidor, entonces el servidor guarda la petición con categoría "Sin
> categoría".

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa y conoce o puede obtener su token/cookie de sesión para autenticar
una petición directa al servidor (por ejemplo, mediante las herramientas
de desarrollador del navegador para copiar la cookie de sesión, o un
cliente HTTP como Postman/curl).

**Pasos:**

1. Construir manualmente una petición HTTP `POST` al endpoint de creación
   de registros del backend (ej. `POST /records`), incluyendo la
   autenticación de sesión del usuario.
2. En el cuerpo de la petición, incluir `descripcion: "Pago sin categoria"`
   y `monto: 20`, sin incluir el campo de categoría (omitirlo del JSON por
   completo, no enviarlo como vacío).
3. Enviar la petición.
4. Ir al listado de registros en la web.

**Resultado esperado:** La petición es aceptada (no rechazada por falta de
categoría). En el listado de registros aparece una fila con descripción
`Pago sin categoria`, monto `20` y categoría `Sin categoría`.

---

### Criterio 14

> Dado que el servicio está caído (Telegram o n8n) no se puede guardar el
> registro hasta que vuelva a estar en servicio, mostrando un mensaje de
> error en el chatbot donde solicita intentar el registro más tarde.

**Precondición:** El usuario de Telegram de prueba ya está vinculado
(Criterio 6 completado). Según ADR-0005, la detección de caída es
reactiva: se dispara cuando el workflow de n8n falla al llamar al endpoint
de FastAPI. Para esta prueba, el endpoint de FastAPI usado por n8n debe
estar detenido o inaccesible (por ejemplo, deteniendo el contenedor
Docker del backend o bloqueando la ruta de red entre n8n y FastAPI).

**Pasos:**

1. Detener el servicio de FastAPI (ej. `docker compose stop api` o
   equivalente según el entorno de despliegue), dejando n8n y Telegram
   operativos.
2. Desde el chat de Telegram ya vinculado, enviar el mensaje
   `Gasté $8 en comida`.
3. Observar la respuesta del bot.
4. Ir al listado de registros en la web (puede requerir loguearse con
   la cuenta vinculada a ese usuario de Telegram) y revisar si aparece
   un registro de $8 en comida.

**Resultado esperado:** Según ADR-0005, el bot responde en el chat con un
mensaje del tipo "Servicio no disponible, intentá más tarde" (mensaje de
error solicitando reintentar). En el paso 4, no aparece ningún registro
nuevo de $8 en comida: la petición no se guardó.

---

### Criterio 15

> Dado que el servicio estaba caído y se volvió a levantar, el chatbot
> envía un mensaje donde aclara que ya está disponible el servicio.

**Precondición:** Se ejecutó el Criterio 14 inmediatamente antes: el
servicio de FastAPI estuvo caído y el mismo usuario de Telegram recibió el
mensaje de "servicio no disponible" tras intentar registrar un gasto.

**Pasos:**

1. Reiniciar/levantar el servicio de FastAPI (ej.
   `docker compose start api` o equivalente).
2. Confirmar que el endpoint de FastAPI responde (ej. accediendo a un
   endpoint de salud o a la web, si corresponde).
3. Desde el mismo chat de Telegram que usó en el Criterio 14, enviar
   nuevamente el mensaje `Gasté $8 en comida`.
4. Observar la respuesta del bot.
5. Ir al listado de registros en la web.

**Resultado esperado:** Según ADR-0005, el bot guarda el registro
normalmente y, además del resumen del registro, la respuesta incluye una
aclaración de que el servicio ya está disponible de nuevo. En el paso 5,
el listado de registros muestra un nuevo registro de categoría `Comida`
y monto `8`.

---

## Registro automático vía Telegram

### Criterio 16

> Dado que un usuario vinculado envía un mensaje de gasto al bot, cuando
> n8n clasifica el mensaje (todo campo debidamente completado), entonces
> el bot responde con un resumen del registro (categoría, descripción,
> monto).

**Precondición:** El usuario de Telegram de prueba está vinculado
(Criterio 6) y el servicio (FastAPI, n8n) está operativo.

**Pasos:**

1. Desde el chat de Telegram vinculado, enviar el mensaje
   `Gasté $15 en supermercado`.
2. Observar la respuesta del bot.
3. Ir al listado de registros en la web.

**Resultado esperado:** El bot responde con un resumen del registro que
incluye explícitamente una categoría, una descripción y un monto de `15`
(o `$15`). En el listado de registros de la web aparece un nuevo registro
con monto `15` y la misma categoría y descripción mostradas en el resumen
del bot.

> Nota de alcance: `spec.md` no define las reglas exactas de clasificación
> de texto a categoría (qué frases mapean a qué categoría); esta prueba
> verifica que el bot complete y muestre el resumen con los tres campos
> exigidos por el criterio, no que la categoría asignada sea una en
> particular.

---

### Criterio 17

> Dado que el usario quiere ver los atajos disponibles, presiona "/" y se
> despligan los comandos disponibles: /comida, /transporte, /vivienda,
> /entretenimiento, /ingresos, /sin-categoria.

**Precondición:** El usuario de Telegram de prueba está vinculado
(Criterio 6).

**Pasos:**

1. Desde el chat de Telegram con el bot, escribir el carácter `/` en el
   campo de texto (sin enviarlo).
2. Observar el menú de comandos que Telegram despliega automáticamente
   sobre el campo de texto.

**Resultado esperado:** El menú desplegado lista exactamente los comandos
`/comida`, `/transporte`, `/vivienda`, `/entretenimiento`, `/ingresos` y
`/sin-categoria` (los seis atajos definidos en el criterio), cada uno con
su descripción breve visible en el menú.

---

## Categorías

### Criterio 18

> Dado que un usuario autenticado accede a la gestión de categorías,
> entonces ve las categorías fijas del sistema disponibles para todos los
> usuarios. Las categorías base son: "Comida, Transporte, Vivienda,
> Entretenimiento y Sin-Categoría, Ingresos. Estas no son editables por el
> usario desde la interfaz web.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa.

**Pasos:**

1. Navegar a la sección de gestión de categorías.
2. Observar la lista de categorías mostrada.
3. Intentar editar la categoría `Comida` (buscar y presionar cualquier
   botón de editar/modificar asociado a ella).

**Resultado esperado:** La lista incluye, al menos, las seis categorías
`Comida`, `Transporte`, `Vivienda`, `Entretenimiento`, `Sin-Categoría` e
`Ingresos`. Ninguna de estas seis tiene un control de edición disponible
(el botón de editar está ausente, deshabilitado, o al presionarlo la
interfaz no permite modificar nombre/datos de la categoría).

---

### Criterio 19

> Dado que un usuario crea una categoría personalizada con nombre único,
> entonces esa categoría queda disponible para ese usuario al crear o
> editar registros.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa. No existe ninguna categoría (fija o propia de este usuario)
llamada `Mascotas`.

**Pasos:**

1. Navegar a la sección de gestión de categorías.
2. Crear una nueva categoría con nombre `Mascotas`.
3. Confirmar la creación.
4. Navegar al formulario de nuevo registro.
5. Abrir el selector de categoría.

**Resultado esperado:** Tras el paso 3, la categoría `Mascotas` aparece en
la lista de categorías. En el paso 5, el selector de categoría del
formulario de nuevo registro incluye la opción `Mascotas` junto con las
categorías fijas.

---

### Criterio 20

> Dado que un usuario intenta crear una categoría personalizada con un
> nombre que ya existe (fija o propia), entonces recibe un error y no se
> crea un duplicado, la validación al crear es case-insensitive.

**Precondición:** Existe la categoría fija `Comida` y la categoría
personalizada `Mascotas` (creada en el Criterio 19), ambas asociadas al
usuario `prueba.alta1@example.com`, que tiene sesión activa.

**Pasos (caso A — duplicado exacto de categoría propia):**

1. Navegar a la sección de gestión de categorías.
2. Intentar crear una nueva categoría con nombre `Mascotas`.
3. Confirmar la creación.

**Resultado esperado (caso A):** La aplicación muestra un mensaje de
error indicando que la categoría ya existe. La lista de categorías sigue
teniendo una sola entrada `Mascotas`.

**Pasos (caso B — duplicado case-insensitive de categoría fija):**

4. Intentar crear una nueva categoría con nombre `comida` (minúsculas).
5. Confirmar la creación.

**Resultado esperado (caso B):** La aplicación muestra un mensaje de error
indicando que la categoría ya existe. La lista de categorías no contiene
dos entradas para "comida"/"Comida".

**Pasos (caso C — duplicado case-insensitive de categoría propia):**

6. Intentar crear una nueva categoría con nombre `MASCOTAS` (mayúsculas).
7. Confirmar la creación.

**Resultado esperado (caso C):** La aplicación muestra un mensaje de error
indicando que la categoría ya existe. La lista de categorías sigue
teniendo una sola entrada para "Mascotas".

---

### Criterio 21

> Dado que un usuario borra una categoría personalizada que tiene
> registros asociados, entonces esos registros quedan sin categoría
> asignada y aparecen agrupados bajo "Sin categoría" en la vista de
> estadísticas y la DB se actualiza, cambiando el campo categoria_id a
> "Sin categoría".

**Precondición:** Existe la categoría personalizada `Mascotas` (del
Criterio 19) asociada al usuario `prueba.alta1@example.com`. Existe al
menos un registro con esa categoría: descripción `Comida para gato`,
monto `30`, categoría `Mascotas`.

**Pasos:**

1. Si el registro descrito en la precondición no existe, crearlo desde el
   formulario de nuevo registro con categoría `Mascotas`, descripción
   `Comida para gato` y monto `30`.
2. Navegar a la sección de gestión de categorías.
3. Borrar la categoría `Mascotas`.
4. Confirmar el borrado si la interfaz lo solicita.
5. Ir al listado de registros y localizar el registro con descripción
   `Comida para gato`.
6. Ir a la vista de estadísticas, en el período que incluya la fecha
   actual.

**Resultado esperado:** Tras el borrado (paso 4), la categoría `Mascotas`
ya no aparece en la gestión de categorías. En el paso 5, el registro
`Comida para gato` / `30` sigue existiendo en el listado, pero ahora
muestra categoría `Sin categoría` (según ADR-0001, el registro pasa a
comportarse como gasto bajo esa categoría). En el paso 6, el monto `30`
aparece agrupado bajo `Sin categoría` en las estadísticas del período
correspondiente.

---

## Estadísticas

### Criterio 22

> Dado que un usuario autenticado accede a la vista de estadísticas,
> entonces ve sus gastos agrupados por categoría para el período
> seleccionado.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa y al menos dos registros de gasto en categorías distintas dentro
del período actual: uno con categoría `Comida` y monto `5.50` (del
Criterio 8, si sigue vigente) y otro con categoría `Transporte` y monto
`12` (del Criterio 9, si sigue vigente). Si esos registros ya no existen,
se vuelven a crear con esos mismos valores antes de empezar.

**Pasos:**

1. Navegar a la vista de estadísticas.
2. Observar el desglose de gastos mostrado.

**Resultado esperado:** La vista muestra los gastos agrupados por
categoría, con al menos una entrada para `Comida` (incluyendo el monto
`5.50`) y otra para `Transporte` (incluyendo el monto `12`), correspondientes
al período seleccionado por defecto.

---

### Criterio 23

> Dado que el usuario selecciona modo quincenal, entonces las estadísticas
> muestran los datos de la quincena en curso (1–15 o 16–fin de mes según
> la fecha actual) con timezone del usuario.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa. La fecha actual del sistema es 24 de junio de 2026 (segunda
quincena del mes: rango 16 al 30 de junio de 2026). Existe un registro de
gasto creado en el día de hoy (24 de junio de 2026) con categoría
`Comida` y monto `7`.

**Pasos:**

1. Navegar a la vista de estadísticas.
2. Seleccionar el modo "Quincenal" en el selector de período.
3. Observar el rango de fechas mostrado por la interfaz (si lo muestra) y
   los montos listados.

**Resultado esperado:** El período mostrado corresponde al rango 16–30 de
junio de 2026 (segunda quincena del mes en curso, según la fecha actual
24 de junio de 2026). El registro de `7` en `Comida` creado hoy aparece
incluido en el total. No se muestran montos de registros con fecha
anterior al 16 de junio de 2026 ni posterior al 30 de junio de 2026.

---

### Criterio 24

> Dado que el usuario selecciona modo mensual, entonces las estadísticas
> muestran los datos del mes calendario en curso con timezone del usuario.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa. La fecha actual del sistema es 24 de junio de 2026. Existe un
registro de gasto creado hoy con categoría `Comida` y monto `7` (el mismo
del Criterio 23, si sigue vigente).

**Pasos:**

1. Navegar a la vista de estadísticas.
2. Seleccionar el modo "Mensual" en el selector de período.
3. Observar el rango de fechas mostrado por la interfaz (si lo muestra) y
   los montos listados.

**Resultado esperado:** El período mostrado corresponde al mes calendario
de junio de 2026 completo (1 al 30 de junio de 2026). El registro de `7`
en `Comida` creado el 24 de junio de 2026 aparece incluido en el total. No
se muestran montos de registros con fecha de mayo de 2026 ni de julio de
2026.

---

### Criterio 25

> Dado que la vista de estadísticas está activa, entonces se muestran el
> total de gastos, el total de ingresos y el balance neto (ingresos −
> gastos) para el período seleccionado, separados en dos gráficos
> continuos.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa. Dentro del período actual existen: un registro de categoría
`Ingresos` con monto `100`, y un registro de categoría `Comida` con monto
`30`.

**Pasos:**

1. Si los registros descritos en la precondición no existen, crearlos
   desde el formulario de nuevo registro: uno con categoría `Ingresos`,
   descripción `Sueldo`, monto `100`; otro con categoría `Comida`,
   descripción `Supermercado`, monto `30`.
2. Navegar a la vista de estadísticas, con el período por defecto que
   incluya la fecha actual.
3. Localizar el total de ingresos, el total de gastos y el balance neto
   mostrados en pantalla.
4. Observar si los gastos y los ingresos se presentan en dos gráficos
   separados.

**Resultado esperado:** El total de ingresos mostrado es `100`. El total
de gastos mostrado incluye `30` (más cualquier otro gasto vigente del
período). El balance neto mostrado equivale a total de ingresos menos
total de gastos (`100` menos el total de gastos del período). Los gastos
y los ingresos se muestran en dos gráficos separados (no mezclados en uno
solo), según lo exige el criterio ("dos gráficos continuos").

---

### Criterio 26

> Dado que el usuario no tiene registros en el período seleccionado,
> entonces la vista muestra montos en cero sin errores ni datos de otros
> períodos.

**Precondición:** El usuario `prueba.alta1@example.com` tiene sesión
activa. No existe ningún registro con fecha dentro de un período
verificable y distinto al actual: por ejemplo, ningún registro con fecha
en enero de 2024 (mes calendario completo, alejado de la fecha actual
24 de junio de 2026, y previo a la creación de la cuenta de prueba, por lo
que no puede tener registros).

**Pasos:**

1. Navegar a la vista de estadísticas.
2. Seleccionar el modo "Mensual".
3. Navegar con el selector de período hacia atrás hasta llegar a enero de
   2024.
4. Observar los totales de gastos, ingresos y balance neto mostrados, y
   si aparece algún mensaje de error en pantalla.

**Resultado esperado:** El total de gastos, el total de ingresos y el
balance neto muestran `0` (cero). No se muestra ningún mensaje de error
ni pantalla en blanco rota. No aparecen montos correspondientes a otros
períodos (por ejemplo, no aparece el `100` de ingresos ni el `30` de
gastos del Criterio 25, que pertenecen al período de junio de 2026).

---
