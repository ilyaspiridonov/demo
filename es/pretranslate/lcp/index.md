---
layout: post
title: Pintura con contenido más grande (LCP)
authors:
  - philipwalton
date: '2019-08-08'
updated: '2020-06-17'
description: |2

  Esta publicación presenta la métrica de pintura con contenido más grande (LCP) y explica

  como medirlo
tags:
  - performance
  - metrics
---

{% Aside %} La pintura de contenido más grande (LCP) es una métrica importante centrada en el usuario para medir [la velocidad de carga percibida](/user-centric-performance-metrics/#types-of-metrics) porque marca el punto en la línea de tiempo de carga de la página cuando es probable que el contenido principal de la página se haya cargado. Un LCP rápido ayuda a asegurar al usuario que el la página es [útil](/user-centric-performance-metrics/#questions) . {% endAside %}

Históricamente, ha sido un desafío para los desarrolladores web medir qué tan rápido se carga el contenido principal de una página web y es visible para los usuarios.

Las métricas más antiguas como [load](https://developer.mozilla.org/en-US/docs/Web/Events/load) o [DOMContentLoaded](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded) no son buenas porque no se corresponden necesariamente con lo que el usuario ve en su pantalla. Y las métricas de rendimiento más nuevas y centradas en el usuario, como [First Contentful Paint (FCP),](/fcp/) solo capturan el comienzo de la experiencia de carga. Si una página muestra una pantalla de bienvenida o muestra un indicador de carga, este momento no es muy relevante para el usuario.

En el pasado, recomendamos métricas de rendimiento como [First Meaningful Paint (FMP)](/first-meaningful-paint/) y [Speed Index (SI)](/speed-index/) (ambos disponibles en Lighthouse) para ayudar a capturar más de la experiencia de carga después de la pintura inicial, pero estas métricas son complejas y difíciles de explicar. y, a menudo, incorrecto, lo que significa que todavía no identifican cuándo se ha cargado el contenido principal de la página.

A veces, lo más simple es mejor. Según las discusiones en el [Grupo de trabajo de rendimiento web](https://www.w3.org/webperf/) del W3C y la investigación realizada en Google, hemos descubierto que una forma más precisa de medir cuándo se carga el contenido principal de una página es observar cuándo se renderizó el elemento más grande.

## ¿Qué es LCP?

La métrica de pintura con contenido más grande (LCP) informa el tiempo de procesamiento de la [imagen o el bloque de texto](#what-elements-are-considered) más grande visible dentro de la ventana gráfica, en relación con el momento en que la página [comenzó a cargarse](https://w3c.github.io/hr-time/#timeorigin-attribute) .

<picture>
  <source srcset="{{ " image imgix media="(min-width: 640px)" width="400" height="100">{% Img src="image/eqprBhZUGfb8WYnumQ9ljAxRrA72/8ZW8LQsagLih1ZZoOmMR.svg", alt="Good LCP values are 2.5 seconds, poor values are greater than 4.0 seconds and anything in between needs improvement", width="400", height="300", class="w-screenshot w-screenshot--filled width-full" %}</source></picture>

### ¿Qué es una buena puntuación LCP?

Para brindar una buena experiencia de usuario, los sitios deben esforzarse por tener la pintura con contenido más grande de **2.5 segundos** o menos. Para asegurarse de que está alcanzando este objetivo para la mayoría de sus usuarios, un buen umbral para medir es el **percentil 75** de cargas de página, segmentado en dispositivos móviles y de escritorio.

{% Aside %} Para obtener más información sobre la investigación y la metodología detrás de esta recomendación, consulte: [Definición de los umbrales de métricas de Core Web Vitals](/defining-core-web-vitals-thresholds/) {% endAside %}

### ¿Qué elementos se consideran?

Como se especifica actualmente en la [API de pintura con contenido más grande](https://wicg.github.io/largest-contentful-paint/) , los tipos de elementos considerados para la pintura con contenido más grande son:

- `<img>` elementos
- `<image>` elementos dentro de un elemento `<svg>`
- `<video>` elementos (se utiliza la imagen del póster)
- Un elemento con una imagen de fondo cargada a través de la función [`url()`](https://developer.mozilla.org/en-US/docs/Web/CSS/url()) (a diferencia de un [degradado CSS](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Images/Using_CSS_gradients) )
- [Elementos a nivel de bloque](https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements) que contienen nodos de texto u otros elementos secundarios de texto a nivel de línea.

Tenga en cuenta que restringir los elementos a este conjunto limitado fue intencional para mantener las cosas simples al principio. Es posible que se agreguen elementos adicionales (por ejemplo, `<svg>` , `<video>` ) en el futuro a medida que se realicen más investigaciones.

### ¿Cómo se determina el tamaño de un elemento?

El tamaño del elemento informado para la pintura con contenido más grande suele ser el tamaño que es visible para el usuario dentro de la ventana gráfica. Si el elemento se extiende fuera de la ventana gráfica, o si alguno de los elementos está recortado o tiene un [desbordamiento](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow) no visible, esas porciones no cuentan para el tamaño del elemento.

Para los elementos de imagen que se han redimensionado de su [tamaño intrínseco](https://developer.mozilla.org/en-US/docs/Glossary/Intrinsic_Size) , el tamaño que se informa es el tamaño visible o el tamaño intrínseco, el que sea menor. Por ejemplo, las imágenes que se reducen a un tamaño mucho más pequeño que su tamaño intrínseco solo informarán el tamaño en el que se muestran, mientras que las imágenes que se estiran o expanden a un tamaño mayor solo informarán sus tamaños intrínsecos.

Para los elementos de texto, solo se considera el tamaño de sus nodos de texto (el rectángulo más pequeño que abarca todos los nodos de texto).

Para todos los elementos, no se considera ningún margen, relleno o borde aplicado a través de CSS.

{% Aside %} Determinar qué nodos de texto pertenecen a qué elementos a veces puede ser complicado, especialmente para elementos cuyos elementos secundarios incluyen elementos en línea y nodos de texto sin formato, pero también elementos a nivel de bloque. El punto clave es que cada nodo de texto pertenece (y solo a) su elemento ancestro de nivel de bloque más cercano. En [términos de especificaciones](https://wicg.github.io/element-timing/#set-of-owned-text-nodes) : cada nodo de texto pertenece al elemento que genera su [bloque contenedor](https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block) . {% endAside %}

### ¿Cuándo se reporta la pintura con mayor contenido?

Las páginas web a menudo se cargan en etapas y, como resultado, es posible que el elemento más grande de la página cambie.

Para manejar este potencial de cambio, el navegador envía un [`PerformanceEntry`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry) del tipo `largest-contentful-paint` más grande que identifica el elemento de contenido más grande tan pronto como el navegador ha pintado el primer marco. Pero luego, después de renderizar los marcos subsiguientes, enviará otra [`PerformanceEntry`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry) cada vez que cambie el elemento de contenido más grande.

Por ejemplo, en una página con texto y una imagen principal, el navegador inicialmente puede simplemente representar el texto, momento en el que el navegador enviaría una entrada de `largest-contentful-paint` `element` probablemente haría referencia a `<p>` o `<h1>` . Más tarde, una vez que la imagen principal termine de cargarse, se enviará `largest-contentful-paint` `element` hará referencia a `<img>` .

Es importante tener en cuenta que un elemento solo puede considerarse el elemento de contenido más grande una vez que se ha renderizado y es visible para el usuario. Las imágenes que aún no se han cargado no se consideran "renderizadas". Tampoco los nodos de texto utilizan fuentes web durante el [período de bloqueo de fuentes](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display#The_font_display_timeline) . En tales casos, un elemento más pequeño se puede informar como el elemento de contenido más grande, pero tan pronto como el elemento más grande termine de renderizarse, se informará a través de otro objeto `PerformanceEntry`

Además de las imágenes y fuentes de carga tardía, una página puede agregar nuevos elementos al DOM a medida que haya contenido nuevo disponible. Si alguno de estos nuevos elementos es más grande que el elemento de contenido más grande anterior, también se informará `PerformanceEntry`

Si un elemento que actualmente es el elemento de contenido más grande se elimina de la ventana gráfica (o incluso se elimina del DOM), seguirá siendo el elemento de contenido más grande a menos que se represente un elemento más grande.

{% Aside %} Antes de Chrome 88, los elementos eliminados no se consideraban como elementos de contenido más grandes y, al eliminar el candidato actual, se enviaría una nueva entrada de `largest-contentful-paint` Sin embargo, debido a patrones de IU populares, como carruseles de imágenes que a menudo eliminaban elementos DOM, la métrica se actualizó para reflejar con mayor precisión lo que experimentan los usuarios. Consulte el [CHANGELOG](https://chromium.googlesource.com/chromium/src/+/master/docs/speed/metrics_changelog/2020_11_lcp.md) para obtener más detalles. {% endAside %}

El navegador dejará de informar nuevas entradas tan pronto como el usuario interactúe con la página (mediante un toque, desplazamiento o pulsación de tecla), ya que la interacción del usuario a menudo cambia lo que está visible para el usuario (lo cual es especialmente cierto con el desplazamiento).

Para fines de análisis, solo debe informar el `PerformanceEntry` enviado más recientemente a su servicio de análisis.

{% Aside 'caution' %} Dado que los usuarios pueden abrir páginas en una pestaña en segundo plano, es posible que la pintura con contenido más grande no ocurra hasta que el usuario enfoque la pestaña, lo que puede ser mucho más tarde que cuando la cargó por primera vez. {% endAside %}

#### Tiempo de carga vs.tiempo de renderizado

Por motivos de seguridad, la marca de tiempo de procesamiento de las imágenes no se expone para las imágenes de origen cruzado que carecen del encabezado [`Timing-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin) En cambio, solo se expone su tiempo de carga (ya que esto ya está expuesto a través de muchas otras API web).

El [siguiente ejemplo de uso](#measure-lcp-in-javascript) muestra cómo manejar elementos cuyo tiempo de renderizado no está disponible. Pero, cuando sea posible, siempre se recomienda configurar el [`Timing-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin) , para que sus métricas sean más precisas.

### ¿Cómo se manejan los cambios de tamaño y diseño de los elementos?

Para mantener baja la sobrecarga de rendimiento de calcular y enviar nuevas entradas de rendimiento, los cambios en el tamaño o la posición de un elemento no generan nuevos candidatos LCP. Solo se considera el tamaño y la posición inicial del elemento en la ventana gráfica.

Esto significa que es posible que no se informen las imágenes que inicialmente se procesan fuera de la pantalla y luego pasan a la pantalla. También significa que los elementos renderizados inicialmente en la ventana gráfica que luego se empujan hacia abajo, fuera de la vista, seguirán informando su tamaño inicial en la ventana gráfica.

### Ejemplos de

A continuación, se muestran algunos ejemplos de situaciones en las que ocurre la pintura con contenido más grande en algunos sitios web populares:

{% Img src="image/admin/bsBm8poY1uQbq7mNvVJm.png", alt="Largest Contentful Paint timeline from cnn.com", width="800", height="311" %}

{% Img src="image/admin/xAvLL1u2KFRaqoZZiI71.png", alt="Largest Contentful Paint timeline from techcrunch.com", width="800", height="311" %}

En las dos líneas de tiempo anteriores, el elemento más grande cambia a medida que se carga el contenido. En el primer ejemplo, se agrega contenido nuevo al DOM y eso cambia qué elemento es el más grande. En el segundo ejemplo, el diseño cambia y el contenido que antes era el más grande se elimina de la ventana gráfica.

Si bien a menudo ocurre que el contenido de carga tardía es más grande que el contenido que ya está en la página, ese no es necesariamente el caso. Los dos ejemplos siguientes muestran la pintura con contenido más grande que se produce antes de que la página se cargue por completo.

{% Img src="image/admin/uJAGswhXK3bE6Vs4I5bP.png", alt="Largest Contentful Paint timeline from instagram.com", width="800", height="311" %}

{% Img src="image/admin/e0O2woQjZJ92aYlPOJzT.png", alt="Largest Contentful Paint timeline from google.com", width="800", height="311" %}

En el primer ejemplo, el logotipo de Instagram se carga relativamente pronto y sigue siendo el elemento más grande incluso cuando se muestra progresivamente otro contenido. En el ejemplo de la página de resultados de búsqueda de Google, el elemento más grande es un párrafo de texto que se muestra antes de que se termine de cargar cualquiera de las imágenes o el logotipo. Dado que todas las imágenes individuales son más pequeñas que este párrafo, sigue siendo el elemento más grande durante todo el proceso de carga.

{% Aside %} En el primer fotograma de la línea de tiempo de Instagram, es posible que notes que el logotipo de la cámara no tiene un cuadro verde alrededor. Esto se debe a que es un `<svg>` elemento, y `<svg>` elementos no son actualmente considerados candidatos LCP. El primer candidato LCP es el texto del segundo cuadro. {% endAside %}

## Cómo medir LCP

El LCP se puede medir [en el laboratorio](/user-centric-performance-metrics/#in-the-lab) o [en el campo](/user-centric-performance-metrics/#in-the-field) y está disponible en las siguientes herramientas:

### Herramientas de campo

- [Informe de experiencia del usuario de Chrome](https://developers.google.com/web/tools/chrome-user-experience-report)
- [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)
- [Search Console (informe de Core Web Vitals)](https://support.google.com/webmasters/answer/9205520)
- [biblioteca JavaScript `web-vitals`](https://github.com/GoogleChrome/web-vitals)

### Herramientas de laboratorio

- [DevTools de Chrome](https://developers.google.com/web/tools/chrome-devtools/)
- [Faro](https://developers.google.com/web/tools/lighthouse/)
- [WebPageTest](https://webpagetest.org/)

### Medir LCP en JavaScript

Para medir el LCP en JavaScript, puede utilizar la [API de pintura con contenido más grande](https://wicg.github.io/largest-contentful-paint/) . El siguiente ejemplo muestra cómo crear un [`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) que escucha las `largest-contentful-paint` y las registra en la consola.

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('LCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'largest-contentful-paint', buffered: true});
```

{% Aside 'warning' %}

Este código muestra cómo registrar las `largest-contentful-paint` con mayor contenido en la consola, pero medir LCP en JavaScript es más complicado. Consulte a continuación para obtener más detalles:

{% endAside %}

En el ejemplo anterior, cada `largest-contentful-paint` registrada representa el candidato LCP actual. En general, el `startTime` de la última entrada emitida es el valor LCP; sin embargo, no siempre es así. No todas las `largest-contentful-paint` son válidas para medir LCP.

La siguiente sección enumera las diferencias entre lo que informa la API y cómo se calcula la métrica.

#### Diferencias entre la métrica y la API

- La API enviará `largest-contentful-paint` para las páginas cargadas en una pestaña de fondo, pero esas páginas deben ignorarse al calcular el LCP.
- La API continuará enviando `largest-contentful-paint` después de que una página haya sido puesta en segundo plano, pero esas entradas deben ignorarse al calcular el LCP (los elementos solo se pueden considerar si la página estuvo en primer plano todo el tiempo).
- La API no informa las `largest-contentful-paint` cuando la página se restaura desde la [caché de retroceso / avance](/bfcache/#impact-on-core-web-vitals) , pero el LCP debe medirse en estos casos, ya que los usuarios las experimentan como visitas de página distintas.
- La API no considera elementos dentro de iframes, pero para medir correctamente el LCP debe considerarlos. Los sub-marcos pueden usar la API para informar sus `largest-contentful-paint` con mayor contenido al marco principal para su agregación.

En lugar de memorizar todas estas diferencias sutiles, los desarrolladores pueden usar la [biblioteca JavaScript de `web-vitals`](https://github.com/GoogleChrome/web-vitals) para medir el LCP, que maneja estas diferencias por usted (cuando sea posible):

```js
import {getLCP} from 'web-vitals';

// Measure and log LCP as soon as it's available.
getLCP(console.log);
```

Puede consultar [el código fuente de `getLCP()`](https://github.com/GoogleChrome/web-vitals/blob/master/src/getLCP.ts) para obtener un ejemplo completo de cómo medir LCP en JavaScript.

{% Aside %} En algunos casos (como los iframes de origen cruzado) no es posible medir LCP en JavaScript. Consulte la sección de [limitaciones](https://github.com/GoogleChrome/web-vitals#limitations) `web-vitals` para obtener más detalles. {% endAside %}

### ¿Qué pasa si el elemento más grande no es el más importante?

En algunos casos, el elemento (o elementos) más importantes de la página no es el mismo que el elemento más grande, y los desarrolladores pueden estar más interesados en medir los tiempos de renderizado de estos otros elementos. Esto es posible utilizando la [API Element Timing](https://wicg.github.io/element-timing/) , como se describe en el artículo sobre [métricas personalizadas](/custom-metrics/#element-timing-api) .

## Cómo mejorar LCP

El LCP se ve afectado principalmente por cuatro factores:

- Tiempos de respuesta del servidor lentos
- JavaScript y CSS que bloquean el renderizado
- Tiempos de carga de recursos
- Representación del lado del cliente

Para profundizar en cómo mejorar LCP, consulte [Optimizar LCP](/optimize-lcp/) . Para obtener orientación adicional sobre técnicas de rendimiento individual que también pueden mejorar LCP, consulte:

- [Aplicar carga instantánea con el patrón PRPL](/apply-instant-loading-with-prpl)
- [Optimización de la ruta de renderización crítica](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/)
- [Optimiza tu CSS](/fast#optimize-your-css)
- [Optimiza tus imágenes](/fast#optimize-your-images)
- [Optimizar las fuentes web](/fast#optimize-web-fonts)
- [Optimice su JavaScript](/fast#optimize-your-javascript) (para sitios renderizados por el cliente)

## Recursos adicionales

- [Lecciones aprendidas de la supervisión del rendimiento en Chrome](https://youtu.be/ctavZT87syI) por [Annie Sullivan](https://anniesullie.com/) en [performance.now ()](https://perfnow.nl/) (2019)

{% include 'content/metrics/metrics-changelog.njk' %}
