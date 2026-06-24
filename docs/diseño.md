# Sistema de diseño — Personal Finance Assistant

Documento de decisiones visuales derivado de `spec.md` (criterios de
aceptación) y restringido por `AGENTS.md` (Tailwind CSS como única
librería de estilos, shadcn/ui permitido, prohibidas librerías de
componentes con sistema de diseño propio como Material UI, Chakra UI
o Ant Design). Todas las decisiones de este documento son
implementables solo con Tailwind y primitivas de shadcn/ui.

No incluye modo oscuro (spec.md no lo exige), ni animaciones o
transiciones más allá de los estados por defecto de Tailwind
(hover/focus/disabled).

---

## Paleta de colores

La app maneja dinero (gastos vs. ingresos) y estados de servicio
(caída de Telegram/n8n, errores de validación de montos y emails
duplicados — criterios 4, 5, 10, 14, 15). Esto exige distinguir con
claridad acción, error y éxito además de fondo y texto. Se proponen
5 colores.

| Rol | Color | Valor (Tailwind) | Justificación |
|---|---|---|---|
| Primario | Azul índigo | `indigo-600` (`#4f46e5`) | Es el color de toda acción principal: guardar registro, confirmar login, vincular Telegram. Un azul saturado se asocia a confianza y estabilidad financiera, no compite con el verde de "ingreso" ni el rojo de "gasto/error", y tiene suficiente contraste sobre fondo blanco para cumplir AA en texto de botón. |
| Fondo | Gris muy claro | `slate-50` (`#f8fafc`) | Fondo neutro que no distrae de los números, que son el contenido real de la app (montos, balances, gráficos). Evita el blanco puro (`white`) para que las `card` con fondo blanco tengan separación visual sin necesidad de bordes pesados. |
| Texto | Gris oscuro casi negro | `slate-900` (`#0f172a`) | Máximo contraste legible sobre `slate-50` y sobre blanco; un negro puro (`#000`) resulta agresivo en pantallas largas de tablas de registros, mientras que `slate-900` mantiene legibilidad sin fatiga visual. |
| Error | Rojo | `red-600` (`#dc2626`) | Usado en mensajes de validación explícitos del spec ("Se debe ingresar monto...", email duplicado, login fallido, servicio caído) y para representar gastos en los gráficos de estadísticas. Rojo es la convención universal de error y de salida de dinero, reduce la carga cognitiva del usuario. |
| Éxito | Verde | `green-600` (`#16a34a`) | Usado para confirmaciones (registro creado, cuenta vinculada, servicio restablecido) y para representar ingresos en los gráficos de estadísticas, en oposición directa al rojo de gastos. Mantener verde/rojo como par fijo ingreso/gasto en toda la app evita que el usuario tenga que reaprender el código de color entre el dashboard y las estadísticas.

Reglas de uso fijas:

- `indigo-600` solo para acciones primarias (botones de submit,
  enlaces de navegación activos). Nunca se usa para indicar
  error o éxito.
- `green-600` y `red-600` se reservan exclusivamente para
  ingresos/éxito y gastos/error respectivamente; no se usan como
  color decorativo en ningún otro componente.
- Todo texto sobre `indigo-600`, `red-600` o `green-600` es blanco
  (`white`) para garantizar contraste.

---

## Stack tipográfico

Una sola familia: **system font stack** de Tailwind
(`font-sans` por defecto: `ui-sans-serif, system-ui, -apple-system,
"Segoe UI", Roboto, Helvetica, Arial, sans-serif`), usada tanto para
títulos como para cuerpo de texto, diferenciando jerarquía solo con
`font-weight` y `font-size` (sin segunda familia para títulos).

Justificación: la app es un dashboard financiero de uso utilitario
—tablas, formularios, montos, gráficos— donde la prioridad es
legibilidad y velocidad de carga, no personalidad tipográfica; el
stack de sistema no requiere fetch de fuentes externas (cero
latencia, cero layout shift) y ya está optimizado por cada sistema
operativo. No se introduce una Google Font porque no hay un
requisito de marca en `spec.md` que lo justifique.

---

## Escala de espaciado

Escala basada en los múltiplos estándar de Tailwind
(`1 = 0.25rem`). Se fija un uso concreto por paso para evitar que
cada implementador decida espaciados ad-hoc.

| Paso Tailwind | Valor rem | Uso asignado |
|---|---|---|
| `1` | 0.25rem | Espacio entre ícono y texto dentro de un mismo control (ej. ícono dentro de un botón). |
| `2` | 0.5rem | Gap entre label y su input dentro de un campo de formulario. |
| `3` | 0.75rem | Gap entre inputs consecutivos dentro de un mismo formulario (ej. categoría → descripción → monto). |
| `4` | 1rem | Padding interno de un `card` (estadísticas, fila de registro) y gap entre botones de una misma barra de acciones. |
| `6` | 1.5rem | Padding interno de contenedores grandes (formulario completo dentro de su card) y gap entre la card de gastos y la card de ingresos en estadísticas. |
| `8` | 2rem | Separación entre secciones dentro de una misma página (ej. entre el selector de período y los gráficos). |
| `12` | 3rem | Separación entre el header de navegación y el contenido principal de la página. |
| `16` | 4rem | Margen vertical superior/inferior de páginas de autenticación (login/registro) respecto al borde del viewport, para centrar el formulario sin que quede pegado a los bordes. |

---

## Componentes UI

Lista cerrada de componentes necesarios según los criterios de
aceptación de `spec.md`. Todos se construyen sobre primitivas de
shadcn/ui (que internamente usa Tailwind) o con Tailwind puro cuando
no hay primitiva equivalente.

| Componente | Variante shadcn/ui base | Uso derivado de spec.md |
|---|---|---|
| `Button` | `button` (variantes `default`, `destructive`, `outline`) | Submit de formularios (login, registro, nuevo registro financiero), botón de borrar registro (`destructive`), botón de logout (`outline`). |
| `Input` | `input` | Campos de email, contraseña, descripción y monto en formularios de auth y de registros (criterios 1, 8, 9, 10). |
| `Label` | `label` | Etiqueta de cada campo de formulario, asociada por `htmlFor` para accesibilidad. |
| `Select` | `select` | Selector de categoría al crear/editar un registro (criterio 8, 13) y selector de modo quincenal/mensual en estadísticas (criterios 22-24). |
| `Form` (wrapper de validación) | `form` + `react-hook-form` (compatible con shadcn/ui) | Estructura de validación de los formularios de login, registro de cuenta, nuevo registro financiero y nueva categoría, mostrando mensajes de error inline (criterios 1, 4, 5, 10, 20). |
| `Card` | `card` | Contenedor de cada fila/bloque de registro en el listado, contenedor de cada gráfico de estadísticas, contenedor del formulario de vinculación de Telegram. |
| `Table` | `table` | Listado de registros financieros con columnas categoría/descripción/monto/fecha y acciones de editar/borrar (criterios 8, 11, 12). |
| `Dialog` | `dialog` | Confirmación antes de ejecutar el borrado físico de un registro o de una categoría personalizada (criterios 12, 21), dado que el borrado es irreversible y sin papelera. |
| `Badge` | `badge` | Etiqueta visual de categoría junto a cada registro en el listado y en los gráficos de estadísticas. |
| `Alert` | `alert` (variantes `default`, `destructive`) | Mensajes de error de validación (monto faltante, email duplicado, credenciales incorrectas) y mensajes de estado del servicio (caído/restablecido) replicados en la web cuando aplique (criterios 4, 5, 10, 14, 15). |
| `Tabs` | `tabs` | Selector de período quincenal/mensual en la vista de estadísticas, como alternativa visual al `Select` cuando solo hay dos opciones (criterios 22-24). |
| `Chart` (gráfico de barras) | Construido con Tailwind + SVG nativo (sin librería de charts externa); estructura visual vía `card` | Los dos gráficos continuos de estadísticas: uno de gastos por categoría, uno de ingresos por categoría, más balance neto (criterio 25). Gastos siempre en `red-600`, ingresos siempre en `green-600`. |
| `Skeleton` | `skeleton` | Estado de carga de la tabla de registros y de los gráficos mientras se obtienen datos del backend, evitando saltos de layout. |
| `Toast` | `toast`/`sonner` | Confirmación efímera tras crear, editar o borrar un registro, y tras vincular Telegram exitosamente. |
| `Empty state` (bloque de texto + ícono, sin librería) | Construido con Tailwind puro | Mensaje "sin registros en este período" cuando las estadísticas no tienen datos (criterio 26), evitando mostrar gráficos vacíos sin contexto. |
| `Navbar` | Construido con Tailwind puro sobre `button`/`link` de shadcn/ui | Navegación entre Dashboard, Registros, Categorías, Estadísticas, Vinculación de Telegram y Logout, visible solo en rutas autenticadas. |

---

## Páginas

Rutas requeridas por `spec.md`, cada una con sus secciones y los
componentes que usa del inventario anterior.

### `/login`

- **Sección formulario de acceso**: campos email y contraseña,
  botón de submit, enlace a `/registro`.
  Componentes: `Card`, `Form`, `Input`, `Label`, `Button`,
  `Alert` (credenciales incorrectas, criterio 5).

### `/registro`

- **Sección formulario de alta de cuenta**: campos email y
  contraseña con reglas de validación visibles (mínimo 6
  caracteres, 1 mayúscula, 1 número, 1 carácter especial),
  botón de submit, enlace a `/login`.
  Componentes: `Card`, `Form`, `Input`, `Label`, `Button`,
  `Alert` (email duplicado, criterio 4).

### `/dashboard`

- **Sección de bienvenida y resumen rápido**: balance neto del
  período actual y accesos directos a Registros y Estadísticas.
  Componentes: `Navbar`, `Card`, `Badge`, `Button`.

### `/registros`

- **Sección listado de registros**: tabla con categoría,
  descripción, monto y acciones de editar/borrar.
  Componentes: `Navbar`, `Table`, `Badge`, `Button`, `Dialog`
  (confirmación de borrado, criterio 12), `Skeleton`, `Empty state`.
- **Sección nuevo/editar registro**: formulario con categoría
  (`Select`), descripción (`Input`, opcional con default "Sin
  descripción"), monto (`Input`, validación mínimo $1, sin
  negativos, decimales permitidos).
  Componentes: `Form`, `Select`, `Input`, `Label`, `Button`,
  `Alert` (monto faltante, criterio 10), `Toast`.

### `/categorias`

- **Sección categorías fijas**: listado de solo lectura de
  Comida, Transporte, Vivienda, Entretenimiento, Sin-Categoría e
  Ingresos.
  Componentes: `Navbar`, `Card`, `Badge`.
- **Sección categorías personalizadas**: listado editable/borrable
  de categorías propias del usuario, formulario de alta con
  validación de nombre único case-insensitive.
  Componentes: `Form`, `Input`, `Label`, `Button`, `Dialog`
  (confirmación de borrado con aviso de reasignación a
  "Sin categoría", criterio 21), `Alert` (nombre duplicado,
  criterio 20), `Toast`.

### `/estadisticas`

- **Sección selector de período**: alternancia entre modo
  quincenal y modo mensual.
  Componentes: `Navbar`, `Tabs`.
- **Sección totales**: total de gastos, total de ingresos y
  balance neto del período seleccionado.
  Componentes: `Card`, `Badge` (verde para ingresos, rojo para
  gastos).
- **Sección gráficos**: dos gráficos continuos separados, gastos
  por categoría e ingresos por categoría.
  Componentes: `Card`, `Chart` (uno en `red-600`, otro en
  `green-600`), `Empty state` (criterio 26), `Skeleton`.

### `/telegram`

- **Sección de vinculación**: instrucciones para iniciar
  conversación con el bot vía `/start`, estado de vinculación
  actual (vinculado/no vinculado), y confirmación tras completar
  el proceso.
  Componentes: `Navbar`, `Card`, `Alert` (estado no vinculado con
  instrucción), `Badge` (estado vinculado), `Toast` (confirmación
  de vinculación exitosa, criterio 6).
