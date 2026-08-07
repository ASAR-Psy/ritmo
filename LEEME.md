# Ritmo — mis gastos

Aplicación web instalable para registrar gastos diarios, ingresos y aportes a fondos personales.
Funciona sin conexión. Los datos se guardan **solo en tu celular**: no hay servidor, ni cuenta, ni suscripción.

---

## Archivos

| Archivo | Para qué sirve |
|---|---|
| `index.html` | La aplicación completa (interfaz + lógica) |
| `manifest.json` | Permite instalarla en la pantalla de inicio |
| `sw.js` | Service worker: hace que funcione sin señal |
| `icon-192.png`, `icon-512.png` | Íconos de la app |

Los cinco archivos deben quedar en la **misma carpeta**.

---

## Publicar en GitHub Pages

1. Crea un repositorio nuevo (por ejemplo `ritmo`) y márcalo como público.
2. Sube los cinco archivos a la raíz del repositorio.
3. Entra a **Settings → Pages**.
4. En *Source* elige `Deploy from a branch`, rama `main`, carpeta `/ (root)`. Guarda.
5. Espera uno o dos minutos. La URL será `https://TU-USUARIO.github.io/ritmo/`.

HTTPS es obligatorio para que el service worker funcione. GitHub Pages ya lo da.

---

## Instalar en el celular

**Android (Chrome):** abre la URL → menú de tres puntos → *Instalar aplicación* / *Añadir a pantalla principal*.

**iPhone (Safari):** abre la URL → botón compartir → *Añadir a pantalla de inicio*.
Debe ser Safari; en iOS los otros navegadores no instalan aplicaciones web.

Una vez instalada, abre desde el ícono. Ya no necesita conexión.

---

## Cómo usarla

**Registrar.** Escribe el monto, toca una categoría, agrega una nota corta si quieres, y guarda.
Tres campos, sin pantallas intermedias.

- **Gasto**: salidas de dinero, por categoría.
- **Ingreso**: entradas (pago del contrato, trabajo extra).
- **Aporte a fondo**: dinero que apartas hacia Independencia, Emprendimiento, Reserva o Contingencia.

**La barra de ritmo (arriba).** No solo muestra cuánto llevas gastado: la línea blanca marca dónde
*deberías* ir según el día del mes. Si el color llena más allá de la marca, vas adelantado en el gasto.
Verde = por debajo del ritmo. Ámbar = adelantado. Rojo = presupuesto del mes superado.

**Mes.** Desglose por categoría con aviso al 80% y al pasarse, gráfico de gasto diario y el balance
del mes (ingresos − gastos − aportes = sin asignar).

**Fondos.** Acumulado histórico de cada fondo y avance hacia su meta.

**Ajustes.** Presupuesto de cada categoría, meta de cada fondo, y copia de seguridad.

---

## Copia de seguridad (importante)

Los datos viven en el navegador de tu celular. Si borras la app, limpias los datos del navegador
o cambias de equipo, **se pierden**.

Entra a **Ajustes → Descargar mis datos** una vez al mes y guarda el archivo `.json` en tu correo
o en la nube. Para recuperarlo: **Ajustes → Restaurar desde archivo**.

---

## Notas técnicas

- Sin dependencias externas: no carga librerías ni fuentes de internet.
- Almacenamiento en `localStorage`, con respaldo en memoria. Si abres el `index.html` en un
  entorno donde `localStorage` está bloqueado, la app avisa que está en vista previa y no guarda.
- Para cambiar las categorías o los fondos, edita los arreglos `CATEGORIAS` y `FONDOS`
  al comienzo del `<script>` en `index.html`.
- Al modificar `index.html`, sube también `sw.js` con el número de versión cambiado
  (`ritmo-v1` → `ritmo-v2`) para que el celular descargue la versión nueva en lugar de la del caché.
