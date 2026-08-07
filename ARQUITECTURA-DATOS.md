# Ritmo — Arquitectura de datos

Documento de referencia para Claude Code. Define cómo deben funcionar las categorías,
fondos y orígenes editables **sin romper los movimientos ya registrados**.

> Léelo completo antes de modificar `index.html`. Este trabajo va **antes** del rediseño visual.

---

## 1. El problema que hay que evitar

Hoy las categorías están escritas a mano en el arreglo `CATEGORIAS` dentro de `index.html`.
Cada movimiento guardado apunta a una categoría por su `id` (`mercado`, `pareja`, …).

Si el usuario puede crear, renombrar y borrar categorías, aparecen tres formas de perder datos:

| Riesgo | Qué pasa | Regla que lo evita |
|---|---|---|
| Renombrar cambia el `id` | Los movimientos viejos quedan huérfanos | El `id` es **inmutable**. Renombrar toca solo `nom`. |
| Borrar una categoría en uso | Sus movimientos desaparecen del desglose | No se borra: se **archiva**. Solo se borra si no tiene movimientos. |
| Restaurar un respaldo viejo | El archivo no tiene las categorías nuevas | Migración por versión de esquema. |

**Principio rector: un movimiento registrado nunca se pierde ni se altera.** Ante cualquier
duda, la app conserva el dato y lo muestra como "Sin categoría" antes que descartarlo.

---

## 2. Modelo de datos

### Claves en `localStorage`

| Clave | Contenido |
|---|---|
| `rt_schema` | Número de versión del esquema. Actual: `2` |
| `rt_movs` | Arreglo de movimientos |
| `rt_cats` | Arreglo de categorías de gasto |
| `rt_fondos` | Arreglo de fondos |
| `rt_origenes` | Arreglo de orígenes de ingreso |

Las claves `rt_pres` y `rt_metas` quedan **obsoletas**: el presupuesto pasa a vivir dentro de
cada categoría y la meta dentro de cada fondo. Solo se leen durante la migración.

### Movimiento (sin cambios de forma)

```js
{
  id: 1723400000000,      // marca de tiempo, inmutable
  fecha: "2026-08-12",    // ISO corto, zona local
  tipo: "gasto",          // "gasto" | "ingreso" | "aporte"
  destino: "mercado",     // id de categoría, fondo u origen según el tipo
  monto: 45000,           // entero, pesos sin decimales
  nota: "plátano y frutas"
}
```

### Categoría

```js
{
  id: "mercado",          // INMUTABLE. Nuevas: "cat_" + Date.now() + sufijo aleatorio
  nom: "Mercado",         // editable
  color: "#406652",       // editable, hex
  presupuesto: 200000,    // editable, entero, 0 = sin presupuesto
  orden: 2,               // posición en la lista de chips
  activa: true            // false = archivada
}
```

### Fondo

```js
{ id: "independencia", nom: "Independencia", meta: 40000000, orden: 0, activa: true }
```

### Origen de ingreso

```js
{ id: "contrato", nom: "Contrato EBS", orden: 0, activa: true }
```

### Semillas iniciales

Los `id` de las semillas **deben conservarse exactamente** como están hoy en el código,
o los movimientos ya registrados en el celular quedan huérfanos:

- Categorías: `ley`, `transporte`, `mercado`, `servicios`, `pareja`, `personal`, `salud`, `imprevisto`
- Fondos: `independencia`, `emprendimiento`, `reserva`, `contingencia`
- Orígenes: `contrato`, `extra`, `otro`

---

## 3. Migración

Al arrancar la app, antes de cualquier render:

```
leer rt_schema (si no existe, asumir 1)

si version < 2:
    si existe rt_cats  -> usarlo
    si no:
        construir rt_cats desde las semillas del código
        para cada categoría, tomar su presupuesto de rt_pres[id] si existe
    lo mismo para rt_fondos usando rt_metas[id]
    construir rt_origenes desde las semillas
    escribir rt_schema = 2
    NO borrar rt_pres ni rt_metas todavía (red de seguridad por si algo falla)
```

La migración debe ser **idempotente**: correrla dos veces no puede duplicar ni alterar nada.

---

## 4. Reglas de comportamiento

### Crear

- Nombre obligatorio, entre 1 y 24 caracteres.
- No se permiten dos categorías activas con el mismo nombre (comparación sin distinguir
  mayúsculas ni tildes). Mensaje: `Ya existe una categoría con ese nombre.`
- `id` generado, nunca derivado del nombre. Dos categorías pueden llamarse igual en momentos
  distintos de la historia sin colisionar.
- `orden` = el mayor existente + 1.

### Renombrar

- Cambia `nom` y nada más. El `id` permanece. Los movimientos históricos pasan a mostrarse
  con el nombre nuevo, lo cual es el comportamiento correcto.

### Archivar

- Es la acción por defecto al intentar eliminar una categoría que tiene movimientos.
- `activa: false`. Deja de aparecer en los chips de Registrar.
- **Sigue apareciendo** en la pantalla Mes si tiene movimientos en el mes en curso, y siempre
  en el historial de movimientos.
- Se puede reactivar desde Ajustes.

### Eliminar de verdad

- Solo si **ningún** movimiento la referencia. La app lo verifica antes de ofrecer la opción.
- Diálogo: `Esta categoría no tiene movimientos. Se eliminará de forma permanente. ¿Continuar?`

### Reasignar

- Al archivar una categoría con movimientos, ofrecer opcionalmente moverlos a otra categoría.
- Si el usuario acepta, actualizar el campo `destino` de esos movimientos y luego permitir el
  borrado definitivo.
- Diálogo: `Mover los N movimientos de "X" a otra categoría antes de archivarla.`

### Categorías huérfanas

- Si un movimiento apunta a un `destino` que no existe en ninguna lista, mostrarlo como
  **"Sin categoría"** con color `#7d757c`.
- Nunca excluirlo de los totales del mes ni del balance.
- En Ajustes, si hay huérfanos, mostrar un aviso con la opción de reasignarlos.

Las mismas reglas aplican a fondos y orígenes.

---

## 5. Respaldo y restauración

El archivo exportado cambia de forma:

```js
{
  version: 2,
  exportado: "2026-08-12T14:30:00.000Z",
  movs: [...], cats: [...], fondos: [...], origenes: [...]
}
```

Al importar:

- Si `version` es 1 (formato viejo con `pres` y `metas`), aplicar la misma migración de la
  sección 3 antes de cargar. Un respaldo antiguo **debe** poder restaurarse.
- Si falta `movs` o no es un arreglo, rechazar con:
  `El archivo no tiene el formato esperado.`
- Si el archivo trae categorías que no existen localmente, **añadirlas**, no reemplazar la lista.
- La importación es un reemplazo completo del estado. Advertir antes:
  `Se reemplazarán todos los datos actuales por los del archivo. ¿Continuar?`

---

## 6. Interfaz de gestión (dentro de Ajustes)

Una sección **"Categorías"** con la lista ordenable. Cada fila muestra: punto de color, nombre,
presupuesto, y un control para editar. Al editar se abre un panel con:

- Campo de nombre
- Selector de color (paleta fija de 8 opciones, no un selector libre)
- Campo de presupuesto mensual
- Botón `Archivar` o `Eliminar` según tenga o no movimientos

Al final de la lista, un botón `Añadir categoría`.

Debajo, plegado por defecto, un bloque **"Archivadas"** con la opción `Reactivar` en cada una.

Secciones equivalentes para **"Fondos"** (nombre + meta) y **"Orígenes de ingreso"** (solo nombre).

Textos de la interfaz, literales:

| Elemento | Texto |
|---|---|
| Botón añadir | `Añadir categoría` |
| Encabezado archivadas | `Archivadas` |
| Acción reactivar | `Reactivar` |
| Estado vacío | `Aún no tienes categorías archivadas.` |
| Aviso de huérfanos | `Hay N movimientos sin categoría. Toca para reasignarlos.` |

---

## 7. Orden de trabajo

Una tarea por vez. Probar en el celular y hacer commit entre cada una.

1. Migración de esquema y lectura desde `rt_cats` / `rt_fondos` / `rt_origenes`, manteniendo la
   interfaz actual intacta. Al terminar, la app debe verse **idéntica** y no perder ningún dato.
2. Sección de gestión de categorías en Ajustes: crear, renombrar, cambiar color y presupuesto.
3. Archivar, reactivar, eliminar y reasignar.
4. Fondos y orígenes con las mismas reglas.
5. Exportación e importación versionada, incluida la lectura de respaldos v1.
6. Manejo de huérfanos y aviso en Ajustes.

**En cada cambio de `index.html`, subir la versión del caché en `sw.js`**
(`ritmo-v1` → `ritmo-v2` → …). Si se olvida, el iPhone seguirá sirviendo la versión vieja
desde el caché y parecerá que el cambio no funcionó.

---

## 8. Pruebas antes de cada commit

- [ ] Registrar un gasto, cerrar la app por completo, reabrir: el gasto sigue ahí.
- [ ] Crear una categoría, registrar un gasto en ella, renombrarla: el gasto conserva el monto
      y aparece con el nombre nuevo.
- [ ] Archivar una categoría con movimientos: desaparece de los chips, sigue en el historial.
- [ ] Exportar, borrar todo, importar el archivo: el estado vuelve completo.
- [ ] Importar un respaldo generado con la versión anterior de la app: se migra sin errores.
- [ ] Modo avión: la app abre desde el ícono y funciona.
