# plan.md — Núcleo (setup → modelo de datos → auth → primer CRUD)

Primeras 15 tareas atómicas de implementación, en orden, para que el
agente `implementer` las ejecute una por una. Cada tarea es ejecutable
en menos de 1 hora y tiene un criterio de hecho verificable sin
ambigüedad.

No incluye: vinculación de Telegram, registro automático vía n8n,
categorías personalizadas (CRUD), estadísticas. Eso es la siguiente
tanda de `plan.md`, una vez cerrado este núcleo.

Stack según `AGENTS.md`: Next.js 15 (App Router, TypeScript estricto,
Tailwind, shadcn/ui) en `web/`; FastAPI (Python) en `api/`; PostgreSQL
como base de datos.

Esta versión incorpora las decisiones tomadas tras la revisión crítica
de la versión anterior del plan. El detalle y la justificación de cada
decisión está en `CONTEXT.md`.

**Arquitectura de comunicación cliente/servidor (aplica a todas las
tareas de auth y CRUD):** el navegador nunca llama directo a FastAPI
(`localhost:8000`). Toda petición protegida pasa por un Route Handler
de Next.js que actúa de proxy: lee el JWT de la cookie `httpOnly`,
lo reenvía a FastAPI en el header `Authorization`, y devuelve la
respuesta al cliente. Esto evita configurar CORS con credenciales
cross-origin y mantiene el JWT fuera del alcance de JavaScript en el
navegador en todo momento.

---

## Tarea 1 — Inicializar proyecto Next.js en `web/` + shadcn/ui

**Qué hacer:** Desde la raíz del repo, generar el proyecto Next.js 15:

```
npx create-next-app@latest web --typescript --tailwind --app --eslint --src-dir --import-alias "@/*"
```

Verificar que `web/tsconfig.json` tenga `"strict": true` (viene por
defecto en Next.js 15; si no, agregarlo). Inicializar shadcn/ui y
agregar los componentes que va a usar el resto del plan:

```
npx shadcn@latest init
npx shadcn@latest add button input label select form card table dialog badge alert tabs skeleton sonner
```

**Criterio de hecho:** Parado en `web/`, `npm run dev` levanta un
servidor en `http://localhost:3000` que muestra la página default de
Next.js sin errores en consola. `web/tsconfig.json` contiene
`"strict": true`. El archivo de estilos globales importa Tailwind
(directivas `@tailwind` o `@import "tailwindcss"` según la versión
instalada). `web/src/components/ui/` contiene los archivos `button.tsx`,
`input.tsx`, `select.tsx`, `form.tsx`, `card.tsx`, `table.tsx`,
`dialog.tsx`, `badge.tsx`, `alert.tsx`, `tabs.tsx`, `skeleton.tsx` y
`sonner.tsx`.

**ADR:** —
**Prueba manual:** —
**Depende de:** ninguna tarea.

---

## Tarea 2 — Inicializar FastAPI con conexión a PostgreSQL en `api/`

**Qué hacer:** Crear `api/` con un entorno virtual de Python e
instalar `fastapi`, `uvicorn`, `sqlalchemy`, `psycopg2-binary`,
`python-dotenv`, `pyjwt`, `passlib[bcrypt]`. Crear
`docker/docker-compose.dev.yml` con un único servicio `db` (imagen
`postgres:16`, variables `POSTGRES_PASSWORD=postgres`,
`POSTGRES_DB=pfa`, puerto `5432`) — se versiona en el repo en vez de
documentar un `docker run` suelto, para que levantar la base de datos
de desarrollo sea reproducible (`docker compose -f docker/docker-compose.dev.yml up -d`).
Crear `api/main.py` con un endpoint `GET /health` que ejecute
`SELECT 1` contra la base y devuelva su resultado. Generar un
`JWT_SECRET` (ej. `openssl rand -hex 32`) y documentarlo en un
`.env.example` en la raíz del repo; copiar su valor real a
`api/.env` (junto a `DATABASE_URL=postgresql://postgres:postgres@localhost:5432/pfa`)
y a `web/.env.local` — es el secreto compartido que exige ADR-0002
para firmar en FastAPI y verificar en Next.js.

**Criterio de hecho:** Con `docker compose -f docker/docker-compose.dev.yml up -d`
y `uvicorn main:app --reload` ejecutándose desde `api/`,
`GET http://localhost:8000/health` devuelve `200` con un cuerpo que
confirma conexión exitosa a la base (ej.
`{"status": "ok", "db": "connected"}`). `api/.env` y `web/.env.local`
contienen el mismo valor de `JWT_SECRET`.

**Nota sobre el alcance de esta tarea:** agrupa entorno Python +
infraestructura Docker + código de health-check + variables
compartidas. Ver justificación en `CONTEXT.md` (no se divide porque
ninguna de esas partes es verificable por sí sola sin las otras).

**ADR:** ADR-0002 (secreto compartido), ADR-0003 (FastAPI como única
puerta de escritura a Postgres — esta tarea sienta la base de esa
puerta).
**Prueba manual:** —
**Depende de:** ninguna tarea.

---

## Tarea 3 — Modelo de datos: tablas `User`, `Category`, `Record`, `TelegramLinkCode`

**Qué hacer:** Definir los modelos SQLAlchemy en `api/models.py` según
ADR-0001 y ADR-0004:
- `User`: `id` (UUID), `email` (único), `password_hash`,
  `telegram_user_id` (nullable), `last_attempt_failed` (bool, default
  `false`).
- `Category`: `id` (UUID), `name`, `slug` (único, ver Tarea 4),
  `is_fixed` (bool), `user_id` (nullable, null para categorías fijas).
  Agregar un índice único parcial sobre `lower(name)` para las filas
  con `user_id IS NULL` (categorías fijas) y otro índice único parcial
  sobre `(user_id, lower(name))` para las filas con `user_id IS NOT NULL`
  (categorías propias de cada usuario). La validación cruzada
  fija-vs-propia del criterio 20 se hace en la capa de aplicación
  cuando se implemente el alta de categorías personalizadas (fuera de
  este núcleo); estos índices solo evitan duplicados dentro de cada
  grupo a nivel de base de datos.
- `Record`: `id` (UUID), `user_id`, `category_id`, `description`,
  `amount` (`Numeric(10, 2)` — dos decimales, suficiente para montos
  personales y sin las complicaciones de serialización JSON de un tipo
  decimal de precisión arbitraria), `created_at`.
- `TelegramLinkCode`: `code`, `user_id`, `expires_at`, `used_at`
  (nullable).

Todas las claves primarias son UUID v4 (no autoincrementales, para no
exponer IDs predecibles en la API). Configurar Alembic en `api/` y
generar la migración inicial.

**Criterio de hecho:** `alembic upgrade head` corre sin error. Conectado
a la base con `psql`, `\dt` lista las 4 tablas (`users`, `categories`,
`records`, `telegram_link_codes`). `\d categories` muestra los dos
índices únicos parciales descritos.

**Nota sobre el alcance de esta tarea:** agrupa diseño de esquema +
configuración de Alembic + ejecución de la migración. Ver
justificación en `CONTEXT.md`.

**ADR:** ADR-0001, ADR-0004.
**Prueba manual:** —
**Depende de:** Tarea 2.

---

## Tarea 4 — Seed de las 6 categorías fijas

**Qué hacer:** Script `api/seed.py` que inserta, si no existen, las 6
categorías fijas con `is_fixed=true` y `user_id=null`: `Comida`
(`slug="comida"`), `Transporte` (`slug="transporte"`), `Vivienda`
(`slug="vivienda"`), `Entretenimiento` (`slug="entretenimiento"`),
`Sin categoría` (`slug="sin-categoria"`), `Ingresos`
(`slug="ingresos"`). El `slug` es la referencia estable: en código,
una constante `INGRESOS_SLUG = "ingresos"` y `SIN_CATEGORIA_SLUG =
"sin-categoria"` se usan en cualquier lugar que necesite identificar
estas categorías por significado (ADR-0001), en vez de depender de un
UUID generado en runtime que podría diferir entre entornos.

**Criterio de hecho:** Ejecutar `python seed.py` y luego
`SELECT name, slug FROM categories WHERE is_fixed = true;` en `psql`
devuelve exactamente esas 6 filas con sus slugs, sin duplicados al
volver a ejecutar el script.

**ADR:** ADR-0001.
**Prueba manual:** Criterio 18 (parcial — solo el dato en backend,
falta la vista de gestión de categorías que se agrega en una tanda
posterior).
**Depende de:** Tarea 3.

---

## Tarea 5 — Endpoint `POST /auth/register` (autentica de inmediato)

**Qué hacer:** En `api/`, implementar `POST /auth/register` que recibe
`email` y `password`, valida: email con formato válido y no existente
en `users`; password con mínimo 6 caracteres, al menos 1 mayúscula, 1
número y 1 carácter especial. Si pasa, hashea el password con `bcrypt`
(vía `passlib`) y crea el `User`. Crear en `api/auth.py` una utilidad
`create_access_token(user)` que firma un JWT con `PyJWT` (payload:
`user_id`, `email`, expiración corta, ej. 15 minutos, firmado con
`JWT_SECRET` de la Tarea 2). El registro exitoso devuelve ese JWT en
la respuesta — el criterio 1 de `spec.md` exige que el usuario "quede
autenticado en la sesión activa" al registrarse, no que tenga que
loguearse por separado después.

**Criterio de hecho:** `POST /auth/register` con
`{"email": "a@a.com", "password": "Clave1!"}` devuelve `201` y un
cuerpo con un JWT válido, y crea la fila en `users`. Repetir la misma
petición con el mismo email devuelve `400` (o `409`) con mensaje de
error y no se crea una segunda fila ni se devuelve token. Una petición
con password `clave1` (sin mayúscula ni carácter especial) devuelve
`400`.

**ADR:** ADR-0002.
**Prueba manual:** Criterios 1, 4 (parcial — falta la página web, se
agrega en la Tarea 8).
**Depende de:** Tarea 3.

---

## Tarea 6 — Endpoint `POST /auth/login` + dependencia de verificación de JWT

**Qué hacer:** Implementar `POST /auth/login` que recibe `email` y
`password`, verifica contra `users` y, si son correctos, reutiliza
`create_access_token` de la Tarea 5 para devolver el JWT. Además,
crear en `api/auth.py` la dependencia de FastAPI `get_current_user`
que lee el header `Authorization: Bearer <token>`, verifica firma y
expiración con `PyJWT` contra `JWT_SECRET`, y devuelve el `User`
correspondiente (o `401` si el token es inválido/expirado). Esta
dependencia es la que usarán todos los endpoints protegidos de las
tareas siguientes (ADR-0002: "FastAPI vuelve a verificar la firma del
JWT en cada endpoint de su API").

**Criterio de hecho:** `POST /auth/login` con las credenciales creadas
en la Tarea 5 devuelve `200` y un JWT en el cuerpo. La misma petición
con password incorrecto devuelve `401` con mensaje de error y sin
token. Un endpoint de prueba protegido con `get_current_user` devuelve
`401` sin header `Authorization` y `200` con un token válido.

**ADR:** ADR-0002.
**Prueba manual:** Criterios 2, 5 (parcial — falta la página web, se
agrega en la Tarea 9).
**Depende de:** Tarea 5.

---

## Tarea 7 — Layout autenticado: `/dashboard` placeholder + `Navbar`

**Qué hacer:** Crear, en `web/`, un layout para las rutas autenticadas
con un `Navbar` (enlaces a Dashboard, Registros — esta última puede
ser un placeholder hasta la Tarea 13 — y un botón "Cerrar sesión" sin
lógica todavía, se conecta en la Tarea 10). Crear la página
`/dashboard` con un texto placeholder ("Bienvenido"). Esto da un
destino real para los redirects de registro/login de las próximas dos
tareas, en vez de asumir una ruta que no existe.

**Criterio de hecho:** Navegar a `/dashboard` muestra el `Navbar` (con
el botón de logout visible, aunque todavía no haga nada) y el texto
placeholder.

**ADR:** —
**Prueba manual:** —
**Depende de:** Tarea 1.

---

## Tarea 8 — Página `/registro` en Next.js

**Qué hacer:** Construir `/registro` según `docs/diseño.md`: `Card`,
`Form`, `Input`/`Label` para email y password, `Button` de submit,
`Alert` para error de email duplicado, enlace a `/login`. Al enviar,
llama a un Route Handler propio (`web/src/app/api/auth/register/route.ts`)
que reenvía la petición a `POST /auth/register` (Tarea 5); si la
respuesta incluye un JWT, ese mismo Route Handler lo guarda en una
cookie `httpOnly` antes de devolver éxito al cliente. Redirige a
`/dashboard`.

**Criterio de hecho:** Completar el formulario con un email nuevo y
password válido crea la cuenta, setea la cookie `httpOnly` y redirige
a `/dashboard` sin pasar por `/login`. Repetir con el mismo email
muestra el `Alert` de error y permanece en `/registro`.

**ADR:** —
**Prueba manual:** Criterios 1, 4.
**Depende de:** Tarea 5, Tarea 7.

---

## Tarea 9 — Página `/login` en Next.js + cookie httpOnly

**Qué hacer:** Construir `/login` según `docs/diseño.md`: `Card`,
`Form`, `Input`/`Label`, `Button`, `Alert` para credenciales
incorrectas. Al enviar, llama a un Route Handler propio
(`web/src/app/api/auth/login/route.ts`) que reenvía la petición a
`POST /auth/login` (Tarea 6) y, si es exitoso, guarda el JWT recibido
en una cookie `httpOnly` (el navegador nunca ve el JWT directamente,
solo el Route Handler que corre en el servidor de Next.js). Redirige a
`/dashboard`.

**Criterio de hecho:** Login con credenciales correctas setea la
cookie `httpOnly` (visible en DevTools → Application → Cookies, no
accesible desde `document.cookie`) y redirige a `/dashboard`. Login
con password incorrecto muestra el `Alert` de error y no setea cookie.

**ADR:** ADR-0002.
**Prueba manual:** Criterios 2, 5.
**Depende de:** Tarea 6, Tarea 7.

---

## Tarea 10 — Middleware de rutas protegidas + logout

**Qué hacer:** Crear `web/middleware.ts` que verifique firma y
expiración del JWT de la cookie en cada request a rutas protegidas
(`/dashboard`, `/registros`, etc.); si no hay cookie válida, redirige a
`/login`. Conectar el botón de logout del `Navbar` (Tarea 7) para que
llame a un Route Handler que borre la cookie y luego redirija a
`/login`.

**Nota de librería:** la verificación se hace con `jose`, no con
`PyJWT`. `PyJWT` es una librería de Python y no puede ejecutarse en el
Edge runtime de Next.js (es JavaScript/TypeScript, no Python); `jose`
es el equivalente compatible con Edge. Ambos leen el mismo
`JWT_SECRET` compartido (Tarea 2), así que la verificación es
equivalente de lado a lado aunque la librería cambie según el
lenguaje. Ver detalle en `CONTEXT.md`.

**Nota de invalidación (ADR-0002):** la ADR exige que exista "un
mecanismo de invalidación (denylist mínima o refresh de corta vida)"
además de borrar la cookie. Se decide que la expiración corta del JWT
(15 minutos, fijada en la Tarea 5/6) cumple ese rol — no se construye
un denylist en este núcleo. Limitación aceptada a propósito: si
alguien obtuvo el JWT crudo antes del logout (fuera del navegador),
ese token sigue siendo válido contra FastAPI hasta que expire
naturalmente, aunque la cookie ya se haya borrado. Para el alcance de
este proyecto y la prueba manual del criterio 3 (mismo navegador, sin
copia externa del token) esto es suficiente. Ver `CONTEXT.md`.

**Criterio de hecho:** Acceder a `/dashboard` sin cookie válida (ej.
en incógnito) redirige a `/login`. Tras presionar el botón de logout
estando autenticado, la sesión se borra y se redirige a `/login`;
pegar luego la URL de `/dashboard` directamente en la barra de
direcciones vuelve a redirigir a `/login`.

**ADR:** ADR-0002.
**Prueba manual:** Criterio 3.
**Depende de:** Tarea 9, Tarea 7.

---

## Tarea 11 — Endpoint `GET /categories`

**Qué hacer:** Implementar `GET /categories` (protegido con
`get_current_user`, Tarea 6) que devuelve las categorías fijas
sembradas en la Tarea 4. (Las categorías personalizadas por usuario se
agregan en una tanda posterior del plan; por ahora la respuesta es la
misma para cualquier usuario autenticado.)

**Criterio de hecho:** `GET /categories` con un JWT válido devuelve
`200` y un array con las 6 categorías fijas (`name`, `slug`, `id`).
Sin header `Authorization` devuelve `401`.

**ADR:** ADR-0001.
**Prueba manual:** —
**Depende de:** Tarea 4, Tarea 6.

---

## Tarea 12 — Endpoint `POST /records` con validaciones de negocio

**Qué hacer:** Implementar `POST /records` (protegido con
`get_current_user`) que recibe `category_id` (opcional), `description`
(opcional), `amount` (obligatorio). Reglas: sin `amount` → rechazar
con mensaje `"Se debe ingresar monto para poder hacer registro"`;
`amount < 1` o negativo → rechazar; decimales permitidos; sin
`description` → guardar `"Sin descripción"`; sin `category_id` →
guardar el id de la categoría fija `slug="sin-categoria"` (Tarea 4).
Crear el Route Handler proxy `web/src/app/api/records/route.ts` (POST)
que reenvía la petición del cliente a este endpoint con el JWT de la
cookie en el header `Authorization`.

**Criterio de hecho:** `POST /records` sin `amount` devuelve `400` con
ese mensaje exacto y no crea fila. Con `amount: 0.5` o `amount: -5`
devuelve error y no crea fila. Con `amount: 1.25` crea la fila. Sin
`description` la fila queda con `"Sin descripción"`. Sin
`category_id` la fila queda con la categoría `"Sin categoría"`.

**Nota de alcance (ADR-0003):** la ADR describe un único
`POST /records` que sirve tanto al CRUD web (sesión JWT, esta tarea)
como a n8n (autenticado con una API key estática propia del canal de
Telegram). Cuando llegue la tanda de integración con Telegram, esa
rama de autenticación se agrega a este mismo endpoint (aceptando JWT o
API key según el header presente) — no se crea un endpoint paralelo,
para no romper la decisión de la ADR de una sola puerta de escritura
con reglas de validación unificadas.

**ADR:** ADR-0001, ADR-0003.
**Prueba manual:** Criterios 8, 9, 10, 13 (parcial — falta UI, se
agrega en la Tarea 13).
**Depende de:** Tarea 6, Tarea 4.

---

## Tarea 13 — Endpoint `GET /records` + página `/registros` (listado y alta)

**Qué hacer:** Implementar `GET /records` (protegido, devuelve solo
los registros del usuario autenticado) y su Route Handler proxy
(`GET` en `web/src/app/api/records/route.ts`, junto al `POST` de la
Tarea 12). Construir `/registros` según `docs/diseño.md`: `Table` con
categoría/descripción/monto, formulario de alta (`Select` de categoría
poblado desde `GET /categories` vía proxy, `Input` de descripción y
monto) que llama al proxy de `POST /records`, `Skeleton` mientras
carga, `Empty state` si no hay registros.

**Criterio de hecho:** Crear un registro desde el formulario web con
categoría `Comida`, descripción `Café`, monto `5.50` lo muestra de
inmediato en la tabla de `/registros` con esos mismos datos.

**ADR:** —
**Prueba manual:** Criterio 8.
**Depende de:** Tarea 12, Tarea 11.

---

## Tarea 14 — Endpoint `PUT /records/{id}` + edición en la UI

**Qué hacer:** Implementar `PUT /records/{id}` (protegido, solo sobre
registros propios) que sobrescribe `category_id`/`description`/
`amount` sin guardar el valor anterior en ningún lado, y su proxy
correspondiente (`web/src/app/api/records/[id]/route.ts`). Agregar
acción de editar en la fila de la tabla de `/registros` que abre el
mismo formulario de alta precargado.

**Criterio de hecho:** Editar un registro existente (cambiar
descripción y monto) y guardar refleja los nuevos valores en la
tabla. Recargar la página o reabrir el registro no muestra en ningún
lugar de la interfaz el valor anterior.

**ADR:** —
**Prueba manual:** Criterio 11.
**Depende de:** Tarea 13.

---

## Tarea 15 — Endpoint `DELETE /records/{id}` + confirmación y borrado físico

**Qué hacer:** Implementar `DELETE /records/{id}` (protegido, solo
sobre registros propios) que borra la fila físicamente de Postgres
(no soft-delete), reusando el proxy `web/src/app/api/records/[id]/route.ts`
de la Tarea 14 (agregar el método `DELETE`). Agregar acción de borrar
en la tabla de `/registros` con `Dialog` de confirmación antes de
ejecutar.

**Criterio de hecho:** Confirmar el borrado de un registro lo quita de
la tabla de inmediato. Recargar la página (F5) confirma que sigue sin
aparecer. `SELECT * FROM records WHERE id = '<id>';` en `psql` no
devuelve filas.

**ADR:** —
**Prueba manual:** Criterio 12.
**Depende de:** Tarea 13.
