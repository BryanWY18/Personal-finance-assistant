# AGENTS.md — Contrato del proyecto

Contrato vinculante para todos los agentes (arquitecto, diseñador, qa,
reviewer, implementer) y para cualquier persona que contribuya código a
Personal Finance Assistant. Ninguno puede desviarse de lo aquí descrito
sin antes actualizar este archivo.

## Stack

- Frontend: Next.js 15 (App Router), TypeScript estricto, Tailwind CSS.
- Backend: Python con FastAPI.
- Base de datos: PostgreSQL.
- Automatización de mensajería: n8n (clasificación de gastos vía Telegram).
- Deploy: VPS propio, contenedores Docker (docker-compose).

## Convenciones de TypeScript

- `strict: true` en `tsconfig.json`, sin excepciones por archivo.
- `any` solo permitido con comentario inline `// any: <justificación>` en
  la misma línea; sin justificación, el reviewer rechaza el PR.
- Firmas de funciones públicas y retornos de API tipados explícitamente;
  se permite inferencia en variables locales.
- `@ts-ignore` / `@ts-expect-error` requieren la misma justificación
  inline que `any`.

## Estructura de carpetas esperada

```
/
├── web/              # Next.js 15, App Router
├── api/              # FastAPI, Python
├── docker/           # docker-compose y configs de despliegue
├── .claude/
│   ├── agents/       # definiciones de agentes custom
│   └── skills/       # skills custom
├── docs/
│   └── adr/          # decisiones de arquitectura
├── insumos/          # material de referencia externo
├── spec.md
├── AGENTS.md
├── CONTEXT.md
└── README.md
```

`web/` y `api/` se inicializan recién cuando exista `plan.md` aprobado;
hasta entonces ningún agente genera código de aplicación.

## Política de commits

- Un commit por unidad funcional verificable. Prohibido un commit
  "implement everything" que mezcle features no relacionadas.
- Mensaje en formato `tipo: descripción corta`, con `tipo` en
  `feat`, `fix`, `docs`, `chore`, `refactor`.
- Ninguna sesión de trabajo de un agente termina sin commitear.

## Flujo git (gitflow)

- `main`: rama estable. Solo recibe merges desde `develop` ya validados.
- `develop`: rama de integración. Todo el trabajo en curso converge aquí.
- Una rama tipada por unidad de trabajo, creada desde `develop`:
  `feat/`, `docs/`, `chore/`, `fix/` + nombre descriptivo.
- Toda rama tipada se mergea a `develop` por revisión; nunca directo a
  `main`.

## Regla de CONTEXT.md

- No se escribe código sin plan aprobado.
- Toda edición manual de código (no generada por un agente) se
  documenta en `CONTEXT.md` con la justificación de por qué fue manual.
- `CONTEXT.md` es la fuente de verdad sobre cualquier desviación del
  flujo normal agente-genera-código.

## Prohibiciones explícitas

- `any` sin justificación inline.
- Librerías de componentes pesadas (Material UI, Chakra UI, Ant Design
  u otras con sistema de diseño propio); estilos solo con Tailwind.
- Tests automatizados: fuera de alcance para este proyecto en su
  estado actual.
- Código generado o editado sin plan aprobado previamente.
- Commits no atómicos o con mensajes genéricos sin `tipo:`.
