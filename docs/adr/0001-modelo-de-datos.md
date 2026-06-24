# ADR 0001: Modelo de datos — tipo de registro derivado de la categoría

## Estado

Aceptado

## Contexto

La spec (criterios #18, #19, #25) define "Ingresos" como una categoría
fija más, junto con Comida, Transporte, Vivienda, Entretenimiento y
Sin categoría. Las categorías personalizadas se crean con nombre único
(#19) pero la spec no contempla que el usuario les asigne un tipo: hoy
la única categoría de tipo ingreso es la fija "Ingresos". La vista de
estadísticas exige separar gastos e ingresos y calcular balance neto
(#25), y al borrar una categoría personalizada con registros asociados
esos registros se reasignan a "Sin categoría" (#21).

## Decisión

- `Record` no tiene campo `tipo`. El tipo de un registro se infiere de
  su `categoria_id`: si la categoría es la fija "Ingresos" (id
  reservado, no editable desde la web — #18), el registro es ingreso;
  cualquier otra categoría (fija o personalizada) es gasto.
- Las queries de estadísticas que separan ingresos de gastos filtran
  por ese id reservado de "Ingresos".
- Al borrar una categoría personalizada con registros asociados (#21),
  los registros se reasignan a `categoria_id` = "Sin categoría" y
  automáticamente pasan a comportarse como gasto, sin tocar ningún
  otro campo.

## Consecuencias

- Evita una columna redundante y el riesgo de que `categoria_id` y un
  campo `tipo` queden desincronizados.
- Toda query que separe gastos/ingresos depende de que el id de la
  categoría "Ingresos" sea estable y esa categoría no sea borrable.
- Si en el futuro se permite que una categoría personalizada sea de
  tipo ingreso (no contemplado en la spec actual), esta decisión
  requiere revisión y probablemente migración estructural.
