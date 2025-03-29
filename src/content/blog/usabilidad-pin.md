---
title: 'Un día cualquiera como desarrollador (¿y diseñador?) en Buda.com'
description: ''
pubDate: 'Mar 18 2024'
heroImage: '/blog-placeholder-3.jpg'
---

Este post está también en 🔗 [el blog de Buda.com](https://blog.buda.com/un-dia-cualquiera-como-desarrollador-y-disenador-en-buda-com/)

---

Cuando hablamos de diseño, muchos piensan en colores, íconos y pantallas bonitas. Pero diseñar también significa entender a fondo los problemas del usuario y encontrar la mejor manera de solucionarlos. Y a veces, quienes escribimos el código somos los mismos que tenemos que pensar cómo hacerlo más claro, más útil y más humano. Este es un ejemplo de cómo una pequeña confusión en la app llevó a una gran mejora... y cómo el camino para solucionarlo fue mucho más que solo escribir líneas de código.

La app de [Buda.com](http://buda.com/) entrega la opción de usar un PIN de 6 dígitos para aprobar envíos de criptomonedas de manera rápida y segura. Implementar una funcionalidad así no es fácil, sobre todo considerando que en algo tan sensible debe siempre primar la seguridad. Por eso cuando lo lanzamos decidimos empezar con una versión inicial, y luego ir identificando mejoras a partir de los comentarios de nuestros usuarios. Así nos dimos cuenta de que había un mensaje en particular que resultaba confuso.

Cuando un usuario con PIN de seguridad activado solicitaba un envío a través de la app, en ciertas ocasiones le mostrábamos la siguiente advertencia, que lamentablemente no era muy útil:

<div style="text-align: center;">
  <img src="/pin-1.png" alt="Advertencia PIN antes" style="max-width: 300px;" />
</div>

🤷 ¿Cuál es mi dispositivo aprobador? ¿Y si no lo tengo qué hago? ¿Por qué me habla de un *retiro* si estoy en el flujo de *envío*? ¿La opción *ir al home* de qué me sirve? ¿Por qué dice *van en camino* si aún me falta un paso?

Acá te cuento cómo mejoré el mensaje basándome en las [🔗 10 heurísticas de usabilidad de Nielsen](https://www.nngroup.com/articles/ten-usability-heuristics/). Éstas son reglas generales para un correcto diseño de interfaces de usuario. Te recomiendo revisar el link antes de seguir leyendo, es cortito, aunque de todas maneras voy a ir mencionando las heurísticas acá mismo. De ahora en adelante uso el emoji ☝️ para referirme al artículo.

Antes de empezar, quiero mencionar que soy desarrollador, no diseñador, así que siéntete en todo el derecho de cuestionar lo que digo. Pero sí trabajo muy cerca del diseño de productos digitales, me interesa mucho el tema, y lo he estudiado bastante.

### Cómo mejorar el mensaje al usuario

☝️ En primer lugar vamos a aplicar la heurística 9: Ayuda a los usuarios a reconocer, diagnosticar y recuperarse de errores. *Los mensajes de error deben expresarse en un lenguaje sencillo (sin códigos de error), indicar con precisión el problema y sugerir de forma constructiva una solución.*

Basándonos en esto, el mensaje debería comunicar:

1. La situación actual: *El PIN que tienes configurado ya no es válido*, que puede ser por dos motivos:
    
    a) *Activaste el PIN en otro dispositivo.*
    
    b) O bien *Se perdió la sesión en que activaste el PIN.*
    
    Con “perder la sesión” me refiero a distintos casos en que se pierde el acceso sin que hayas pinchado el botón *Cerrar sesión*. Puede ser por ejemplo si desinstalaste la app. 
    
    Este caso no es tan simple de identificar, porque lo único que sabemos es que el usuario no ha usado esa sesión en un cierto tiempo.
    
2. Y la acción requerida de parte del usuario para cada uno de los casos:
    
    a) *Abre la app en el otro dispositivo para poder aprobar el envío.*
    
    b) O bien *Vuelve a configurar el PIN de seguridad.*
    

Lo ideal sería detectar si es el caso a) o el b), y mostrar un mensaje correspondiente. Pero como mencioné, hacer esta distinción no es tan sencillo, así que para una primera versión opté por una solución más fácil de implementar.

Sumado a esto tenemos que asegurarnos de mantener la consistencia usando la palabra *envío* en vez de la palabra *retiro*. 

☝️ Esto responde a la heurística 4: Coherencia y estándares. *Los usuarios no deberían tener que preguntarse si diferentes palabras, situaciones o acciones significan lo mismo. Sigue las convenciones de la plataforma y del sector.*

Y también tenemos que cambiar el título *Van en camino* para reflejar mejor el hecho de que aún falta un paso.

☝️Esto responde a la heurística 1: Visibilidad del estado del sistema. *El diseño debe mantener siempre informados a los usuarios sobre lo que está sucediendo.*

### Así quedó finalmente

Tomando todo esto en cuenta mejoré el título, el mensaje de advertencia y reemplacé la acción sugerida por el botón principal. Ahora el usuario entiende mucho mejor qué está pasando, y cómo solucionarlo:

<div style="text-align: center;">
  <img src="/pin-2.png" alt="Advertencia PIN después" style="max-width: 300px;" />
</div>