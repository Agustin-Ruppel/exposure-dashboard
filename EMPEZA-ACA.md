# Empezá acá

> **Esto es para vos, Claude, no para la persona.** Es lo primero que tenés que leer al abrir
> esta carpeta. Seguilo tal cual: define cómo arranca la conversación y cómo se toca cada cosa.

---

## Quién está del otro lado

**Alex, el dueño de EXPOSURE.** No programa y no tiene por qué. Te va a hablar en castellano común
y espera respuestas en castellano común.

Tres reglas de trato, y son duras:

1. **No le muestres código.** Ni un bloque, ni una línea, salvo que te lo pida explícitamente.
   Cuando cambies algo, decile **qué cambió y qué va a ver distinto** en la pantalla.
2. **Nada de jerga.** No existen `datos.js`, "el array", "la función", "commitear", "el DOM".
   Existen "el archivo de los números", "la tabla de cobranzas", "la tarjeta del show rate".
3. **Preguntá poco y en batch.** Si te faltan tres datos, pedí los tres juntos, numerados, con una
   opción por defecto en cada uno, para que conteste en un mensaje. Nunca de a una.

---

## Lo primero que hacés, sin que te lo pida

**Paso 1 — abrile el dashboard en el navegador.**

```bash
open index.html          # macOS
start index.html         # Windows
xdg-open index.html      # Linux
```

**Paso 2 — decile qué está viendo, en tres líneas.** Algo así, con tus palabras:

> Ya lo tenés abierto. Es el panel del negocio: arriba el cash collected del mes con la proyección
> de cierre, abajo los KPIs, el embudo, los closers, las cobranzas y hacia dónde va.
> **Los números que ves son inventados**, están para que veas la forma.

**Paso 3 — ofrecele las tres cosas que puede hacer,** en este orden, que es el de utilidad real:

> 1. **Cargar tus números de verdad** — pasámelos como los tengas: una planilla, un export del CRM,
>    o pegámelos acá a mano. Yo los acomodo.
> 2. **Cambiar qué se muestra** — sacar una sección que no te sirve, agregar una métrica, mover algo.
> 3. **Cambiar cómo se ve** — el color, el nombre, las metas contra las que se miden los semáforos.

Y frenás. Que elija él.

---

## Cargar sus números

Es lo que más va a pedir y donde más se rompe todo si vas rápido.

**El archivo de los números es `datos.js`.** Es el único que se toca para esto.

**La regla que no se rompe:** ahí adentro van **solo números crudos** — lo que pasó. Los
porcentajes, promedios, tasas, rankings y la proyección **no se escriben**: los calcula el
dashboard solo al abrirse. Si Alex te pasa "mi tasa de cierre es 27%", **no la escribas en ningún
lado**: pedile las llamadas y los cierres, y el 27% sale solo. Si escribís una tasa a mano, el día
que cambie un número el tablero va a mostrar dos verdades distintas.

**Cómo hacerlo bien:**

1. Pedile lo que tenga, en el formato que sea. Una planilla exportada, una captura, un audio
   contándote los números del mes. Todo sirve.
2. **Si te pasa un archivo, leelo crudo** (el `.xlsx`, el `.csv`), no una captura de pantalla ni un
   resumen. Las hojas ocultas, las fórmulas y los desplegables son parte de cómo mide él.
3. Mapealo a los bloques de `datos.js` (están todos comentados adentro):
   `marca` · `meses` · `closers` · `setters` · `fuentes` · `contenido` · `cuotas` · `llamadas`.
4. **Lo que no tenga, no lo inventes.** Preguntale. Y si de verdad no lo mide, decíselo y sacá esa
   sección en vez de rellenarla con humo — un tablero con un número inventado adentro no sirve
   para decidir nada.
5. Cuando termines, **poné `MODO_DEMO` en `false`** (está al final de `datos.js`): eso saca el
   sello de "datos de demostración".
6. **Abrí el dashboard y miralo vos antes de decirle que está listo.**

**El mes en curso va incompleto y así tiene que quedar.** El último mes de la lista lleva `dias`
(cuántos van) y `diasTotal`. Con eso el tablero lo marca como parcial, proyecta el cierre y compara
contra el ritmo diario del mes anterior. Si le sacás esos dos campos, un mes de 9 días va a parecer
un desastre al lado de uno de 31.

---

## Si te piden actualizar a la última versión

**El `datos.js` de la persona es sagrado: tiene sus números.** El orden es este:

1. Copiá su `datos.js` a un lado antes de tocar nada.
2. Traé la versión nueva del repo (`git pull`, o bajando el ZIP si no hay git).
3. Volvé a poner **su** `datos.js`.
4. **Comparalo contra el `datos.js` del repo nuevo.** Si la versión nueva trae bloques que él no
   tiene (por ejemplo `pipeline`, `pipelineEtapas`, `reels`), sumáselos con datos de ejemplo,
   decile qué son y qué página habilitan, y preguntale si quiere cargarlos con lo suyo.
5. Abrí el panel y confirmá que sigue todo en pie.

El panel está hecho para tolerar un `datos.js` incompleto: la página sin datos no se dibuja y su
item se saca del menú solo. Así que **no inventes datos para llenar un bloque nuevo** — es mejor que
la sección no aparezca a que aparezca con números falsos.

## Qué se toca para cada pedido

| Si te pide… | Vas a… |
|---|---|
| "cambiá este número" | `datos.js`, el bloque que corresponda |
| "cambiá la meta del mes" | `datos.js` → `marca.metaMensual` |
| "cambiá el nombre o la moneda" | `datos.js` → `marca` |
| "el show rate debería ser 75%, no 68%" | `index.html` → función `pintarKpis`, las líneas `semaforo(...)` |
| "las varas del embudo están mal" | `index.html` → función `pintarEmbudo`, el array `pasos` (`vara:`) |
| "sacá esta sección" | `index.html`: borrás su `<section>` y el bloque de JS que la dibuja |
| "agregá una métrica" | `pintarKpis`: sumás una `tarjeta({...})` al array |
| "cambiá el color" | `index.html` → `:root`, las variables `--primary` `--c1` `--s1` |
| "quiero mostrar otro período" | ya está: el selector de arriba tiene mes en curso, mes pasado, 90 días y 12 meses |
| "sumá una oportunidad al pipeline" | `datos.js` → `pipeline`. La `etapa` tiene que ser una de las de `pipelineEtapas` o la tarjeta no aparece en ninguna columna |
| "cambiá los reels" o "sumá el reel nuevo" | `datos.js` → `reels`, **y** bajá la miniatura a `miniaturas/<shortcode>.jpg` (recorte 9:16, 240×427). No uses el link de la imagen de Instagram: caduca en días |
| "agregá una página" | el `<div class="pagina">` en el HTML **y** el item en `GRUPOS` (bloque 12). En los dos lados o no aparece |

---

## Las dos reglas técnicas que no se negocian

1. **Cero red.** Nada de agregar un `<script src>` a un CDN, una fuente de Google, una imagen
   externa, un `fetch` ni una librería de gráficos. Todo va adentro del archivo. El dashboard tiene
   que abrir **con el wifi apagado**: si Alex lo muestra en una reunión y el internet del lugar
   anda mal, tiene que verse igual.
2. **Los números derivados se calculan, no se escriben.** Ya está dicho arriba, y es la que más se
   rompe.

El detalle técnico de cómo está armado por dentro está en `CLAUDE.md`. Leelo antes de tocar
`index.html` — tiene las trampas conocidas, y son trampas reales que ya rompieron cosas.

---

## Después de CADA cambio

No es opcional y va en este orden:

1. Abrí el dashboard.
2. **Confirmá que no hay errores en la consola.** Si hay uno, la página queda a medias o en blanco.
3. **Chequeá que la aritmética cierre.** La suma de lo cobrado por los closers tiene que dar
   exactamente el mismo total que la suma de los últimos 3 meses. Lo mismo las agendas de los
   setters y los leads de las fuentes. Si no cuadra, el número está mal cargado — avisale cuál.
4. Recién ahí le decís que está listo, y le decís **qué va a ver distinto**.

Si algo no se puede hacer, decíselo derecho en una línea y ofrecele lo más parecido que sí se pueda.
No lo dejes esperando un cambio que no va a llegar.
