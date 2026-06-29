# SETUP.md — Preparación previa al build

Pasos manuales que un humano debe completar **antes** de que `implementer`
ejecute la Tarea 1 de `plan.md`. Nada de esto es código de la aplicación;
es la base sobre la que corren las Tareas 1 y 2.

Stack (`AGENTS.md`): Next.js 15 en `web/`, FastAPI en `api/`, PostgreSQL
como base de datos, sin proveedores externos de auth ni de base de datos
(ADR-0002, ADR-0003) — todo corre self-hosted (Docker en desarrollo, VPS
propio en producción).

---

## 1. Software que debe estar instalado localmente

| Herramienta | Para qué | Usado en |
|---|---|---|
| Node.js 18.18+ (o 20+) y npm | Next.js 15 requiere esta versión mínima | Tarea 1 |
| Python 3.11+ y `venv` | Entorno virtual de FastAPI | Tarea 2 |
| Docker + Docker Compose | Levantar Postgres 16 vía `docker compose -f docker/docker-compose.dev.yml up -d` | Tarea 2 |
| `openssl` (o equivalente: `python -c "import secrets; print(secrets.token_hex(32))"`) | Generar `JWT_SECRET` | Tarea 2 |
| `psql` (cliente de Postgres) | Verificar el criterio de hecho de Tarea 2 (`GET /health`) y de tareas posteriores (`\dt`, `SELECT ...`) | Tarea 2+ |

No hace falta crear ninguna cuenta en un servicio externo para las Tareas
1-15 (núcleo de `plan.md`): no hay Supabase, no hay proveedor de auth de
terceros (no-goal explícito en `spec.md`), no hay base de datos gestionada.
Todo el stack de este núcleo es local.

---

## 2. Variables de entorno

Plantilla en `.env.example` (raíz del repo). Sus valores se copian a dos
archivos que la Tarea 2 crea y que **no se commitean**:

| Variable | Va en | De dónde sale |
|---|---|---|
| `JWT_SECRET` | `api/.env` **y** `web/.env.local` (mismo valor en ambos) | Generarlo vos: `openssl rand -hex 32`. No es la clave de ningún servicio externo, es un secreto que vos mismo creás una vez y reutilizás en los dos archivos. |
| `DATABASE_URL` | `api/.env` | Valor fijo de desarrollo: `postgresql://postgres:postgres@localhost:5432/pfa`. Coincide con las credenciales hardcodeadas en `docker/docker-compose.dev.yml` (que la propia Tarea 2 crea) — no hay que "obtenerlo" de ningún lado, solo copiarlo tal cual. |

Pasos concretos:

1. Copiar `.env.example` a `api/.env` y a `web/.env.local` (ambos archivos
   los crea la Tarea 2; si todavía no existen `api/` ni `web/`, esto se
   hace en el momento de ejecutar esa tarea, no antes).
2. Generar el valor de `JWT_SECRET` una sola vez y pegarlo igual en los
   dos archivos.
3. Dejar `DATABASE_URL` con el valor de la tabla anterior.

---

## 3. Cuentas externas

Ninguna cuenta externa bloquea la Tarea 1 ni la Tarea 2. El proyecto sí va
a necesitar, en una tanda posterior de `plan.md` (vinculación de Telegram,
registro automático vía n8n — explícitamente fuera de este núcleo según
`plan.md` y ADR-0003/0004/0005):

- Un bot de Telegram creado vía [@BotFather](https://t.me/BotFather) (da
  un `bot_token` y un `bot_username`).
- Una instancia de n8n (propia, en el mismo VPS según `AGENTS.md`) con un
  workflow que llame a `POST /records` de FastAPI usando una API key
  estática (ADR-0003).

Ninguno de los dos tiene todavía un nombre de variable de entorno
definido en `plan.md` — eso se decide cuando se escriba esa tanda del
plan, no ahora. Si querés tener la cuenta de Telegram lista de antemano,
crear el bot con BotFather no tiene costo ni efectos secundarios y se
puede hacer en cualquier momento; pero no es un requisito para arrancar
el build.

---

## 4. Checklist — ¿se puede arrancar el build?

### Tarea 1 (Next.js + shadcn/ui)

- [ ] Node.js 18.18+ (o 20+) instalado (`node -v`)
- [ ] npm disponible (`npm -v`)
- [ ] Conexión a internet (npm e instalación de shadcn/ui descargan paquetes)

### Tarea 2 (FastAPI + Postgres)

- [ ] Python 3.11+ instalado (`python --version` o `python3 --version`)
- [ ] Docker y Docker Compose instalados y el daemon corriendo (`docker ps`)
- [ ] Forma de generar un valor hex aleatorio para `JWT_SECRET` disponible
      (`openssl` u otro)
- [ ] `psql` instalado para poder verificar el criterio de hecho
- [ ] `.env.example` leído y entendido: sé qué valor va en `JWT_SECRET` y
      cuál en `DATABASE_URL` antes de que la Tarea 2 cree `api/.env` y
      `web/.env.local`

Si todas las casillas están marcadas, `implementer` puede ejecutar la
Tarea 1 (y, encadenada, la Tarea 2) sin frenarse a buscar una credencial
a mitad de camino.
