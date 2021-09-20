---
layout: post
title: Métricas de rendimiento centradas en el usuario
authors:
  - philipwalton
date: '2019-11-08'
description: |2

  Las métricas de rendimiento centradas en el usuario son una herramienta fundamental para comprender y

  mejorar la experiencia de su sitio de una manera que beneficie a los

  usuarios.
tags:
  - performance
  - metrics
---

Todos hemos escuchado lo importante que es el desempeño. Pero cuando hablamos de rendimiento y de hacer que los sitios web sean "rápidos", ¿a qué nos referimos específicamente?

La verdad es que el rendimiento es relativo:

- Un sitio puede ser rápido para un usuario (en una red rápida con un dispositivo potente) pero lento para otro usuario (en una red lenta con un dispositivo de gama baja).
- Es posible que dos sitios terminen de cargarse exactamente en la misma cantidad de tiempo, pero puede *parecer* que uno se carga más rápido (si carga el contenido de forma progresiva en lugar de esperar hasta el final para mostrar algo).
- *Puede parecer* que un sitio se carga rápidamente, pero luego responde lentamente (o no responde) a la interacción del usuario.

Entonces, cuando se habla de desempeño, es importante ser preciso y referirse al desempeño en términos de criterios objetivos que se pueden medir cuantitativamente. Estos criterios se conocen como *métricas* .

Pero el hecho de que una métrica se base en criterios objetivos y pueda medirse cuantitativamente no significa necesariamente que esas medidas sean útiles.

## Definición de métricas

Históricamente, el rendimiento web se ha medido con el evento <code>[load](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event)</code> Sin embargo, aunque la <code>load</code> es un momento bien definido en el ciclo de vida de una página, ese momento no se corresponde necesariamente con nada que le importe al usuario.

Por ejemplo, un servidor podría responder con una página mínima que se "carga" inmediatamente, pero luego pospone la búsqueda de contenido y muestra cualquier cosa en la página hasta varios segundos después de que se `load` evento de carga. Si bien una página de este tipo técnicamente podría tener un tiempo de carga rápido, ese tiempo no correspondería a cómo un usuario realmente experimenta la carga de la página.

En los últimos años, los miembros del equipo de Chrome, en colaboración con el [Grupo de trabajo de rendimiento web](https://www.w3.org/webperf/) del W3C, han estado trabajando para estandarizar un conjunto de nuevas API y métricas que miden con mayor precisión cómo los usuarios experimentan el rendimiento de una página web.

Para ayudar a garantizar que las métricas sean relevantes para los usuarios, las enmarcamos en torno a algunas preguntas clave:

<table id="questions">
  <tr>
    <td><strong>¿Está sucediendo?</strong></td>
    <td>¿La navegación se inició correctamente? ¿Ha respondido el servidor?</td>
  </tr>
  <tr>
    <td><strong>¿Es útil?</strong></td>
    <td>¿Se ha renderizado suficiente contenido para que los usuarios puedan interactuar con él?</td>
  </tr>
  <tr>
    <td><strong>¿Es utilizable?</strong></td>
    <td>¿Pueden los usuarios interactuar con la página o está ocupada?</td>
  </tr>
  <tr>
    <td><strong>¿Es delicioso?</strong></td>
    <td>¿Son las interacciones suaves y naturales, libres de retrasos y jank?</td>
  </tr>
</table>

## Cómo se miden las métricas

Las métricas de rendimiento generalmente se miden de dos maneras:

- **En el laboratorio:** uso de herramientas para simular la carga de una página en un entorno controlado y uniforme
- **En el campo** : en usuarios reales que realmente cargan e interactúan con la página

Ninguna de estas opciones es necesariamente mejor o peor que la otra; de hecho, generalmente desea utilizar ambas para garantizar un buen rendimiento.

### En el laboratorio

Probar el rendimiento en el laboratorio es esencial al desarrollar nuevas funciones. Antes de que las funciones se publiquen en producción, es imposible medir sus características de rendimiento en usuarios reales, por lo que probarlas en el laboratorio antes del lanzamiento de la función es la mejor manera de evitar regresiones de rendimiento.

### En el campo

Por otro lado, si bien las pruebas en el laboratorio son un indicador razonable del rendimiento, no reflejan necesariamente cómo todos los usuarios experimentan su sitio en la naturaleza.

El rendimiento de un sitio puede variar drásticamente según las capacidades del dispositivo del usuario y las condiciones de su red. También puede variar en función de si (o cómo) un usuario está interactuando con la página.

Además, las cargas de páginas pueden no ser deterministas. Por ejemplo, los sitios que cargan contenido o anuncios personalizados pueden experimentar características de rendimiento muy diferentes de un usuario a otro. Una prueba de laboratorio no capturará esas diferencias.

La única forma de saber realmente el rendimiento de su sitio para sus usuarios es medir realmente su rendimiento a medida que esos usuarios lo cargan e interactúan con él. Este tipo de medición se conoce comúnmente como [Monitoreo de usuarios reales,](https://en.wikipedia.org/wiki/Real_user_monitoring) o RUM para abreviar.

## Tipos de métricas

Hay varios otros tipos de métricas que son relevantes para cómo los usuarios perciben el rendimiento.

- **Velocidad de carga percibida:** la rapidez con la que una página puede cargar y representar todos sus elementos visuales en la pantalla.
- **Capacidad de respuesta de carga:** la rapidez con la que una página puede cargar y ejecutar cualquier código JavaScript requerido para que los componentes respondan rápidamente a la interacción del usuario.
- **Capacidad de respuesta en tiempo de ejecución:** después de la carga de la página, ¿qué tan rápido puede responder la página a la interacción del usuario?
- **Estabilidad visual:** ¿los elementos de la página cambian de una forma que los usuarios no esperan e interfieren potencialmente con sus interacciones?
- **Suavidad:** ¿las transiciones y animaciones se procesan a una velocidad de fotogramas constante y fluyen con fluidez de un estado al siguiente?

Dados todos los tipos de métricas de rendimiento anteriores, es de esperar que esté claro que ninguna métrica es suficiente para capturar todas las características de rendimiento de una página.

## Métricas importantes para medir

- **[Primera pintura con contenido (FCP)](/fcp/) :** mide el tiempo desde que la página comienza a cargarse hasta que cualquier parte del contenido de la página se representa en la pantalla. *( [laboratorio](#in-the-lab) , [campo](#in-the-field) )*
- **[Pintura con contenido más grande (LCP)](/lcp/) :** mide el tiempo desde que la página comienza a cargarse hasta que el bloque de texto o elemento de imagen más grande se representa en la pantalla. *( [laboratorio](#in-the-lab) , [campo](#in-the-field) )*
- **[Retraso de la primera entrada (FID)](/fid/) :** mide el tiempo desde que un usuario interactúa por primera vez con su sitio (es decir, cuando hace clic en un enlace, toca un botón o usa un control personalizado impulsado por JavaScript) hasta el momento en que el navegador realmente puede para responder a esa interacción. *( [campo](#in-the-field) )*
- **[Time to Interactive (TTI)](/tti/) :** mide el tiempo desde que la página comienza a cargarse hasta que se procesa visualmente, se han cargado sus scripts iniciales (si los hay) y es capaz de responder de manera confiable a la entrada del usuario rápidamente. *( [laboratorio](#in-the-lab) )*
- **[Tiempo de bloqueo total (TBT)](/tbt/) :** mide la cantidad total de tiempo entre FCP y TTI donde el hilo principal estuvo bloqueado durante el tiempo suficiente para evitar la respuesta de entrada. *( [laboratorio](#in-the-lab) )*
- **[Cambio de diseño acumulativo (CLS)](/cls/) :** mide la puntuación acumulada de todos los cambios de diseño inesperados que se producen entre el momento en que la página comienza a cargarse y el momento en que el [estado de su ciclo de vida](https://developers.google.com/web/updates/2018/07/page-lifecycle-api) cambia a oculto. *( [laboratorio](#in-the-lab) , [campo](#in-the-field) )*

Si bien esta lista incluye métricas que miden muchos de los diversos aspectos del rendimiento relevantes para los usuarios, no lo incluye todo (por ejemplo, la capacidad de respuesta en tiempo de ejecución y la fluidez no están cubiertas actualmente).

En algunos casos, se introducirán nuevas métricas para cubrir las áreas que faltan, pero en otros casos, las mejores métricas son las que se adaptan específicamente a su sitio.

## Métricas personalizadas

Las métricas de rendimiento enumeradas anteriormente son buenas para obtener una comprensión general de las características de rendimiento de la mayoría de los sitios en la web. También son buenos para tener un conjunto común de métricas para que los sitios comparen su desempeño con el de sus competidores.

Sin embargo, puede haber ocasiones en las que un sitio específico sea único de alguna manera y requiera métricas adicionales para capturar la imagen completa del rendimiento. Por ejemplo, la métrica LCP está destinada a medir cuándo ha terminado de cargarse el contenido principal de una página, pero puede haber casos en los que el elemento más grande no forme parte del contenido principal de la página y, por lo tanto, LCP puede no ser relevante.

Para abordar estos casos, el Grupo de trabajo de rendimiento web también ha estandarizado API de nivel inferior que pueden ser útiles para implementar sus propias métricas personalizadas:

- [API de tiempo de usuario](https://w3c.github.io/user-timing/)
- [API de tareas largas](https://w3c.github.io/longtasks/)
- [API de sincronización de elementos](https://wicg.github.io/element-timing/)
- [API de tiempo de navegación](https://w3c.github.io/navigation-timing/)
- [API de tiempo de recursos](https://w3c.github.io/resource-timing/)
- [Tiempo del servidor](https://w3c.github.io/server-timing/)

Consulte la guía sobre [métricas personalizadas](/custom-metrics/) para saber cómo utilizar estas API para medir las características de rendimiento específicas de su sitio.
