# BUILD-READY.md — Checklist de arranque del build

Cada ítem es una casilla con respuesta sí/no, no una opinión. Marcalas
una por una. Si **todas** quedan en sí, arrancá el build (Tarea 1 de
`plan.md`) sin dudar. Si alguna queda en no, el texto bajo "Si es no"
te dice exactamente qué cerrar primero.

Las líneas "Evidencia" son el resultado de una verificación hecha
sobre el estado real del repo al momento de generar este checklist
(2026-06-29, rama `docs/build-ready`, mismo commit que `develop`). Si
pasó tiempo desde entonces, no asumas que siguen vigentes sin
revisarlas de nuevo.

---

## 1. Spec sin huecos bloqueadores

- [ ] `spec.md` tiene Objetivo, Scope (qué sí/qué no) y Criterios de
      aceptación numerados de forma correlativa sin huecos.
  **Evidencia:** 26 criterios, numerados 1–26 sin saltos, agrupados en
  Autenticación, Vinculación de Telegram, CRUD, Registro automático,
  Categorías y Estadísticas. Sección de no-goals presente y específica.

- [ ] Ningún criterio de aceptación depende de una decisión todavía no
      tomada (ej. "se decidirá más adelante").
  **Evidencia:** no se encontró ese patrón en los 26 criterios.

---

## 2. ADRs cerrados y consistentes con la spec

- [ ] Todos los ADR en `docs/adr/` tienen `## Estado` → `Aceptado`.
  **Evidencia:** hay **5** ADRs (0001–0005), no 3 — el número subió
  desde que se escribió el objetivo de esta tarea. Los 5 están en
  estado `Aceptado`.

- [x] Ningún ADR contradice a otro sobre el mismo criterio.
  **Corregido (2026-06-29):** ADR-0003 decía que la caída de servicio
  (#14, #15) "se implementa como un health-check de n8n contra el
  endpoint de FastAPI", contradiciendo a ADR-0005 ("Sin monitoreo
  activo ni health-check programado. La detección es reactiva"). Se
  editó el párrafo de ADR-0003 para que remita a ADR-0005 como la
  decisión vigente sobre #14/#15, en vez de describir un mecanismo
  propio que quedaba obsoleto. Verificado que `docs/diseño.md` y
  `plan.md` no repiten la mención al mecanismo de health-check
  descartado (la única otra mención de "health-check" en `plan.md`
  es el endpoint `GET /health` de la Tarea 2, un concepto distinto:
  liveness check, no detección de caída para Telegram).

- [ ] Cada ADR referencia los criterios de `spec.md` que motivan la
      decisión (no decisiones "porque sí").
  **Evidencia:** los 5 ADR citan números de criterio en su sección de
  Contexto.

---

## 3. Diseño coherente con los ADRs

- [ ] `docs/diseño.md` cubre una página por cada flujo que los ADRs
      dan por existente (login, registro, dashboard, registros,
      categorías, estadísticas, vinculación de Telegram).
  **Evidencia:** las 7 páginas están presentes (`/login`, `/registro`,
  `/dashboard`, `/registros`, `/categorias`, `/estadisticas`,
  `/telegram`).

- [ ] Ningún componente o flujo de `docs/diseño.md` asume un mecanismo
      que un ADR contradice.
  **Depende del punto 2:** revisar si `/telegram` en `docs/diseño.md`
  describe o ilustra el mecanismo de ADR-0003 (health-check) en vez
  del de ADR-0005 (reactivo) — si lo hace, corregirlo junto con el
  ADR.

---

## 4. Cada criterio de aceptación tiene su prueba manual

- [ ] Los 26 criterios de `spec.md` tienen una sección `### Criterio
      N` correspondiente en `docs/pruebas-manuales.md`.
  **Evidencia:** se confirmaron las 26 secciones (`Criterio 1` a
  `Criterio 26`), sin huecos.

- [ ] Ninguna prueba manual fue generada fuera de la skill
      `nueva-prueba-manual` (formato canónico: precondición concreta,
      pasos numerados, resultado esperado observable, estado).
  **Cómo verificarlo:** no es derivable solo del contenido del
  archivo; confirmalo vos mismo si alguna prueba se escribió a mano
  sin pasar por la skill.

---

## 5. `plan.md` con criterio de hecho por tarea

- [ ] Cada tarea de `plan.md` tiene una línea `**Criterio de hecho:**`
      verificable sin ambigüedad (comando o acción + resultado
      esperado concreto).
  **Evidencia:** las 15 tareas (Tarea 1 a Tarea 15) tienen esa línea.

- [ ] Quedó explícito que `plan.md` actual es solo el núcleo (setup →
      modelo de datos → auth → primer CRUD) y que Telegram, n8n,
      categorías personalizadas y estadísticas son una tanda futura
      todavía no planificada.
  **Evidencia:** así lo declara el propio `plan.md` en su
  introducción. Esto **no bloquea** arrancar el build de las 15 tareas
  actuales, pero significa que este checklist certifica el arranque
  del núcleo, no del proyecto completo — antes de la Tarea 16 hace
  falta otra ronda de planificación.

---

## 6. `.env.example` y `SETUP.md`

- [ ] `.env.example` existe en la raíz y documenta cada variable que
      `plan.md` necesita (`JWT_SECRET`, `DATABASE_URL`).
  **Evidencia:** presente, con ambas variables y su origen explicado.

- [ ] `SETUP.md` lista software a instalar, pasos de variables de
      entorno y un checklist propio de "¿puedo arrancar la Tarea 1/2?".
  **Evidencia:** presente, con tabla de herramientas, pasos de env vars
  y checklist al final.

---

## 7. Agentes y skills con smoke-test pasado

- [ ] Existe evidencia registrada (commit, `CONTEXT.md`, archivo de
      resultado) de que cada agente (`arquitecto`, `diseñador`, `qa`,
      `reviewer`, `implementer`) y cada skill (`nuevo-adr`,
      `nueva-prueba-manual`) corrió al menos una vez con resultado
      correcto.
  **Nota:** este ítem quedó fuera de la verificación a pedido
  explícito del usuario (2026-06-29) — no se revisó ni se corrigió.
  Marcalo vos mismo cuando corresponda.

---

## 8. Repo en gitflow con `develop` al día

- [ ] `main` y `develop` existen y `main` solo recibe merges desde
      `develop`.
  **Evidencia:** confirmado por historial — `develop` está 29 commits
  adelante de `main` (trabajo aún no liberado a `main`, esperado en
  gitflow), y no hay commits en `main` que falten en `develop`.

- [ ] Todas las ramas tipadas (`feat/`, `docs/`, `chore/`) ya están
      mergeadas a `develop`; no queda trabajo terminado sin integrar.
  **Evidencia:** las 10 ramas tipadas existentes tienen 0 commits sin
  mergear a `develop`.

- [ ] El working tree está limpio (sin cambios sin commitear).
  **Evidencia:** `git status` limpio en el momento de esta verificación.

- [ ] `develop` local está sincronizado con `origin/develop`.
  **Si es no:** `develop` local tiene 4 commits que `origin/develop`
  todavía no tiene (`chore: .env.example y SETUP.md...`, su merge,
  `docs: plan.md inicial...`, su merge). **Qué cerrar primero:** `git
  push origin develop` si la intención es que el remoto refleje el
  estado real antes de empezar el build.

---

## Resultado

- Si marcaste **todas** las casillas en sí: arrancá la Tarea 1 de
  `plan.md`.
- La contradicción ADR-0003 / ADR-0005 (punto 2) ya quedó corregida.
- El punto 7 (smoke-test) quedó deliberadamente sin revisar a pedido
  del usuario; resolvelo o descartalo según corresponda antes de
  considerar el checklist completo.
