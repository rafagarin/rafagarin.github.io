---
title: 'Aprendizajes del desarrollo de la verificación en Buda.com'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Jun 01 2024'
heroImage: '/blog-placeholder-5.jpg'
---

# Intro

Mientras desarrollaba la feature de verificación aproveché de pensar harto en cómo ordenar bien nuestro código en el frontend. Al mismo tiempo estaba leyendo el libro [🔗 Modern Software Engineering](https://www.goodreads.com/book/show/57345270-modern-software-engineering) que habla de conceptos fundamentales de ingeniería de software que me ayudaron mucho a darle un poco más de formalidad a mis ideas.

Acá dejo mis aprendizajes. Parto explicando el contexto de la feature construida y mi planificación original. Luego menciono los ajustes que fui haciendo en el camino y las ventajas y desventajas que vi en esta manera de ordenar el código. Al final dejo una propuesta actualizada en base a estos aprendizajes y mis ideas sobre cómo avanzar en esta dirección. Le puse harto cariño, ¡espero que les sirva!

# Contexto

Una de las metas del squad este semestre era rediseñar el flujo de verificación para que sea más amigable para el usuario. Para hacerse una idea general creo que sirve mejor ver esta demo del MVP:


<div style="text-align: center;">
  <img src="/verificacion-1.gif" alt="Demo de la verificación" style="max-width: 600px;" />
</div>

Tal vez no es evidente de inmediato, pero construir el front de una feature como esta involucra mucha complejidad. Hay 36 campos distintos, pero no todos se muestran siempre, sino que muchos dependen variables como el país y si el usuario le corresponde hacer el flujo con biometría o no. Muchos de estos campos dependen de otros campos para sus valores por defecto, opciones posibles, labels, etc. Hay 18 pantallas distintas con navegación entre ellas que también depende de qué flujo le toca al usuario. Hay que tener en mente los distintos estados de la verificación, considerando no sólo el happy path de revisión aprobada sino que también estados pendientes, rechazos, y revisiones manuales.

Nuestro sitio es muchísimo más complejo de lo que podría caber en la mente de una sóla persona en un momento, al menos en la mía. Y por sobre eso hay que tener en mente que en el futuro llegarán a modificar el código personas que van a tener mucho menos contexto que quien lo escribió originalmente. 

¿Cómo manejar esa complejidad? ¡Manteniendo ordenado nuestro código! Teniendo en mente técnicas como separación de responsabilidades, abstracción, acoplamiento y cohesión 🤓.

# Propuesta original

Cuando empecé a desarrollar la feature compartí mis ideas en un RFC. En esta sección quiero repasar esa propuesta original y en las siguientes explicar qué modificaciones tuvimos que hacerle y si sirvió o no.

Para entender la feature sirve entender el esquema principal y definir nombres para cada concepto. A grandes rasgos: El flujo consiste en varias **etapas**. Cada etapa tiene varios **pasos**, y cada paso tiene varios **campos**.


Esta es la pantalla que muestra la lista de etapas:

<div style="text-align: center;">
  <img src="/verificacion-2.png" alt="Demo de la verificación" style="max-width: 600px;" />
</div>


Y este es uno de los pasos:

<div style="text-align: center;">
  <img src="/verificacion-3.png" alt="Demo de la verificación" style="max-width: 600px;" />
</div>

Este paso en particular tiene un sólo campo, el país del documento. Y a la derecha se ve su posición en la etapa actual.

### Responsabilidades

La propuesta consiste en separar claramente las disintas responsabilidad. En principio podemos distinguir las siguientes:

- Comunicación con el backend
- Lógica de decidir qué **campos** mostrar en cada caso
- Interfaz de cada **paso**
- Navegación entre **pasos** y **etapas**

Se separan estas responsabilidades 💡 en distintos archivos, ordenados en tres categorías:

**Capa de interacción con el backend**

Las responsabilidades principales de esta capa son:

1. Leer el backend, a través de un hook que llamamos `useCurrentVerificationState`.
2. Mutar el backend, a través de un hook que llamamos `useUpdateVerificationState`.

**Capa de lógica.**

Las responsabilidades principales de esta capa son:

1. Conocer las reglas sobre cuándo hay que mostrar cada campo, a través de `verificationFieldsConfig`:
2. Saber si hay que mostrar un campo en este momento, a través de  `useVerificationFieldsConfig`:

**Capa de interfaz**

Las responsabilidades principales de esta capa son:

1. La configuración de la navegación, a través de `verificationNavigationConfig`
2. La navegación misma, a través de `useVerificationNavigation`
3. Mostrar las vistas, a través de los componentes `____Step`:

Si hacemos un diagrama de la interacción entre estos distintos módulos, queda así: 


<div style="text-align: center;">
  <img src="/verificacion-4.png" alt="Separación de archivos" style="max-width: 600px;" />
</div>


(Amarillo: sólo JS/TS, sin React. Azul: hooks de React. Verde: componentes de React)

Podemos notar que todas las dependencias entre capas van hacia abajo: La capa de lógica no sabe nada de la interfaz, mientras que el backend no sabe nada ni de lógica ni de interfaz.

Por otro lado la interfaz sí va a conocer las capas inferiores, aunque nos vamos a asegurar de que conozca lo menos posible el detalle de *cómo* funcionan por detrás.

También podemos notar que la navegación está arriba de los steps. A pesar de que están en la misma capa, esto tiene sentido. La navegación necesita conocer los steps (para saber en qué orden van y cuándo mostrar cada uno), pero un step no tiene por qué saber de navegación (sólo llama a la función `onSubmit` que le entrega el hook de navegación).

# Beneficios

Mi hipótesis era que esta estructura nos iba a servir porque se hace más fácil:

- Entender el código.
- Hacer cambios en el futuro.
- Reutilizar el código entre distintas partes del frontend.
- El testing unitario.

¿Se cumplen estas ventajas? ¿Y cuáles podrían ser las desventajes?

### Más fácil entender

Yo opino que sí se cumple.

Vuelvo a usar un paso como ejemplo:

<div style="text-align: center;">
  <img src="/verificacion-3.png" alt="Demo de la verificación" style="max-width: 600px;" />
</div>

El código de este componente es bien simple porque sólo tiene responsabilidades de interfaz, a pesar de que por detrás representa toda la complejidad de la navegación y la interacción con el backend. Es lo hace mucho más fácil de trabajar.

Cada una de las partes se hace fácil de entender gracias a que tienen una solo responsabilidad, es decir tienen alta cohesión 💡.

### Más fácil reutilizar

Check ✔️

Reutilizamos algunas funciones de navegación para mostrar el estado de la verificación en la barra lateral derecha del inicio: 

<div style="text-align: center;">
  <img src="/verificacion-5.png" alt="Demo de la verificación" style="max-width: 600px;" />
</div>

Y nos funcionó bastante bien. Nuevamente la abstracción fue útil para llegar y usar las funciones sin preocuparse de cómo funcionan por dentro.

### Más fácil de modificar

Definitivamente. Creo que este es el beneficio más claro.

Digamos por ejemplo que queremos cambiar el orden de los pasos. En este momento la prueba de vida se hace antes que los formularios, pero eso es algo que podríamos querer cambiar. ¿Sería muy difícil? Nop. Como los paso son agnósticos a la navegación (simplemente se encargan de llamar a la función de navegación `onSubmit` cuando el usuario pincha el botón principal), bastaría con modificar el archivo de configuración de la navegación.

O digamos que queremos cambiar el endpoint que usamos para guardar el progreso parcial del usuario. En ese caso bastaría con modificar el hook de interacción con el backend, y el cambio quedaría funcional para toda la feature.

Si queremos usar una librería de navegación basta con modificar el hook de navegación.

Si queremos reemplazar una librería de interfaz se haría un poco más difícil porque está en muchos pasos, pero al menos sabemos que basta con modificar esos componentes de la interfaz y nada más.

Si quiero mover un campo de un paso a otro, basta con mover el input que renderiza ese paso. Toda la lógica de decidir si mostrarlo o y de actualizarlo en el backend vive en otras partes.

Esta facilidad para modificar el código es punto es importantísimo porque la interfaz varía mucho en el tiempo. En el caso de la verificación este es el segundo gran rediseño en poco más de un año. E incluso obviando esos grandes cambios estamos constantemente haciendo mini ajustes y mejoras. Otras features también cambian mucho.

Podemos ser mucho más productivos si es que al cambiar la interfaz estamos efectivamente sólo cambiando la interfaz.

### Más fácil testear

Sip.

Si estoy testeando un componente de front, sólo me preocupo de la interfaz y su interacción con otros componentes. Por ejemplo este es el componente para seleccionar el país, junto con sus tests:

<div style="text-align: center;">
  <img src="/verificacion-3.png" alt="Demo de la verificación" style="max-width: 600px;" />
</div>

<div style="text-align: center;">
  <img src="/verificacion-6.png" alt="Demo de la verificación" style="max-width: 600px;" />
</div>

Si estoy testeando un hook de lógica se me hace la vida más fácil porque de lo más apestoso al testear front es tener que andar revisando el DOM e interactuando con botones y cosas así. Acá me olvido de todo eso. Por ejemplo, estos son los tests del hook de configuración de campos:

<div style="text-align: center;">
  <img src="/verificacion-7.png" alt="Demo de la verificación" style="max-width: 600px;" />
</div>

Sólo me preocupo de revisar un booleano al final.

Y si estoy testeando la interacción con el backend, sólo me preocupo de la interacción con el backend. Por ejemplo, estos son los tests del hook que actualiza el backend:

<div style="text-align: center;">
  <img src="/verificacion-8.png" alt="Demo de la verificación" style="max-width: 600px;" />
</div>

Ahora bien, hay que considerar que esta manera muy unitaria de escribir tests no es la única. También es válido tener tests más de integración/E2E.

# Desventajas

### ~~Complejidad extra~~

Mi principal hipótesis con respecto a las desventajas era que iba a agregar complejidad extra. Y la verdad yo diría que esto no se cumplió. Más bien esta manera de ordenar el código nos ofrece mejores maneras de *representar* la complejidad que ya existe inherentemente a la propia feature. 

Por ejemplo la función `shouldSkipScreen` del hook `useVerificationNavigation` incluye condiciones que leen la configuración así:

```tsx
const screenConfig = verificationScreensConfig[screenName];
const shouldSkipBecauseMethodIsFallback =
  screenConfig?.skipIf?.includes('methodIsFallback') &&
  verificationMethod === 'fallback';
```

El string `methodIsFallback` pertenece a un type que define todos los motivos por el cual se podría saltar una vista:

```tsx
type StepSkipReasons =
  | 'methodIsExternalIdentityVerification'
  | 'livenessCheckCompleted'
  | 'hasVerificationProgress'
  | 'residenceProofUploaded'
  | 'methodIsFallback';
```

Este type es algo propio de la comunicación entre el hook y la configuración, y no existiría si es que no hubiéramos hecho esa separación. Se podría considerar complejidad extra, pero creo que cumple una función muy útil y el código termina quedando más claro gracias a este tipo de definiciones, sobre todos si usamos TypeScript y buenos nombres.

### Hay que comunicarlo

Otro problema que hay que considerar es que este orden del código no queda inmediatamente claro para otros devs que lleguen a modificar la feature.

Por ejemplo, hay casos en que el estado de una etapa depende del estado de la solicitud de verificación. Digamos que queremos representar esto en una función `getStageStatusFromConfirmationAttempt(stage, confirmationAttempt)`. ¿Dónde iría esta función?

En principio tiene sentido dejarla en `currentVerificationStateUtils` por su relación con los `confirmationAttempts`. Pero en realidad la función incluye el concepto de `stage` que es propio de la navegación y por tanto no debe mezclarse con temas de backend. La respuesta es agregarla en `useVerificationNavigation`. ¿Pero cómo comunicarlo para que se cumpla? Esto es uno de los desafíos de esta propuesta.

# Propuesta

¿Con qué nos podemos quedar de todo esto?

En primer lugar, los conceptos fundamentales de separación de responsabilidades, abstracción, acoplamiento y cohesión que fui mencionando aplican **siempre**. No solo para esta feature, y no solo para el front, sino que cada vez que escribimos código. Y sobre todo si es que estamos escribiendo código que van a ver otras personas y que se va a mantener en el tiempo.

Avanzando de a poco hacia lo más concreto, el concepto de separar la interfaz de temas que no dependen de ella creo que aplica de manera general para cualquier proyecto de frontend.

Y por último, la separación de temas de este caso en particular (navegación, interfaz, lógica y comunicación con el backend) creo que es muy aplicable para todas nuestras features que siguen esta estructura de inputs y pasos, como remesas, vista simple, perfilamiento, y por supuesto verificación. Siempre teniendo en consideración que cada una es distinta y podría tener complejidades que no estoy considerando acá.

Todo esto es fundamental, pero si buscamos una conclusión simple y concreta propongo que sea esto:

<aside>
💡 Separemos nuestro código front en dos: por un lado todo lo que es propio de la interfaz, y por otro lado todo lo que se mantendría igual independiente de la interfaz.

</aside>

Es decir, de las tres capas que había propuesto originalmente, propongo mantener sólo dos. ¿Por qué? Porque es un buen equilibrio entre lo fácil y lo útil, el 20% del esfuerzo que nos entregaría el 80% de los beneficios ([🔗 Principio de Pareto](https://es.wikipedia.org/wiki/Principio_de_Pareto)).

Entonces cada vez que agrego o modifico algo en el front me tengo que preguntar: **¿Esto es código que podría compartir con la app o bien mantener en caso de hacer un rediseño visual?** Si la respuesta es sí, lo mantengo separado de la interfaz.