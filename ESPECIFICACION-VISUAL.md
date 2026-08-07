# Ritmo — Especificación visual

Traducción del diseño generado en Google Stitch a valores concretos, **con las correcciones
necesarias** para que funcione sin conexión, bajo sol directo, y sin perder funcionalidad.

> Este trabajo va **después** de la arquitectura de datos. No mezclar ambos en un mismo commit.

---

## 1. Reglas no negociables

Antes de cualquier detalle estético, tres restricciones que el diseño original de Stitch viola
y que hay que corregir al implementar:

**1.1 Cero dependencias de red.** Ritmo se usa en zona rural sin señal. Está prohibido:
`cdn.tailwindcss.com`, `fonts.googleapis.com`, `fonts.gstatic.com`, Material Symbols, o
cualquier `<script src>` o `<link href>` externo. Todo va en el repositorio.

- **Tipografía:** descargar los `.woff2` de Plus Jakarta Sans (pesos 400, 600, 700) a
  `fonts/` y declararlos con `@font-face` y `font-display: swap`. Añadirlos a la lista
  `ARCHIVOS` del service worker.
- **Iconos:** reemplazar Material Symbols por SVG inline con `stroke="currentColor"`.
- **CSS:** escrito a mano en el `<style>` de `index.html`. No introducir Tailwind.

**1.2 Solo móvil.** Stitch generó variantes de escritorio con rejillas tipo bento y una barra
de navegación superior. Descartarlas por completo. Ritmo es una app de una sola columna en
vertical. Nada de `md:` ni breakpoints de escritorio.

**1.3 Nada de interfaz muerta.** Eliminar el avatar de perfil y la campana de notificaciones
que Stitch añadió en la cabecera. Ritmo no tiene cuentas ni notificaciones. La cabecera
superior completa se elimina: el espacio lo ocupa la tarjeta de ritmo.

---

## 2. Fichas de diseño

Declarar todo como variables CSS en `:root`. Cambiar la paleta completa debe ser cuestión de
editar este bloque y nada más.

```css
:root{
  /* Fondo */
  --fondo:            #FDFDFF;
  --destello-rosa:    rgba(246, 217, 250, 0.40);
  --destello-menta:   rgba(194, 236, 210, 0.40);

  /* Superficies de vidrio */
  --vidrio:           rgba(255, 255, 255, 0.62);
  --vidrio-denso:     rgba(255, 255, 255, 0.74);
  --vidrio-blur:      32px;
  --borde-claro:      rgba(255, 255, 255, 0.80);

  /* Texto */
  --tinta:            #201731;
  --tinta-ciruela:    #2D1B33;
  --tinta-suave:      #4B454B;

  /* Acentos */
  --salvia:           #3E6B55;
  --salvia-clara:     #A6D0B7;
  --salvia-tenue:     #C2ECD2;
  --ambar:            #9C6B14;
  --ambar-tenue:      #F7E7C8;
  --granate:          #A8323F;
  --granate-tenue:    #FAD9DC;
  --lila:             #F6D9FA;
  --lila-dim:         #D9BEDE;
  --neutro:           #7D757C;

  /* Geometría */
  --r-tarjeta: 22px;  --r-campo: 12px;  --r-pildora: 9999px;
  --e-xs: 4px; --e-sm: 8px; --e-md: 16px; --e-lg: 24px; --e-xl: 32px;
  --margen: 20px;
}
```

### Nota sobre los tres colores de estado

Stitch no entregó una escala de estados: usó ciruela para todo y el rojo genérico `#ba1a1a`
para errores. Los tres valores de arriba (`--salvia`, `--ambar`, `--granate`) están escogidos
para pertenecer a la misma familia pastel pero con suficiente saturación y oscuridad para
leerse a pleno sol. **No aclararlos.**

### Fondo del documento

```css
body{
  background-color: var(--fondo);
  background-image:
    radial-gradient(circle at 15% 50%, var(--destello-rosa),  transparent 40%),
    radial-gradient(circle at 85% 30%, var(--destello-menta), transparent 40%);
  background-attachment: fixed;
}
```

### Tarjeta de vidrio

```css
.vidrio{
  position: relative;
  background: var(--vidrio);
  backdrop-filter: blur(var(--vidrio-blur));
  -webkit-backdrop-filter: blur(var(--vidrio-blur));
  border-radius: var(--r-tarjeta);
  border: 1px solid transparent;
  background-clip: padding-box;
}
.vidrio::before{
  content:""; position:absolute; inset:-1px; z-index:-1;
  border-radius: inherit;
  background: linear-gradient(135deg,
    rgba(255,255,255,.85), var(--destello-rosa), var(--destello-menta), rgba(255,255,255,.45));
  -webkit-mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor; mask-composite: exclude;
  pointer-events: none;
}
```

Stitch usó opacidad `0.45` en el vidrio. Aquí se sube a `0.62` porque a `0.45` los destellos
de color atraviesan la tarjeta y bajan el contraste del texto por debajo de lo legible al sol.
El efecto de vidrio se conserva; solo pierde un poco de transparencia.

---

## 3. Tipografía

Plus Jakarta Sans, autoalojada. Escala tal cual la definió Stitch:

| Rol | Tamaño / interlínea | Peso | Espaciado | Uso |
|---|---|---|---|---|
| `display` | 48 / 56 | 700 | -0.02em | Monto grande de la cabecera |
| `titular` | 28 / 36 | 700 | -0.01em | Montos de fondos |
| `titulo` | 20 / 28 | 600 | — | Títulos de tarjeta |
| `cuerpo` | 16 / 24 | 400 | — | Texto general |
| `menor` | 14 / 20 | 400 | — | Notas, secundarios |
| `versalita` | 12 / 16 | 700 | 0.05em, mayúsculas | Microetiquetas |

**Todas las cifras de dinero** llevan `font-variant-numeric: tabular-nums;` para que los
dígitos no bailen al actualizarse. Aplicarlo también a los porcentajes.

Formato de moneda: `$1.284.500`. Punto como separador de miles, sin decimales.
En el diseño de Stitch aparecen valores con coma (`$45,000`); es error de la herramienta.

---

## 4. La barra de ritmo

**Es el elemento central de la app y Stitch lo implementó mal.** En su versión el relleno y la
marca de "HOY" están ambos en 38%, lo que la convierte en una barra de progreso normal con un
adorno encima. También puso la marca en blanco sobre una pista casi blanca: invisible.

### Qué significa cada parte

- **Relleno** = `gastado del mes ÷ presupuesto total del mes`
- **Marca HOY** = `día actual ÷ días del mes`

Son dos cantidades independientes. Que coincidan es casualidad, no diseño.
La lectura completa es: *si el relleno no alcanza la marca, vas bien; si la pasó, vas gastando
más rápido de lo que el mes aguanta.*

### Especificación

| Elemento | Valor |
|---|---|
| Alto de la pista | 36px, radio completo |
| Fondo de la pista | `rgba(45, 27, 51, 0.12)` |
| Relleno | color según estado, radio completo a la izquierda, transición 0.5s |
| Marca | 2px de ancho, color `var(--tinta-ciruela)`, sobresale 3px arriba y abajo |
| Etiqueta `HOY` | 9px, 700, mayúsculas, `var(--tinta-ciruela)`, centrada sobre la marca |

**La marca es ciruela oscura, no blanca.** Debe contrastar contra la pista, que es clara.

### Estados

| Condición | Color del relleno | Texto de veredicto |
|---|---|---|
| relleno por detrás de la marca | `var(--salvia)` | `Vas $X por debajo` |
| relleno pasó la marca | `var(--ambar)` | `Vas $X adelantado` |
| gastado ≥ presupuesto total | `var(--granate)` | `Presupuesto superado` |

El texto del veredicto va en peso 700 y del mismo color que el relleno.
Debajo, a la izquierda y en `--tinta-suave`: `A esta altura del mes: $1.420.000`.

Si no hay presupuestos definidos, ocultar la barra y mostrar en su lugar:
`Define tus presupuestos en Ajustes`.

---

## 5. Pantallas

Padding lateral `var(--margen)`. Separación entre tarjetas `var(--e-md)`.
Padding interno de tarjeta `var(--e-lg)`. Espacio inferior para la barra flotante: 120px.

### 5.1 Registrar

1. **Tarjeta de ritmo.** Microetiqueta `AGOSTO 2026 · GASTADO`, monto en `display`,
   línea menor `14 movimientos · día 12 de 31`, y la barra de la sección 4.
2. **Tarjeta de captura.**
   - Control segmentado de **tres** opciones: `Gasto` · `Ingreso` · `Aporte a fondo`.
     Stitch entregó solo dos y eliminó los aportes; hay que restituir la tercera.
     Fondo del riel `rgba(255,255,255,.30)`, opción activa en blanco sólido con sombra suave.
   - Microetiqueta `MONTO` y campo grande con prefijo `$`, alineado a la izquierda,
     tipografía `display`, separadores de miles mientras se escribe.
   - Microetiqueta que cambia con el tipo: `CATEGORÍA` / `ORIGEN` / `FONDO`.
   - Chips en fila con desplazamiento horizontal. Inactivo: `rgba(255,255,255,.55)` con
     borde claro y texto `--tinta-ciruela`. Activo: fondo `--tinta-ciruela`, texto blanco.
     Sin iconos dentro del chip: con ocho o más categorías el ancho se vuelve inmanejable.
   - Microetiqueta `NOTA CORTA` y campo de texto.
     **Stitch eliminó este campo. Restituirlo.**
   - Botón `Guardar movimiento`, ancho completo, fondo `--tinta-ciruela`, texto blanco,
     radio `--r-campo`, alto 52px.
3. **Tarjeta `Hoy`** con los movimientos del día.

### 5.2 Mes

En una sola columna, en este orden: `Por categoría`, `Ritmo diario`, `Balance del mes`,
`Todos los movimientos`. Descartar la rejilla bento de Stitch.

- **Por categoría:** nombre en 600 a la izquierda; a la derecha el gasto en negrita seguido de
  `/ presupuesto` en `--tinta-suave`. Debajo, barra de 8px con el color de estado. Insignia
  `80%` en ámbar o `Excedido` en granate junto al nombre cuando corresponda.
- **Ritmo diario:** una barra por día del mes. Barras en `--salvia-clara` **a opacidad plena**
  (Stitch las puso al 40%: ilegible al sol). La barra de hoy en `--tinta-ciruela`.
  Etiquetas `Día 1` y `Día 31` debajo.
- **Balance del mes:** cuatro filas con divisor de 1px `rgba(255,255,255,.45)`.
  Etiqueta a la izquierda, cifra en negrita a la derecha.

### 5.3 Fondos

Tarjeta de encabezado con el título `Mis fondos` y la línea explicativa. Luego una subtarjeta
por fondo: nombre en 600 y acumulado en `titular` a la derecha; barra de progreso de 8px en
`--salvia`; debajo, porcentaje a la izquierda y `Falta $X` a la derecha en `menor`.

Los iconos circulares que Stitch puso en cada fondo son opcionales. Si se conservan, deben ser
SVG inline; si complican, eliminarlos sin pérdida.

### 5.4 Ajustes

Una columna. Tarjetas en este orden: `Categorías`, `Fondos`, `Orígenes de ingreso`,
`Copia de seguridad`. El contenido de las tres primeras está definido en
`ARQUITECTURA-DATOS.md`, sección 6.

Botones de respaldo, ancho completo y apilados:

| Botón | Estilo |
|---|---|
| `Descargar mis datos` | fondo `--salvia-tenue`, texto `--salvia` |
| `Restaurar desde archivo` | fondo `--salvia-tenue`, texto `--salvia` |
| `Borrar todo` | fondo `--granate-tenue`, texto `--granate` |

### 5.5 Barra de navegación

Píldora flotante: `bottom: var(--e-lg)`, ancho 90%, máximo 420px, centrada.
Fondo `rgba(255,255,255,.55)`, blur 32px, borde `--borde-claro`, radio completo.
Cuatro destinos: `Registrar`, `Mes`, `Fondos`, `Ajustes`.

Icono SVG de trazo (1.9px) sobre etiqueta de 11px en peso 700.
Activo: fondo `--lila` en píldora interna y texto `--tinta-ciruela`.
Inactivo: texto `--tinta-suave`.

Respetar `env(safe-area-inset-bottom)`.

---

## 6. Movimiento

Discreto. La app se abre decenas de veces al día; una animación que encanta la primera vez
estorba la número cuarenta.

- Relleno de la barra de ritmo: 0.5s `cubic-bezier(.4,0,.2,1)`.
- Barras de categoría: 0.4s.
- Botones y chips al pulsar: `scale(0.98)`, 0.1s.
- Cambio de pantalla: sin transición.
- Envolver todo en `@media (prefers-reduced-motion: reduce){ *{transition:none!important} }`.

---

## 7. Verificación antes del commit

- [ ] Modo avión, app abierta desde el ícono: carga y funciona completa.
- [ ] Buscar en `index.html`: cero coincidencias de `googleapis`, `cdn.`, `http://`, `https://`.
- [ ] La marca de `HOY` y el relleno están en posiciones distintas cuando los valores difieren.
- [ ] Los tres estados de la barra se ven correctos (probar cambiando presupuestos a la baja).
- [ ] El campo de nota existe y se guarda.
- [ ] El segmentado tiene las tres opciones y `Aporte a fondo` registra en el fondo correcto.
- [ ] Ninguna cifra usa coma como separador de miles.
- [ ] Prueba al aire libre a mediodía: todos los textos y barras se leen.
- [ ] Versión del caché subida en `sw.js`.
