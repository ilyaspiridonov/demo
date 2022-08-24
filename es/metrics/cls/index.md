---
layout: post UPDATED
title: Cambio de diseño acumulativo (CLS)
authors:
  - philipwalton
  - mihajlija
date: '2019-06-11'
updated: '2021-06-01'
description: |2

  Esta publicación presenta la métrica Cumulative Layout Shift (CLS) y explica

  como medirlo
tags:
  - performance
  - metrics
  - web-vitals
---

{% Banner 'caution', 'body' %} **1 de junio de 2021:** la implementación de CLS ha cambiado. Para obtener más información sobre los motivos del cambio, consulte [Evolución de la métrica CLS](/evolving-cls) . {% endBanner %}

{% Aside 'key-term' %} El cambio de diseño acumulativo (CLS) es una métrica importante centrada en el usuario para medir [la estabilidad visual](/user-centric-performance-metrics/#types-of-metrics) porque ayuda a cuantificar la frecuencia con la que los usuarios experimentan cambios de diseño inesperados: un CLS bajo ayuda a garantizar que la página sea [agradable](/user-centric-performance-metrics/#questions) . {% endAside %}

¿Alguna vez has estado leyendo un artículo en línea cuando algo cambia repentinamente en la página? Sin previo aviso, el texto se mueve y has perdido tu lugar. O peor aún: está a punto de tocar un enlace o un botón, pero en el instante antes de que su dedo aterrice, BOOM, el enlace se mueve y termina haciendo clic en otra cosa.

La mayoría de las veces, este tipo de experiencias son simplemente molestas, pero en algunos casos pueden causar un daño real.

<figure class="w-figure">
  <video autoplay controls loop muted class="w-screenshot" poster="https://storage.googleapis.com/web-dev-assets/layout-instability-api/layout-instability-poster.png" width="658" height="510">
    <source src="https://storage.googleapis.com/web-dev-assets/layout-instability-api/layout-instability2.webm" type="video/webm; codecs=vp8">
    <source src="https://storage.googleapis.com/web-dev-assets/layout-instability-api/layout-instability2.mp4" type="video/mp4; codecs=h264">
  </source></source></video>
  <figcaption class="w-figcaption w-figcaption--fullbleed">Un screencast que ilustra cómo la inestabilidad del diseño puede afectar negativamente a los usuarios.</figcaption></figure>

El movimiento inesperado del contenido de la página generalmente ocurre porque los recursos se cargan de forma asíncrona o los elementos DOM se agregan dinámicamente a la página sobre el contenido existente. El culpable podría ser una imagen o video con dimensiones desconocidas, una fuente que se muestra más grande o más pequeña que su respaldo, o un anuncio o widget de terceros que cambia de tamaño de forma dinámica.

Lo que hace que este problema sea aún más problemático es que la forma en que funciona un sitio en desarrollo a menudo es bastante diferente de cómo lo experimentan los usuarios. El contenido personalizado o de terceros a menudo no se comporta igual en el desarrollo que en la producción, las imágenes de prueba a menudo ya están en la memoria caché del navegador del desarrollador y las llamadas a la API que se ejecutan localmente suelen ser tan rápidas que la demora no se nota.

La métrica de cambio de diseño acumulativo (CLS) lo ayuda a abordar este problema al medir la frecuencia con la que ocurre para los usuarios reales.

## ¿Qué es CLS?

CLS es una medida de la mayor ráfaga de *puntajes de cambio de diseño* para cada cambio de diseño [inesperado](/cls/#expected-vs.-unexpected-layout-shifts) que ocurre durante toda la vida útil de una página.

Un *cambio de diseño* ocurre cada vez que un elemento visible cambia su posición de un cuadro renderizado al siguiente. (Consulte a continuación para obtener detalles sobre cómo se calculan las [puntuaciones de cambio de diseño](#layout-shift-score) individuales).

Una ráfaga de cambios de diseño, conocida como [*ventana de sesión*](/evolving-cls/#why-a-session-window) , es cuando uno o más cambios de diseño individuales ocurren en rápida sucesión con menos de 1 segundo entre cada cambio y un máximo de 5 segundos para la duración total de la ventana.

La ráfaga más grande es la ventana de la sesión con la máxima puntuación acumulada de todos los cambios de diseño dentro de esa ventana.

<figure class="w-figure">
  <video controls autoplay loop muted class="w-screenshot" width="658" height="452">
    <source src="https://storage.googleapis.com/web-dev-assets/better-layout-shift-metric/session-window.webm" type="video/webm">
    <source src="https://storage.googleapis.com/web-dev-assets/better-layout-shift-metric/session-window.mp4" type="video/mp4">
  </source></source></video>
  <figcaption class="w-figcaption">Ejemplo de ventanas de sesión. Las barras azules representan las puntuaciones de cada cambio de diseño individual.</figcaption></figure>

TEST
### ¿Qué es una buena puntuación CLS?

Para proporcionar una buena experiencia de usuario, los sitios deben esforzarse por tener una puntuación CLS de **0,1** o menos. Para asegurarse de alcanzar este objetivo para la mayoría de sus usuarios, un buen umbral para medir es el **percentil 75** de cargas de página, segmentado en dispositivos móviles y de escritorio.

<picture>
  <source srcset="{{ " image imgix media="(min-width: 640px)" width="400" height="100">{% Img src="image/tcFciHGuF3MxnTr1y5ue01OGLBn2/uqclEgIlTHhwIgNTXN3Y.svg", alt="Good CLS values are under 0.1, poor values are greater than 0.25 and anything in between needs improvement", width="400", height="300", class="w-screenshot w-screenshot--filled width-full" %}</source></picture>

{% Aside %} Para obtener más información sobre la investigación y la metodología detrás de esta recomendación, consulte: [Definición de los umbrales de métricas de Core Web Vitals](/defining-core-web-vitals-thresholds/) {% endAside %}

## Cambios de diseño en detalle

Los cambios de diseño están definidos por la [API de inestabilidad de diseño](https://github.com/WICG/layout-instability) , que informa las entradas `layout-shift` cada vez que un elemento visible dentro de la ventana gráfica cambia su posición inicial (por ejemplo, su posición superior e izquierda en el modo de [escritura](https://developer.mozilla.org/en-US/docs/Web/CSS/writing-mode) predeterminado) entre dos marcos. Dichos elementos se consideran *elementos inestables* .

Tenga en cuenta que los cambios de diseño solo ocurren cuando los elementos existentes cambian su posición inicial. Si se agrega un nuevo elemento al DOM o un elemento existente cambia de tamaño, no cuenta como un cambio de diseño, siempre que el cambio no provoque que otros elementos visibles cambien su posición inicial.

### Puntuación de cambio de diseño

Para calcular la *puntuación de cambio de diseño* , el navegador observa el tamaño de la ventana gráfica y el movimiento de *elementos inestables* en la ventana gráfica entre dos marcos renderizados. La puntuación de cambio de diseño es un producto de dos medidas de ese movimiento: la fracción de *impacto* y la *fracción de distancia* (ambas definidas a continuación).

```text
layout shift score = impact fraction * distance fraction
```

### fracción de impacto

La [fracción de impacto](https://github.com/WICG/layout-instability#Impact-Fraction) mide cómo *los elementos inestables* impactan el área de la ventana gráfica entre dos fotogramas.

La unión de las áreas visibles de todos *los elementos inestables* del cuadro anterior *y* el cuadro actual, como una fracción del área total de la ventana gráfica, es la *fracción de impacto* del cuadro actual.

{% Img src="image/tcFciHGuF3MxnTr1y5ue01OGLBn2/BbpE9rFQbF8aU6iXN1U6.png", alt="Impact fraction example with one *unstable element*", width="800", height="600", linkTo=true %}

En la imagen de arriba hay un elemento que ocupa la mitad de la ventana gráfica en un cuadro. Luego, en el siguiente cuadro, el elemento se desplaza hacia abajo un 25 % de la altura de la ventana gráfica. El rectángulo punteado rojo indica la unión del área visible del elemento en ambos fotogramas, que, en este caso, es el 75 % del área de visualización total, por lo que su *fracción de impacto* es `0.75` .

### Fracción de distancia

La otra parte de la ecuación de puntuación de cambio de diseño mide la distancia que se han movido los elementos inestables, en relación con la ventana gráfica. La *fracción de distancia* es la mayor distancia que se ha movido cualquier *elemento inestable* en el marco (ya sea horizontal o verticalmente) dividida por la dimensión más grande de la ventana gráfica (ancho o alto, lo que sea mayor).

{% Img src="image/tcFciHGuF3MxnTr1y5ue01OGLBn2/ASnfpVs2n9winu6mmzdk.png", alt="Distance fraction example with one *unstable element*", width="800", height="600", linkTo=true %}

En el ejemplo anterior, la dimensión más grande de la ventana gráfica es la altura, y el elemento inestable se ha movido un 25 % de la altura de la ventana gráfica, lo que hace que la *fracción de distancia* sea 0,25.

Entonces, en este ejemplo, la *fracción de impacto* es `0.75` y la *fracción de distancia* es `0.25` , por lo que la *puntuación de cambio de diseño* es `0.75 * 0.25 = 0.1875` .

{% Aside %} Inicialmente, la puntuación de cambio de diseño se calculó solo en función de *la fracción de impacto* . La *fracción de distancia* se introdujo para evitar penalizar demasiado los casos en los que los elementos grandes se desplazan en una pequeña cantidad. {% endAside %}

El siguiente ejemplo ilustra cómo la adición de contenido a un elemento existente afecta la puntuación de cambio de diseño:

{% Img src="image/tcFciHGuF3MxnTr1y5ue01OGLBn2/xhN81DazXCs8ZawoCj0T.png", alt="Layout shift example with stable and *unstable elements* and viewport clipping", width="800", height="600", linkTo=true %}

El "¡Haz clic en mí!" El botón se adjunta a la parte inferior del cuadro gris con texto negro, que empuja el cuadro verde con texto blanco hacia abajo (y parcialmente fuera de la ventana gráfica).

En este ejemplo, el cuadro gris cambia de tamaño, pero su posición inicial no cambia, por lo que no es un *elemento inestable* .

El "¡Haz clic en mí!" El botón no estaba previamente en el DOM, por lo que su posición de inicio tampoco cambia.

Sin embargo, la posición de inicio del cuadro verde cambia, pero dado que se movió parcialmente fuera de la ventana gráfica, el área invisible no se considera al calcular la *fracción de impacto* . La unión de las áreas visibles del cuadro verde en ambos marcos (ilustrada por el rectángulo punteado rojo) es la misma que el área del cuadro verde en el primer cuadro: el 50 % de la ventana gráfica. La *fracción de impacto* es `0.5` .

La *fracción de distancia* se ilustra con la flecha morada. El cuadro verde se ha movido hacia abajo aproximadamente un 14 % de la ventana gráfica, por lo que la *fracción de distancia* es `0.14` .

La puntuación de cambio de diseño es `0.5 x 0.14 = 0.07` .

Este último ejemplo ilustra múltiples *elementos inestables* :

{% Img src="image/tcFciHGuF3MxnTr1y5ue01OGLBn2/FdCETo2dLwGmzw0V5lNT.png", alt="Layout shift example with multiple stable and *unstable elements*", width="800", height="600", linkTo=true %}

En el primer cuadro de arriba hay cuatro resultados de una solicitud de API para animales, ordenados alfabéticamente. En el segundo cuadro, se agregan más resultados a la lista ordenada.

El primer elemento de la lista ("Gato") no cambia su posición de inicio entre fotogramas, por lo que es estable. Del mismo modo, los nuevos elementos agregados a la lista no estaban previamente en el DOM, por lo que sus posiciones de inicio tampoco cambian. Pero los elementos etiquetados como "Perro", "Caballo" y "Cebra" cambian sus posiciones de inicio, lo que los convierte en *elementos inestables* .

Una vez más, los rectángulos punteados rojos representan la unión de estos tres *elementos inestables* 'antes y después de las áreas, que en este caso es alrededor del 38% del área de la ventana gráfica ( *fracción de impacto* de `0.38` ).

Las flechas representan las distancias que *los elementos inestables se* han movido desde sus posiciones iniciales. El elemento "Cebra", representado por la flecha azul, es el que más se ha movido, aproximadamente un 30 % de la altura de la ventana gráfica. Eso hace que la *fracción de distancia* en este ejemplo sea `0.3` .

La puntuación de cambio de diseño es `0.38 x 0.3 = 0.1172` .

### Cambios de diseño esperados e inesperados

No todos los cambios de diseño son malos. De hecho, muchas aplicaciones web dinámicas cambian con frecuencia la posición inicial de los elementos de la página.

#### Cambios de diseño iniciados por el usuario

Un cambio de diseño solo es malo si el usuario no lo espera. Por otro lado, los cambios de diseño que ocurren en respuesta a las interacciones del usuario (hacer clic en un enlace, presionar un botón, escribir en un cuadro de búsqueda y similares) generalmente están bien, siempre que el cambio ocurra lo suficientemente cerca de la interacción para que la relación sea claro para el usuario.

Por ejemplo, si la interacción de un usuario desencadena una solicitud de red que puede tardar un tiempo en completarse, es mejor crear un espacio de inmediato y mostrar un indicador de carga para evitar un cambio de diseño desagradable cuando se complete la solicitud. Si el usuario no se da cuenta de que algo se está cargando, o no sabe cuándo estará listo el recurso, puede intentar hacer clic en otra cosa mientras espera, algo que podría salirse de debajo de él.

Los cambios de diseño que ocurren dentro de los 500 milisegundos de la entrada del usuario tendrán el indicador [`hadRecentInput`](https://wicg.github.io/layout-instability/#dom-layoutshift-hadrecentinput) establecido, por lo que pueden excluirse de los cálculos.

{% Aside 'caution' %} El indicador `hadRecentInput` solo será verdadero para eventos de entrada discretos como tocar, hacer clic o presionar una tecla. Las interacciones continuas, como desplazamientos, arrastres o gestos de pellizcar y hacer zoom, no se consideran "entradas recientes". Consulte la [Especificación de inestabilidad de diseño](https://github.com/WICG/layout-instability#recent-input-exclusion) para obtener más detalles. {% endAside %}

#### Animaciones y transiciones

Las animaciones y las transiciones, cuando se hacen bien, son una excelente manera de actualizar el contenido de la página sin sorprender al usuario. El contenido que cambia abrupta e inesperadamente en la página casi siempre crea una mala experiencia para el usuario. Pero el contenido que se mueve de forma gradual y natural de una posición a la siguiente a menudo puede ayudar al usuario a comprender mejor lo que está sucediendo y guiarlo entre los cambios de estado.

Asegúrese de respetar la configuración preferida del navegador [`prefers-reduced-motion`](/prefers-reduced-motion/) , ya que algunos visitantes del sitio pueden experimentar efectos nocivos o problemas de atención debido a la animación.

La propiedad de [`transform`](https://developer.mozilla.org/en-US/docs/Web/CSS/transform) CSS le permite animar elementos sin activar cambios de diseño:

- En lugar de cambiar las propiedades de `height` y `width` , usa `transform: scale()` .
- Para mover elementos, evita cambiar las propiedades `top` , `right` , `bottom` o `left` y usa `transform: translate()` en su lugar.

## Cómo medir CLS

CLS se puede medir [en el laboratorio](/user-centric-performance-metrics/#in-the-lab) o [en el campo](/user-centric-performance-metrics/#in-the-field) , y está disponible en las siguientes herramientas:

{% Aside 'caution' %} Las herramientas de laboratorio normalmente cargan páginas en un entorno sintético y, por lo tanto, solo pueden medir los cambios de diseño que ocurren durante la carga de la página. Como resultado, los valores de CLS informados por las herramientas de laboratorio para una página determinada pueden ser menores que los que experimentan los usuarios reales en el campo. {% endAside %}

### herramientas de campo

- [Informe de experiencia de usuario de Chrome](https://developers.google.com/web/tools/chrome-user-experience-report)
- [Perspectivas de PageSpeed](https://developers.google.com/speed/pagespeed/insights/)
- [Consola de búsqueda (informe Core Web Vitals)](https://support.google.com/webmasters/answer/9205520)
- [Biblioteca de JavaScript `web-vitals`](https://github.com/GoogleChrome/web-vitals)

### herramientas de laboratorio

- [Herramientas para desarrolladores de Chrome](https://developers.google.com/web/tools/chrome-devtools/)
- [Faro](https://developers.google.com/web/tools/lighthouse/)
- [Prueba de página web](https://webpagetest.org/)

### Medir CLS en JavaScript

Para medir CLS en JavaScript, puede usar la [API de inestabilidad de diseño](https://github.com/WICG/layout-instability) . El siguiente ejemplo muestra cómo crear un [`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) que detecta entradas `layout-shift` inesperadas, las agrupa en sesiones y registra el valor máximo de la sesión cada vez que cambia.

```js
let clsValue = 0;
let clsEntries = [];

let sessionValue = 0;
let sessionEntries = [];

new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    // Only count layout shifts without recent user input.
    if (!entry.hadRecentInput) {
      const firstSessionEntry = sessionEntries[0];
      const lastSessionEntry = sessionEntries[sessionEntries.length - 1];

      // If the entry occurred less than 1 second after the previous entry and
      // less than 5 seconds after the first entry in the session, include the
      // entry in the current session. Otherwise, start a new session.
      if (sessionValue &&
          entry.startTime - lastSessionEntry.startTime < 1000 &&
          entry.startTime - firstSessionEntry.startTime < 5000) {
        sessionValue += entry.value;
        sessionEntries.push(entry);
      } else {
        sessionValue = entry.value;
        sessionEntries = [entry];
      }

      // If the current session value is larger than the current CLS value,
      // update CLS and the entries contributing to it.
      if (sessionValue > clsValue) {
        clsValue = sessionValue;
        clsEntries = sessionEntries;

        // Log the updated value (and its entries) to the console.
        console.log('CLS:', clsValue, clsEntries)
      }
    }
  }
}).observe({type: 'layout-shift', buffered: true});
```

{% Aside 'warning' %}

Este código muestra la forma básica de calcular y registrar CLS. Sin embargo, medir CLS con precisión de una manera que coincida con lo que se mide en el[Informe de experiencia del usuario de Chrome](https://developers.google.com/web/tools/chrome-user-experience-report) (CrUX) es más complicado. Vea a continuación para más detalles:

{% endAside %}

En la mayoría de los casos, el valor CLS actual en el momento en que se descarga la página es el valor CLS final para esa página, pero existen algunas excepciones importantes:

La siguiente sección enumera las diferencias entre lo que informa la API y cómo se calcula la métrica.

#### Diferencias entre la métrica y la API

- Si una página se carga en segundo plano, o si está en segundo plano antes de que el navegador pinte cualquier contenido, entonces no debería informar ningún valor CLS.
- Si una página se restaura desde la [memoria caché de retroceso/avance](/bfcache/#impact-on-core-web-vitals) , su valor CLS debe restablecerse a cero, ya que los usuarios experimentan esto como una visita a la página distinta.
- La API no informa las entradas de `layout-shift` para los cambios que ocurren dentro de los iframes, pero para medir CLS correctamente, debe tenerlos en cuenta. Los submarcos pueden usar la API para informar sus entradas `layout-shift` al marco principal para su [agregación](https://github.com/WICG/layout-instability#cumulative-scores) .

Además de estas excepciones, CLS tiene cierta complejidad añadida debido al hecho de que mide la vida útil completa de una página:

- Los usuarios pueden mantener una pestaña abierta durante *mucho* tiempo: días, semanas, meses. De hecho, es posible que un usuario nunca cierre una pestaña.
- En los sistemas operativos móviles, los navegadores normalmente no ejecutan devoluciones de llamada de descarga de página para pestañas en segundo plano, lo que dificulta informar el valor "final".

Para manejar tales casos, se debe informar CLS cada vez que una página está en segundo plano, además de cada vez que se descarga (el [evento de cambio de `visibilitychange`](https://developers.google.com/web/updates/2018/07/page-lifecycle-api#event-visibilitychange) cubre ambos escenarios). Y los sistemas de análisis que reciben estos datos deberán calcular el valor CLS final en el backend.

En lugar de memorizar y lidiar con todos estos casos usted mismo, los desarrolladores pueden usar la [biblioteca JavaScript `web-vitals`](https://github.com/GoogleChrome/web-vitals) para medir CLS, que representa todo lo mencionado anteriormente:

```js
import {getCLS} from 'web-vitals';

// Measure and log CLS in all situations
// where it needs to be reported.
getCLS(console.log);
```

Puede [consultar el código fuente de `getCLS)`](https://github.com/GoogleChrome/web-vitals/blob/master/src/getCLS.ts) para ver un ejemplo completo de cómo medir CLS en JavaScript.

{% Aside %} En algunos casos (como iframes de origen cruzado) no es posible medir CLS en JavaScript. Consulte la sección de [limitaciones](https://github.com/GoogleChrome/web-vitals#limitations) de la biblioteca `web-vitals` para obtener más información. {% endAside %}

## Cómo mejorar CLS

{% Banner 'info', 'body' %} **Nuevo:** consulte los [patrones de Web Vitals](/patterns/web-vitals-patterns) para ver las implementaciones de patrones de UX comunes optimizados para Core Web Vitals. Esta colección incluye patrones que a menudo son difíciles de implementar sin cambios de diseño. {% endBanner %}

Para la mayoría de los sitios web, puede evitar todos los cambios de diseño inesperados siguiendo algunos principios rectores:

- **Incluya siempre atributos de tamaño en sus imágenes y elementos de video, o reserve el espacio requerido con [cuadros de relación de aspecto CSS](https://css-tricks.com/aspect-ratio-boxes/) .** Este enfoque garantiza que el navegador pueda asignar la cantidad correcta de espacio en el documento mientras se carga la imagen. Tenga en cuenta que también puede usar la [política de características de medios sin tamaño](https://github.com/w3c/webappsec-feature-policy/blob/master/policies/unsized-media.md) para forzar este comportamiento en navegadores que admitan políticas de características.
- **Nunca inserte contenido sobre contenido existente, excepto en respuesta a una interacción del usuario.** Esto garantiza que se esperan los cambios de diseño que se produzcan.
- **Prefiere las animaciones de transformación a las animaciones de propiedades que desencadenan cambios de diseño.** Anime las transiciones de una manera que proporcione contexto y continuidad de un estado a otro.

Para obtener información detallada sobre cómo mejorar CLS, consulte [Optimizar CLS](/optimize-cls/) y [Depurar cambios de diseño](/debug-layout-shifts) .

## Recursos adicionales

- Guía de Google Publisher Tag para [minimizar el cambio de diseño](https://developers.google.com/doubleclick-gpt/guides/minimize-layout-shift)
- [Comprender el cambio de diseño acumulativo](https://youtu.be/zIJuY-JCjqw) por [Annie Sullivan](https://anniesullie.com/) y [Steve Kobes](https://kobes.ca/) en [#PerfMatters](https://perfmattersconf.com/) (2020)

{% include 'content/metrics/metrics-changelog.njk' %}
