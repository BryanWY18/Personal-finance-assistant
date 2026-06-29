# CONTEXT.md — Desviaciones y decisiones fuera del flujo normal

Registro de decisiones sobre `plan.md` que no se derivan directamente
de `spec.md`, las ADR o `docs/diseño.md`, según exige `AGENTS.md`.

---

## 2026-06-24 — Revisión y enmienda de `plan.md` (núcleo)

`plan.md` fue generado, criticado (vaguedad, tareas que mezclan
responsabilidades, criterios de hecho incompletos) y luego enmendado
en la misma sesión, antes de que `implementer` ejecutara ninguna
tarea. Pasó de 13 a 15 tareas. Decisiones tomadas por el usuario y
aplicadas:

- **Tarea 4 (id de "Ingresos"):** se agrega columna `slug` única en
  `Category` y se usa `slug="ingresos"` como referencia estable en
  código, en vez de depender de un UUID generado en runtime que podría
  diferir entre entornos.

- **Tarea 3 (`amount`):** se usa `Numeric(10, 2)` en Postgres en vez
  de un entero en centavos. Razón: Postgres maneja `Numeric` sin error
  de redondeo (a diferencia de un `float`), y evita la complejidad
  extra de convertir a/desde centavos en cada capa (API, frontend,
  Telegram) sin necesidad real dado el volumen de la app.

- **Tarea 3 (unique case-insensitive en `Category.name`):** se agregan
  dos índices únicos parciales en la base — uno para categorías fijas
  (`user_id IS NULL`) y otro por usuario (`user_id, lower(name)`) para
  categorías propias. La validación cruzada "no puede repetir el
  nombre de una fija" (criterio 20) no se puede expresar solo con
  estos índices (son grupos distintos a nivel de fila) y se deja para
  la capa de aplicación cuando se implemente el alta de categorías
  personalizadas, fuera de este núcleo.

- **Tarea 9 → ahora Tarea 10 (librería de verificación de JWT):** el
  usuario pidió usar `PyJWT`. Se aplicó `PyJWT` en el lado de FastAPI
  (firma en Tarea 5/6, verificación en la dependencia
  `get_current_user` de Tarea 6) porque ahí corre Python. Para el
  middleware de Next.js (Tarea 10) **no se puede usar `PyJWT`**: es
  una librería de Python y el middleware corre en el Edge runtime de
  Next.js, que ejecuta JavaScript/TypeScript, no Python — no hay forma
  de que `PyJWT` se ejecute ahí sin importar qué tan compatible sea
  con Edge. Se usó `jose` en su lugar para ese punto específico,
  leyendo el mismo `JWT_SECRET` compartido. Si la intención original
  era otra (por ejemplo, mover la verificación de rutas protegidas a
  una llamada server-side a FastAPI en vez de verificar en el
  middleware), eso es un cambio de arquitectura distinto al de ADR-0002
  y debería decidirse explícitamente, no asumirse.

- **Tareas 2 y 3 (mezclan responsabilidades):** se decidió no
  dividirlas. Tarea 2 agrupa entorno Python + Docker Postgres +
  health-check + variables compartidas; Tarea 3 agrupa diseño de
  esquema + configuración de Alembic + migración. En ambos casos,
  separar los pasos generaría una tarea intermedia sin nada propio que
  verificar (ej. "instalar Alembic" no tiene un criterio de hecho
  distinto de "correr la migración"). Se resolvió la preocupación real
  de fondo — el `docker run` manual no quedaba versionado — moviéndolo
  a `docker/docker-compose.dev.yml`.

- **Tareas 7, 8, 11, 12, 13 (ahora 8, 9, 13, 14, 15) — mezclan backend
  y frontend:** se decidió mantenerlas como "vertical slice" (endpoint
  + UI en la misma tarea) en vez de dividirlas en ~18 tareas. Razón:
  `AGENTS.md` exige "un commit por unidad funcional verificable"; un
  commit de solo-backend o solo-UI de una misma feature no es
  verificable de punta a punta por sí solo, así que dividirlas iría
  en contra de esa misma regla.

- **Tarea 1:** se agrega verificación de que Tailwind esté
  efectivamente importado en los estilos globales (no solo que el flag
  `--tailwind` se haya usado en el comando), y se agrega la
  inicialización de shadcn/ui con los componentes que el resto del
  plan necesita, porque ninguna tarea posterior lo hacía pese a usarlos
  todos.

- **Dashboard/Navbar:** se agrega la Tarea 7 (nueva) para crear un
  layout autenticado real con `/dashboard` placeholder y `Navbar` con
  botón de logout (sin lógica todavía). Las Tareas 7-8 (registro,
  login) del plan original redirigían a una ruta que ninguna tarea
  creaba.

- **JWT compartido:** se agrega a la Tarea 2 la generación de
  `JWT_SECRET` y su documentación en `.env.example`, copiado a
  `api/.env` y `web/.env.local`. ADR-0002 exige este secreto
  compartido pero ninguna tarea lo creaba explícitamente.

- **CORS (opción B):** se adoptó el patrón de Route Handlers de
  Next.js como proxy hacia FastAPI para *todas* las llamadas
  protegidas (auth y records), no solo para login. El navegador nunca
  llama directo a `localhost:8000`; el JWT vive en una cookie
  `httpOnly` que solo el servidor de Next.js lee y reenvía. Evita
  configurar `CORSMiddleware` con credenciales cross-origin.

- **`GET /categories`:** se agrega la Tarea 11 (nueva). El formulario
  de alta de registros (Tarea 13) necesita poblar un `Select` de
  categorías y no existía ningún endpoint que las devolviera — solo
  había un script de seed directo a la base.

- **Registro no autenticaba de inmediato (opción A):** `POST
  /auth/register` (Tarea 5) ahora también firma y devuelve un JWT,
  igual que login, porque el criterio 1 de `spec.md` exige que el
  usuario quede autenticado al registrarse, no que tenga que loguearse
  aparte después.

---

## 2026-06-29 — Verificación de `plan.md` contra ADRs y pruebas manuales

Se verificó cada tarea de `plan.md` contra los 5 ADR y
`docs/pruebas-manuales.md`. Hallazgos y su resolución:

- **Tarea 5/8 vs. ADR-0002 (registro emite JWT) — resuelto:**
  ADR-0002 decía que el JWT se emite "al login exitoso" — no
  contemplaba emitirlo también al registrarse, pero la Tarea 5 lo hace
  porque el criterio 1 de `spec.md` lo exige. Se corrigió ADR-0002
  directamente (edición manual del archivo, no generada por el agente
  `arquitecto`): "FastAPI emite, al login **o registro** exitoso, un
  JWT...". Ya no hay desviación entre la ADR y `plan.md`.

- **Tarea 10 vs. ADR-0002 (mecanismo de invalidación de logout):**
  ADR-0002 exige explícitamente que exista "un mecanismo de
  invalidación (denylist mínima o refresh de corta vida)" más allá de
  borrar la cookie. Se decidió que la expiración corta del JWT (15
  minutos, ya fijada en la Tarea 5/6) cumple ese rol — no se construye
  un denylist en este núcleo. Limitación aceptada conscientemente: un
  JWT obtenido fuera del navegador antes del logout sigue siendo válido
  contra FastAPI hasta que expire naturalmente (máximo 15 minutos),
  aunque la cookie ya se haya borrado. Alcanza para la prueba manual
  del criterio 3 (mismo navegador). Si en el futuro se necesita
  revocación inmediata real (ej. requisito de seguridad más estricto),
  hay que agregar un denylist — no está planeado en este núcleo.

- **Tarea 12 vs. ADR-0003 (endpoint compartido con n8n):** ADR-0003
  describe un único `POST /records` que sirve tanto al CRUD web (JWT)
  como a n8n (API key estática). La Tarea 12 solo construye la rama
  JWT, correcto mientras Telegram quede fuera de este núcleo. Se
  agregó una nota explícita en la tarea para que, cuando llegue la
  tanda de integración con Telegram, la rama de API key se agregue al
  mismo endpoint en vez de crear uno paralelo — evita romper la
  decisión de "una sola puerta de escritura" de la ADR.

- **Nota aparte, no accionada:** ADR-0004 dice en su contexto que
  "ADR-0001 ya define la entidad `TelegramLinkCode`", pero ADR-0001
  (tal como está redactada) no la define — solo describe `Record`/
  `Category`. La Tarea 3 cita ambas ADR para esa tabla porque los
  campos coinciden con los que sí lista ADR-0004; la inconsistencia es
  entre las dos ADR, no en `plan.md`, y no se corrigió en esta sesión.
