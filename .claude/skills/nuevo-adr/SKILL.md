---
name: nuevo-adr
description: Usar cuando se quiere registrar una nueva decisión arquitectónica del proyecto en docs/adr/. Garantiza formato canónico, numeración correlativa automática y completitud mínima (alternativas reales, consecuencia negativa explícita, contexto específico del proyecto) — rechaza el ADR y señala qué falta en vez de inventar contenido. No propone ni evalúa la decisión.
argument-hint: <título corto> | <contexto> | <decisión> | <alternativas consideradas> | <consecuencias>
---

**Objetivo:** dado el contenido de una decisión arquitectónica ya
tomada (provisto por un humano o por el agente `arquitecto`), generar
un archivo en `docs/adr/` con el formato canónico del proyecto,
número correlativo correcto y un mínimo de completitud verificable.
Este skill es un procedimiento, no un rol: no opina sobre si la
decisión es buena, no inventa contenido que falte — solo valida y
formatea lo que recibe.

## Template exacto del ADR

```markdown
# ADR {NNNN}: {Título corto y descriptivo}

## Estado

{Propuesto|Aceptado|Deprecado}

## Contexto

{Por qué hizo falta esta decisión. Debe nombrar el problema concreto
del proyecto que la dispara — un criterio de spec.md, una limitación
del stack (AGENTS.md), un ADR previo relacionado, o una restricción
de negocio real. Mínimo 2 líneas.}

## Decisión

{Qué se decidió, en términos concretos y accionables. No ambiguo ni
condicional.}

## Alternativas consideradas

- **{Alternativa 1}**: {trade-off concreto — costo, riesgo o
  complejidad real de tomar ese camino en lugar del elegido}
- **{Alternativa 2 (opcional, si existe)}**: {trade-off concreto}

## Consecuencias

- {Consecuencia positiva o neutra}
- {Consecuencia negativa o trade-off aceptado — obligatoria}
```

`{NNNN}` se calcula (paso 1), nunca se pide como input. Todo lo demás
entre `{}` viene del contenido recibido de quien invoca el skill, sin
parafrasear ni resumir.

## Pasos obligatorios, en este orden

1. **Numerar.** Listar los archivos `NNNN-*.md` existentes en
   `docs/adr/` (ignorando `.gitkeep`), tomar el número más alto y usar
   `máximo + 1` como número correlativo, formateado a 4 dígitos
   (`0001`, `0002`, ...). Si `docs/adr/` no tiene ningún ADR todavía,
   el número es `0001`.
2. **Verificar las 5 secciones del template.** Si falta Título,
   Contexto, Decisión, Alternativas o Consecuencias en el input
   recibido, detenerse y devolver:
   `ADR rechazado: falta la sección "<sección>".`
   sin generar el archivo ni completar la sección por cuenta propia.
3. **Validar Estado.** Si no se especifica, usar `Propuesto` por
   defecto. Si se especifica un valor fuera de
   `Propuesto`/`Aceptado`/`Deprecado`, rechazar:
   `ADR rechazado: estado "<valor>" no es válido (usar Propuesto, Aceptado o Deprecado).`
4. **Validar que el Contexto sea específico, no genérico.** Rechazar
   si el contexto no menciona ningún elemento concreto del proyecto
   (un criterio numerado de `spec.md`, un componente del stack listado
   en `AGENTS.md`, un ADR previo, o una restricción de negocio
   nombrada) — es decir, si el párrafo podría pegarse sin cambios en
   cualquier otro proyecto. Mensaje de rechazo:
   `ADR rechazado: el contexto es genérico, no referencia ningún elemento concreto de este proyecto (cita un criterio de spec.md, un ADR previo o una restricción real).`
5. **Validar que las alternativas sean reales.** Una alternativa
   cuenta solo si describe un curso de acción distinto al elegido y
   declara al menos un trade-off concreto. "No hacer nada", "mantener
   el status quo" o "dejarlo como está" sin ningún trade-off propio
   asociado NO cuentan. Si no queda ninguna alternativa real tras
   descartar esas, rechazar:
   `ADR rechazado: ninguna alternativa real con trade-off (descartadas las de tipo "no hacer nada" sin trade-off propio).`
6. **Validar que haya consecuencia negativa.** Si todas las
   consecuencias listadas son beneficios o neutras, sin ningún costo
   o trade-off aceptado explícito, rechazar:
   `ADR rechazado: falta al menos una consecuencia negativa o trade-off aceptado.`
7. **Si pasa los pasos 2-6, ensamblar el archivo** siguiendo el
   template exacto, con el contenido recibido sin parafrasear ni
   "mejorar" la decisión o el contexto, y escribirlo en:
   `docs/adr/{NNNN}-{slug-del-titulo}.md`
   (slug en minúsculas, sin acentos, palabras separadas por guiones).
8. **Confirmar al terminar** con el path del archivo creado y el
   número asignado.

## Criterios de aceptación

- Todo archivo generado sigue el template exacto, con las 5 secciones
  variables en ese orden, sin secciones adicionales no solicitadas.
- El número correlativo es siempre `máximo existente en docs/adr/ + 1`;
  nunca se reutiliza ni se deja un hueco.
- Ningún archivo se escribe si el contexto es genérico (sin referencia
  concreta al proyecto), si la única alternativa es "no hacer nada"
  sin trade-off propio, o si Consecuencias no incluye al menos un
  ítem negativo o trade-off aceptado.
- Toda validación fallida produce un mensaje de rechazo concreto (qué
  falta o qué regla se violó), nunca contenido inventado para
  completar el hueco.

## No-goals

- No propone decisiones arquitectónicas ni alternativas: eso es
  trabajo del agente `arquitecto` ([[arquitecto]]). Este skill solo
  formatea y valida lo que ya se decidió.
- No juzga si la decisión tomada es técnicamente buena o mala.
- No edita, reabre ni cambia el estado de ADRs ya existentes en
  `docs/adr/`: solo crea archivos nuevos.
- No completa secciones faltantes ni el contexto por su cuenta: si es
  genérico o incompleto, rechaza y pide la información concreta que
  falta.
