---
name: nueva-prueba-manual
description: Usar cuando se quiere agregar una prueba manual al plan de pruebas del proyecto (docs/pruebas-manuales.md), ya sea invocado por el agente qa al generar el plan completo o por un humano que agrega una prueba suelta. Garantiza el formato canónico (ID correlativo, exactamente 1 criterio de spec.md referenciado, precondición concreta, pasos numerados sin ambigüedad, resultado esperado observable, estado) — rechaza la prueba y señala qué falta o qué regla viola en vez de inventar contenido. No ejecuta pruebas ni decide si pasaron.
argument-hint: <criterio de spec.md> | <precondición> | <pasos numerados> | <resultado esperado> [| <estado>]
---

**Objetivo:** dado el contenido de una prueba manual ya redactada
(provisto por el agente `qa` o por un humano), agregarla a
`docs/pruebas-manuales.md` con el formato canónico del proyecto, ID
correlativo correcto y un mínimo de completitud verificable. Este
skill es un procedimiento, no un rol: no ejecuta la prueba, no decide
si pasó — solo valida y formatea lo que recibe.

## Template exacto de una prueba

```markdown
### Prueba {ID}

**Criterio:** {N} — "{texto exacto del criterio, citado sin parafrasear de spec.md}"

**Precondición:** {estado concreto de datos/sesión inmediatamente antes de ejecutar la prueba}

**Pasos:**

1. {acción puntual, sin ambigüedad, con dato concreto si aplica}
2. {acción puntual, sin ambigüedad, con dato concreto si aplica}

**Resultado esperado:** {observable, verificable con sí/no}

**Estado:** {pendiente|pasada|fallida}
```

`{ID}` se calcula (paso 1), nunca se pide como input. `{N}` y el texto
citado deben corresponder exactamente a un criterio existente en
`spec.md`. Todo lo demás entre `{}` viene del contenido recibido de
quien invoca el skill, sin parafrasear ni "mejorar" la redacción.

## Pasos obligatorios, en este orden

1. **Numerar.** Listar los IDs `PM-NNN` existentes en
   `docs/pruebas-manuales.md` (formato canónico de este skill: bloques
   `### Prueba PM-NNN`), tomar el más alto y usar `máximo + 1` como
   correlativo, formateado a 3 dígitos (`PM-001`, `PM-002`, ...). Si no
   existe ningún bloque con ese formato todavía (archivo vacío o con
   solo entradas previas a este skill), el ID es `PM-001`.
2. **Validar que referencia exactamente 1 criterio.** Leer `spec.md` y
   confirmar que el número de criterio recibido existe, citando su
   texto exacto. Si la prueba no nombra ningún criterio, nombra uno
   que no existe en `spec.md`, o nombra más de uno, detenerse y
   devolver:
   `Prueba rechazada: debe referenciar exactamente 1 criterio de spec.md (recibidos: <0|N>). Si cubre más de un criterio, generar una prueba separada por cada uno.`
   sin decidir por cuenta propia cómo dividirla.
3. **Validar que la precondición sea concreta.** Rechazar si está
   vacía o describe un estado genérico sin datos verificables (ej.
   "el sistema está listo", "todo configurado"). Mensaje de rechazo:
   `Prueba rechazada: la precondición no describe un estado concreto de datos/sesión (falta qué existe o no existe antes de empezar).`
4. **Validar que cada paso sea inequívoco.** Un paso cuenta solo si
   describe una acción puntual con objeto y, cuando corresponda, un
   dato concreto (valores explícitos, no "un valor válido"). Pasos con
   lenguaje vago ("navega un poco", "prueba algo", "hace lo que
   corresponda", "más o menos") NO cuentan. Si algún paso es ambiguo,
   rechazar señalando cuál:
   `Prueba rechazada: el paso <n> ("<texto>") es ambiguo — especificar la acción exacta y el dato concreto.`
5. **Validar que el resultado esperado sea observable.** Debe poder
   verificarse con sí/no sin juicio subjetivo: un texto, valor,
   estado de pantalla o mensaje específico. Resultados como "la app
   funciona bien", "se ve correcto" o "todo ok" sin ningún detalle
   verificable NO cuentan. Si no es observable, rechazar:
   `Prueba rechazada: el resultado esperado ("<texto>") no es observable ni verificable con sí/no — especificar qué se ve exactamente, en qué pantalla, mensaje o valor.`
6. **Validar el estado.** Si no se especifica, usar `pendiente` por
   defecto. Si se especifica un valor fuera de
   `pendiente`/`pasada`/`fallida`, rechazar:
   `Prueba rechazada: estado "<valor>" no es válido (usar pendiente, pasada o fallida).`
   El skill nunca asigna `pasada` o `fallida` por iniciativa propia
   (no ejecuta la prueba ni evalúa su resultado): solo registra el
   estado que se le indique explícitamente, o `pendiente` si no se
   indica ninguno.
7. **Si pasa los pasos 2-6, ensamblar el bloque** siguiendo el
   template exacto, con el contenido recibido sin parafrasear, y
   agregarlo a `docs/pruebas-manuales.md` bajo la sección temática
   correspondiente al criterio (la misma agrupación usada en el
   archivo, ej. "Autenticación", "Estadísticas"), después de cualquier
   otra prueba existente para ese mismo criterio.
8. **Confirmar al terminar** con el ID asignado y el número de
   criterio referenciado.

## Criterios de aceptación

- Toda prueba agregada sigue el template exacto, con los 5 campos
  (Criterio, Precondición, Pasos, Resultado esperado, Estado) en ese
  orden, sin campos adicionales no solicitados.
- El ID correlativo es siempre `máximo existente en formato PM-NNN + 1`;
  nunca se reutiliza ni se deja un hueco.
- Ninguna prueba escrita referencia 0 criterios ni más de 1; si el
  input cubre varios, se generan pruebas separadas (una invocación del
  skill por prueba).
- Ningún paso ambiguo ni resultado esperado subjetivo llega a
  escribirse: toda validación fallida produce un mensaje de rechazo
  concreto, nunca contenido inventado para completar el hueco.
- El campo Estado es `pendiente` salvo que el input indique
  explícitamente `pasada` o `fallida`; el skill nunca lo decide por
  su cuenta.

## No-goals

- No ejecuta pruebas: no abre la aplicación, no interactúa con el
  sistema bajo prueba, no compara un resultado real contra el
  esperado.
- No decide si una prueba pasó o falló: solo registra el estado que
  se le indica explícitamente (o `pendiente` por defecto). Esa
  decisión es de la persona que ejecuta la prueba.
- No genera tests automatizados (unitarios, e2e, de integración).
- No propone ni redacta criterios de aceptación: eso vive en
  `spec.md`. Si un criterio referenciado no existe ahí, rechaza en vez
  de inventarlo.
- No corrige resultados esperados vagos ni inventa datos para volver
  verificable un paso ambiguo: rechaza y pide la información concreta
  que falta a quien invocó el skill (el agente `qa` o el humano).
