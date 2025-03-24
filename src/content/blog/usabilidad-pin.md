---
title: 'Aplicando conceptos de usabilidad para mejorar la experiencia en Buda.com'
description: ''
pubDate: 'Mar 18 2024'
heroImage: '/blog-placeholder-1.jpg'
---

### Contexto

Cuando un usuario con PIN de seguridad activado solicita un envío a través de la app, en ciertas ocasiones le mostramos la siguiente advertencia, que lamentablemente no es muy útil.

<aside>
🤷 ¿Cuál es mi dispositivo aprobador? ¿Y si no lo tengo qué hago? ¿Por qué me habla de un <i>retiro</i> si estoy en el flujo de <i>envío</i>? ¿La opción <i>ir al home</i> de qué me sirve? ¿Por qué dice <i>van en camino</i> si me falta un paso?
</aside>

Acá les cuento cómo lo mejoré y aprovecho de asociarlo a las [🔗 10 heurísticas de usabilidad de Nielsen](https://www.nngroup.com/articles/ten-usability-heuristics/) (lectura muy recomendada), que son reglas generales para un correcto diseño de interfaces de usuario. Idealmente revisen el link antes de seguir leyendo, es cortito, aunque de todas maneras voy a ir mencionando las heurísticas acá mismo. De ahora en adelante uso el emoji ☝️ para hacer referencia al artículo.

Disclaimer: no soy diseñador, así que tampoco me crean taanto tanto lo que digo acá.

<div style="text-align: center;">
  <img src="/pin-1.png" alt="Advertencia PIN antes" style="max-width: 300px;" />
</div>

### Estados del PIN

Para entender cómo mejorarla hay que entender los distintos estados del PIN. Después de que el usuario activa su PIN, éste puede estar:

1. **Habilitado** si la sesión en la app (más concretamente la API key) con la que lo activó aún está activa. Esto puede corresponder a alguno de los siguientes casos:
    
    a) El usuario sigue usando el mismo dispositivo (caso esperado o <i>happy path</i>)
    
    b) O bien el usuario inició sesión en un segundo dispositivo ⚠️
    
    A esto se refiere la advertencia con <i>ir a tu dispositivo aprobador</i>. El usuario está en un segundo dispositivo pero el PIN está en el primero.
    
    c) O bien el usuario "perdió" la sesión en ese dispositivo y luego volvió a iniciarla ⚠️
    
    "Perder" la sesión se refiere a distintos casos en que se pierde el acceso sin haber pinchado el botón <i>Cerrar sesión</i>. Puede ser por ejemplo con el botón <i>Olvidé mi PIN</i> o desinstalando la app.
    
    Por motivos técnicos en esos casos no tenemos cómo comprobar que la sesión ya no está activa. Lo que hacemos es asumir que se perdió luego de que pasa un cierto tiempo de desuso.
    
2. **No habilitado** si la sesión en la app dejó de estar activa. Puede ser porque:
    
    d) El usuario cerró sesión correctamente con el botón <i>Cerrar sesión</i>.
    
    e) O bien porque pasó el tiempo máximo de desuso y marcamos automáticamente esa sesión como inactiva.
    

Los casos marcados con ⚠️ son los que hay que abordar en el mensaje <i>Para aprobar este retiro debes ir a tu dispositivo aprobador.</i>

### Cómo mejorar el mensaje <i>Para aprobar este retiro debes ir a tu dispositivo aprobador</i>

Basándome en la lectura recomendada, determiné que el mensaje debería comunicar:

1. La situación actual: <i>El PIN que tienes configurado no es válido</i>, que puede deberse a alguno de los casos marcados con ⚠️:
    
    a) <i>Activaste el PIN en otro dispositivo.</i>
    
    b) O bien <i>Iniciaste una nueva sesión en este dispositivo.</i>
    
2. La acción requerida del usuario:
    
    a) <i>Abre la app en el otro dispositivo para poder aprobar el envío.</i>
    
    b) O bien <i>Vuelve a configurar el PIN de seguridad.</i>

<aside>
<b>Esto responde a la heurística 9: Help Users Recognize, Diagnose, and Recover from Errors</b>
</aside>

Además en el caso b) una vez configurado el PIN nuevamente se cancelan automáticamente los envíos asociados al PIN anterior. Es necesario explicar esto claramente y mencionar que debe volver a solicitar el envío.

<aside>
<b>Esto responde a la heurística 1: Visibility of System Status. The design should always keep users informed about what is going on, through appropriate feedback within a reasonable amount of time.</b>
</aside>

También debemos asegurarnos de usar la palabra <i>envío</i> en vez de la palabra <i>retiro</i>.

<aside>
<b>Esto responde a la heurística 4: Consistency and Standards. Users should not have to wonder whether different words, situations, or actions mean the same thing.</b>
</aside>

### Cambios implementados

Con los nuevos cambios la pantalla queda así:

<div style="text-align: center;">
  <img src="/pin-2.png" alt="Advertencia PIN después" style="max-width: 300px;" />
</div>


Y luego de completar el flujo de configuración del nuevo PIN se ve este mensaje:

<div style="text-align: center;">
  <img src="/pin-3.png" alt="Advertencia PIN después 2" style="max-width: 300px;" />
</div>

### Soluciones descartadas (por ahora al menos)

Decidí no ofrecer la acción de <i>Ir al home</i> que estaba antes, porque no soluciona nada y ya hay una X en la esquina de la pantalla que hace eso mismo.

Además consideré agregar esto al mensaje:

> Para más información revisa [este artículo](https://soporte.buda.com/es/articles/7257832-todo-sobre-el-pin-de-seguridad-para-retiros-cripto) o escríbenos a soporte@buda.com
> 

<aside>
<b>Esto respondería a la heurística 10: Help and Documentation: It's best if the system doesn't need any additional explanation. However, it may be necessary to provide documentation to help users understand how to complete their tasks.</b>
</aside>

Pero creo que es mucho texto. El mensaje <i>sin</i> eso último ya está medio largo (que hace menos probable que el usuario lo lea) y en el iPhone SE cabe justo:

<div style="text-align: center;">
  <img src="/pin-4.png" alt="Advertencia PIN en celular pequeño" style="max-width: 300px;" />
</div>

<aside>
<b>Finalmente primó la heurística 8: Aesthetic and Minimalist Design: Interfaces should not contain information that is irrelevant or rarely needed. Every extra unit of information in an interface competes with the relevant units of information and diminishes their relative visibility.</b>
</aside>

Además hay dos cambios que dejarían todo aún más claro para el usuario, pero que por motivos más prácticos descarté por el momento:

1. Mostrar distintos mensajes para las dos situaciones posibles. Implicaría cambios mayores para poder diferenciarlas.

<aside>
<b>Esto mejoraría aún más la heurística 9: Help Users Recognize, Diagnose, and Recover from Errors</b>
</aside>

1. Mostrar la advertencia <i>al inicio</i> del flujo de retiro cripto, para ayudar al usuario a darse cuenta del problema y resolverlo <i>antes</i> de crear el retiro. Descartado porque en ese momento aún no se sabe si es que el backend va a pedir PIN para ese retiro, implicaría cambios mayores.

<aside>
<b>Esto respondería a la heurística 5: Error Prevention. Good error messages are important, but the best designs carefully prevent problems from occurring in the first place. Either eliminate error-prone conditions, or check for them and present users with a confirmation option before they commit to the action.</b>
</aside>

## ¿Te interesó el tema?

[10 Usability Heuristics for User Interface Design](https://www.nngroup.com/articles/ten-usability-heuristics/)

En el mismo artículo hay hartos links y recursos para indagar más. Por ejemplo este mini torpedo con las heurísticas:

<div style="text-align: center;">
  <img src="/pin-5.png" alt="Heurísticas" style="max-width: 300px;" />
</div>