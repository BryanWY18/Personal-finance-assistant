# ADR 0001: Modelo de datos — tipo de transacción snapshotted en Record

## Estado

Aceptado

## Contexto

El spec (criterios #20, #25) requiere separar gastos e ingresos y calcular
un balance neto en estadísticas. "Ingresos" es una categoría fija más, y
las categorías personalizadas necesitan declarar si son de tipo ingreso o
gasto. El criterio #21 establece que al borrar una categoría personalizada
con registros asociados, esos registros se reasignan a la categoría fija
"Sin categoría" (tipo gasto). Si el tipo de un registro se derivara siempre
en vivo desde su categoría actual, ese borrado distorsionaría
retroactivamente las estadísticas históricas: un ingreso pasado pasaría a
contar como gasto.

## Decisión

- Cada `Category` (fija o personalizada) tiene un campo `tipo`
  (`ingreso` | `gasto`). Categorías fijas tipo `gasto`: Comida,
  Transporte, Vivienda, Entretenimiento, Sin categoría. La única fija
  tipo `ingreso`: Ingresos. Al crear una categoría personalizada, el
  usuario elige su tipo.
- Cada `Record` guarda su propio campo `tipo`, copiado de
  `category.tipo` en el momento de la creación del registro. Las
  estadísticas siempre usan el `tipo` del registro, nunca el de la
  categoría actual.
- Entidades: `User`, `Category`, `Record`, `TelegramLinkCode` (detalle
  de campos en `spec.md` y `AGENTS.md`).

## Consecuencias

- Pequeña redundancia de datos (`tipo` vive en dos lugares), pero el
  balance neto histórico no se distorsiona si se borra o reclasifica
  una categoría.
- Si el tipo de una categoría cambia después de creada (no contemplado
  en el spec actual), los registros existentes no se actualizan
  automáticamente — comportamiento esperado, no un bug.
