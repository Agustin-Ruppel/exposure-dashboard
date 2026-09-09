# EXPOSURE · Panel del negocio — mapa para Claude

> 🔴 **Si es la primera vez que abrís esta carpeta, leé [`EMPEZA-ACA.md`](EMPEZA-ACA.md) antes que
> este archivo.** Ahí está cómo arranca la conversación, quién está del otro lado y cómo hablarle.
> Este documento es la referencia técnica: se lee cuando ya vas a tocar `index.html`.

Dashboard de negocio en **un solo HTML autosuficiente**. Se abre con `file://`, sin servidor.

## Las dos reglas que no se rompen

1. **Cero red.** Nada de `<script src>` a un CDN, `@import` de fuentes, `fetch`, imágenes externas
   ni librerías. Los íconos son SVG inline (objeto `IC`) y los gráficos son SVG escrito a mano.
   La única etiqueta externa permitida es `<script src="datos.js">`, que es un archivo local.
2. **En `datos.js` solo van números crudos.** Toda tasa, promedio, delta, ranking o proyección se
   calcula en `index.html`. Si te piden agregar un porcentaje a `datos.js`, la respuesta es
   calcularlo, no escribirlo.

## Estructura

```
index.html    todo: CSS en <style>, marcado, y el JS en un IIFE al final
datos.js      LO ÚNICO que edita el usuario. Define DATOS y MODO_DEMO
README.md     instrucciones para alguien que no programa
```

Dentro de `index.html`, el JS está numerado por bloques y en este orden:

| Bloque | Qué hace |
|---|---|
| formato | `miles` `money` `moneyK` `pct` `etiqMes` `mesLargo` `diasDe` `esParcial` |
| `IC` / `ico()` | los íconos Lucide inline |
| cálculo | `agregar` `derivar` `rango` `calc` `delta` `semaforo` — el motor |
| `CU` | totales de cobranzas (estado de hoy, no depende del período) |
| animación | `countUp` `activar` y el `IntersectionObserver` |
| tooltip | `mostrarTip` / `ocultarTip` |
| helpers de gráfico | `escala` `pathDe` `largoPath` `sparkline` `colorNombre` |
| 1-13 | encabezado, hero, KPIs, año, embudo, equipo, fuentes, contenido, cobranzas, proyección, llamadas, pie, **sidebar**, arranque |

**Son páginas, no una página larga.** Cada `<div class="pagina" id="p-xxx">` envuelve una o más
`<section>`, y el router por hash (bloque 12, función `ir()`) muestra una sola. Va por hash y **no**
por `history.pushState`, porque el panel se abre con `file://` y ahí `pushState` tira `SecurityError`.
Al cambiar de página se vuelve a disparar la animación de entrada de esa página.

**Los reels** (`datos.js` → `reels`) traen métricas reales de Instagram más la atribución de
negocio. **Las miniaturas son archivos locales en `miniaturas/<shortcode>.jpg`**: nunca referencies
la `thumbnail_url` de Instagram, caduca a los pocos días y las tarjetas quedan en blanco. Para
actualizarlas: bajar el JPG, recortarlo 9:16, redimensionar a 240×427 y guardarlo con el shortcode
de nombre.

**El resto de cada página se inyecta desde JS** (bloque 12b): `secc(pagina, titulo, titular, periodo,
cuerpo)` crea una `<section class="rise">` y la cuelga de `#p-<pagina>`. Se hace así para no duplicar
marcado y porque `ir()` ya recorre las secciones de la página que abre, así que entran con la misma
animación. Helpers disponibles ahí: `tabla(cols, filas)` y `mini(valor, label, color)`.
**Toda sección nueva va con un período explícito** (`marca-per`) — mezclar un bloque de 90 días con
uno del mes en curso sin decirlo es la forma más fácil de que dos números no cierren.

**La sidebar** se genera desde el array `GRUPOS` del bloque 12. Cada item apunta al `id` de una
`<section class="rise">` y lleva una cifra viva que sale del mapa `CIFRA`. **El orden de la sidebar
tiene que ser el orden real de las secciones en el HTML**: si no coincide, el usuario hace clic y el
scroll salta para atrás. Si agregás una sección, sumala en los dos lugares.

## Cómo se anima (importante al agregar cosas)

No hay librería. Todo funciona con atributos que lee `activar(raiz)`:

- `data-cu="1234" data-fmt="money|pct|num|x"` → count-up del número.
- `data-w="63.2%"` en un elemento con `width:0` → la barra crece.
- `data-off="0"` sobre un path con `stroke-dashoffset` igual a su largo → la línea se dibuja.
- clases `.bar-anim` / `.area-anim` / `.punto-anim` → entran al agregarles `.in`.

`activar()` se llama al final de cada render y desde el `IntersectionObserver` de las secciones.
**Si agregás un bloque nuevo, llamá a `activar(tuContenedor)` o no se va a animar nada.**

## Trampas conocidas

- **El mes en curso es parcial.** `esParcial(m)` es true cuando el mes trae `dias`. En toda serie de
  *totales* (facturado, cobrado, cierres) ese mes se **proyecta** antes de dibujarlo, o el gráfico
  muestra un derrumbe que no existe. Las *tasas* no se proyectan: ya son ratios.
- **Los deltas de totales se calculan por día** (`tipo:"diario"`), no en bruto. Las tasas se
  comparan en puntos porcentuales (`tipo:"puntos"`).
- **Un ícono sin caja explícita ocupa la pantalla entera.** Ya pasó. Todo `ico()` va dentro de un
  `.ico-box` o de un selector con `width`/`height`.
- **Cuidado al nombrar clases CSS.** Es una sola hoja sin scoping: `.barra` ya existe para el
  gráfico, `.chip` para las tablas. Buscá el nombre antes de usarlo.
- **`largoPath()` monta un SVG temporal en el DOM** para medir. No lo llames dentro de un loop
  caliente.

## Verificar antes de decir que está listo

```bash
node qa.mjs     # si existe; si no, abrir el archivo y revisar la consola
```

Como mínimo: abrir `index.html`, confirmar **cero errores de consola**, que la aritmética cierre
(la suma de los closers tiene que dar el mismo cobrado que la suma de los últimos 3 meses), que
`document.body.scrollWidth` no supere el ancho de la ventana, y probar a 390px de ancho.
