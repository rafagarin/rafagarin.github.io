---
title: 'Visión para el frontend'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Aug 22 2022'
heroImage: '/blog-placeholder-4.jpg'
---

---

# Intro

Esta guía incluye nuestra visión sobre cómo queremos que se vea nuestro frontend en el futuro.

Se divide esta visión en cinco temas principales: las herramientas que apoyan nuestro desarrollo, los patrones que ordenan nuestro código, las estrategias de rendering que usamos, nuestro uso de type checking, y una última sección para otras consideraciones más aisladas.

En cada uno de estos temas se proponen buenas prácticas, comentando sobre los beneficios que nos pueden traer.

<aside>
A lo largo de la guía usamos emojis para entregar información secundaria que puede complementar el mensaje principal. Si prefieres, puedes ignorarlos. Son los siguientes:

➕ Indica la relación de algún tema con los beneficios de la sección de beneficio.

🔖 Indica la relación de algún tema con *otros temas.*

📖 Lectura recomendada.

`Code highlighting` distingue cosas a nivel de implementación (herramientas, frameworks, librerías), en contraste con conceptos de más alto nivel.

</aside>

# Motivación

¿Por qué es importante todo esto? Distinguimos tres razones que hacen estos temas particularmente importantes en el frontend.

1. El frontend cambia mucho.
    
    Estamos constantemente rediseñando la interfaz de nuestro sitio. Al momento de escribir esto venimos terminando un rediseño completo de la verificación, aún vamos a mitad de camino con el rediseño de la vista simple (lo que llamamos la nueva arquitectura), y estamos preparándonos para rediseñar el historial, la landing principal, y gran parte de la aplicación móvil. E incluso obviando esos grandes cambios estamos constantemente haciendo mini ajustes y mejoras en base al feedback que nos entregan nuestros usuarios.
    
2. En el frontend no hay mucha estructura definida.
    
    En nuestro backend, Rails toma muchas decisiones por nosotros (Rails mismo se define como *opinionated*), por ejemplo dónde va cada responsabilidad (siguiendo patrón MVC) y cómo estructurar el código (los nombres de las carpetas, archivos, etc). En el front nosotros mismos tenemos que tomar esas decisiones.
    
    Rails también nos dice qué dependencias usar y cómo actualizarlas. No todas, pero sí hartas. Por ejemplo para el manejo de datetimes Rails nos entrega clases como [TimeWithZone](https://api.rubyonrails.org/classes/ActiveSupport/TimeWithZone.html). Por otro lado, en front existen varias librerías de manejo de fechas, como date-fns, dayjs, luxon, y moment.
    
3. El frontend está en dos partes: en el sitio web y en la app móvil.
    
    Si no abordamos esto correctamente terminamos haciendo el trabajo dos veces y con riesgo de inconsistencias entre ambas plataformas.
    
    Nos pasa justamente que la app móvil suele quedar relegada. Todas nuestras mejoras, tanto de producto como cosas internas de desarrollo, se aplican primero al sitio web.
    

# Beneficios

En esta guía vamos proponiendo mejoras a nuestro front y especificando los beneficios esperados de éstas. Estos beneficios los categorizamos en:

Mejorar la **experiencia de desarrollo (DX)** a través de:

- Disminuir la **carga cognitiva**.
    
    Es decir disminuir la cantidad de información que es necesario conocer al momento de desarrollar y revisar código. Lo ideal es tener herramientas como linters y generadores de código que hagan toda la pega automatizable.
    
- Acortar el ciclo de **feedback**.
    
    En otras palabras, si me equivoco me debiera dar cuenta lo antes posible. Es mucho mejor que me avise el linter a darnos cuenta cuando ya llegó a prod porque un usuario nos alegó en soporte.
    
- Facilitar la **mantención** del código.
    
    Es decir, queremos disminuir el tiempo que debemos dedicarle a actualización de dependencias y cosas así.
    

Mejorar el **rendimiento del equipo** de desarrollo, a través de:

- Mejorar la **rapidez** de nuestros resultados.
- Disminuir nuestra tasa de **errores**.
    
    Para esta guía basta con entender estos dos puntos conceptualmente, pero si quiesiéramos medirlos existen distintas métricas que podríamos usar.
    
    📖 Lectura recomendada: [DORA metrics](https://dora.dev/guides/dora-metrics-four-keys/) mide factores parecidos enfocándose en los deployments.
    

Mejorar la **experiencia del usuario (UX)** a través de:

- Una experiencia **rápida.**
    
    Esto lo podemos medir usando [web vitals](https://web.dev/articles/vitals).
    
- Una experiencia **clara.**
    
    Esto lo podríamos evaluar siguiendo [principios de usabilidad](https://www.nngroup.com/articles/ten-usability-heuristics/). Acá entramos más en territorio de diseño, pero definitivamente podemos aportar desde desarrollo porque entendemos bien los estados de carga y error, operaciones asíncronas, casos borde, etc.
    
- Una experiencia **consistente.**
    
    Esto tiene que ver por ejemplo con mantener paridad entre el sitio móvil y la app web, en términos de funcionalidades disponibles, aspecto visual, etc.
    

# Parte 1: Herramientas de desarrollo

Para una buena experiencia de desarrollo es fundamental tener herramientas actualizadas que nos apoyen en nuestro ambiente local.

En esta parte distinguimos dos temas principales: dev server y automatización.

### Dev server

[*➕ beneficios/DX/ciclo de feedback*][*➕ beneficios/DX/mantención*]

Un development server es, como su nombre lo dice, un **programa que actúa de servidor en nuestro ambiente de desarrollo local**. El development server se encarga de manejar correctamente todos los distintos tipos de archivos que componen nuestro front –.js, .ts, .jsx, .html, .css, .webp, etc.– y de mandarlos al navegador de una manera optimizada y rápida.

Un buen development server mejora nuestra experiencia de desarrollo a través de mejoras como **Hot Module Replacement** (piensa en lo tedioso que se hace tener que esperar un par de segundos y volver a cargar la página para ver tus cambios) y configuración funcional out-of-the-box.

En Buda nos decidimos por Vite. Para entender mejor qué hace Vite y los beneficios de usarlo recomendamos el artículo 📖 [Why Vite](https://vitejs.dev/guide/why).

### Automatización

Otra manera en que nos gustaría mejorar nuestra experiencia de desarrollo es dejándole a las máquinas cualquier trabajo que puedan hacer mejor que los humanos.

Si delegamos a las máquinas lo que hacen mejor que nosotros, podemos dedicar más energía a hacer lo que nosotros los humanos hacemos mejor que las máquinas, como  entender el contexto en que está inmerso el código y usar nuestra nuestra creatividad para encontrar mejores soluciones.

**Linting**

[➕ *beneficios/DX/ciclo de feedback*][➕ *beneficios/DX/carga cognitiva*][🔖 *relacionado a muchas otra secciones de la guía*]

El ejemplo más claro de automatización creo que son los linters (como `ESLint`) y herramientas relacionadas (como `Prettier` y `Husky`). Nos ayudan con una variedad de temas como estilo de código y estándares del equipo, detección de errores, performance, seguridad, accesibilidad, etc. , de manera instantánea  y disminuyendo en gran parte la carga que implicaría estar pendientes de esas cosas con toda su complejidad . Esto nos hace la vida más fácil tanto al escribir código como también al revisarlo.

Nos gustaría potenciar estos beneficios asegurando que cualquier definición que acordemos para nuestro código que pueda ser linteado, sea efectivamente linteado en vez de vivir sólo en nuestras mentes.

**Generación de código**

Otro manera de automatizar nuestro trabajo es la generación de código. En la sección de type safety [🔖 type safety/definición de la API] explicamos el uso de herramientas como [`Kubb`](https://kubb.dev/) que nos podrían ayudar a autogenerar toda la parte de nuestro código que tiene que ver con la interacción entre el front y el back. 🔜

También los asistentes de IA como `Cursor` son una buena manera de generar código. Un buen ejemplo de cómo aplicarlos es para generar tests. Es un buen caso de uso porque es un contexto muy acotado, basta con pasarle el archivo testeado y un test de ejemplo y con eso tiene prácticamente todo lo que necesita. Estos asistentes no nos libran de tener que entender cómo funciona el código, pero sí en muchos casos pueden ahorrarnos tiempo por la rapidez con que generan el código [➕ *beneficios/equipo/rapidez*].

# Parte 2: Manejo de dependencias

Para esta sección, al hablar de dependencias me refiero básicamente a todos los imports que podemos encontrar en el front, tanto a librerías externas [🔖 temas/librerías] como también a archivos nuestros. Un buen manejo de dependencias implica entender cuáles son las distintas partes de nuestro código y cómo se relacionan, para así poder modificarlas y actualizarlas sin romper todos los lugares en que se usan [➕ *beneficios/DX/mantención*].

### Patrones para compartir código

La app móvil y la aplicación web son en esencia dos versiones distintas de lo mismo: una interfaz para que los usuarios puedan interactuar con nuestra plataforma. La manera de mostrarlo cambia, pero lo que hay detrás es lo mismo. Son las mismas funcionalidades, con la misma información, las mismas acciones, etc…

En teoría. Porque la verdad es que la app siempre se queda atrás en los desarrollos.

Si tuviéramos una manera de compartir código entre la app móvil y la aplicación web sería mucho más fácil lanzar, actualizar y arreglar las features consistentemente en ambas partes [➕ *beneficios/equipo/rapidez*] [➕ *beneficios/UX/experiencia consistente*] [➕ *beneficios/equipo/errores*]. Además podríamos mantener la consistencia de código entre los dos repos para una mejor experiencia de desarrollo [➕ *beneficios/DX/carga cognitiva*].

Para poner un ejemplo: tanto la web y la app muestran el saldo total del usuario, convertido a fiat luego de sumar todo lo que hay en las billeteras y objetivos.

Si bien la interfaz es distinta, el cálculo del número debería ser exactamente el mismo. Lo mismo para los requests que obtienen esa info del backend. Entonces sería muuuy útil tener todo exportado en hooks y funciones compartibles. 

Una manera de hacer esto es mediante un `monorepo`: básicamente un único gran repositorio que contiene nuestros distintos frontends junto con código compartido entre ellos. 

Para avanzar hacia eso el primer paso es empezar a separar todo lo que *sí* podemos compartir entre plataformas (cálculos, interacción con el backend, etc.) de lo que *no* podemos compartir (UI).

Como mencionamos, parte importante de esto es manejar bien las dependencias entre las distintas partes, por lo que también definimos reglas de linter [🔖 herramientas/automatización] que moderan la interacción entre las secciones del código. Por ejemplo, el cálculo del balance no debería depender en nada de aspectos de interfaz, lo que en la práctica se traduce en una regla que prohíbe imports desde la carpeta shared a archivos de interfaz. 

Al momento de implementar el monorepo y efectivamente empezar a compartir código entre plataformas hay herramientas como `pnpm` que permite definir los distintos paquetes que componen el monorepo, y `Nx` que nos ayuda con temas de building, testing, deployment, etc. `Nx` tiene un 📖 [tutorial de monorepos con React](https://nx.dev/getting-started/tutorials/react-monorepo-tutorial) que va muy en línea con cómo lo usaríamos nosotros, ahí mismo se ve por ejemplo que también usan reglas de linter para ordenar las dependencias entre carpetas, además de generación de código para mantener la consistencia de tooling [🔖 herramientas/automatización].

# Parte 3: Estrategias de rendering

¿Qué es Next.js? ¿Qué es Remix?  Si los frameworks como esos se han convertido en algo estándar del mundo de frontend, ¿Por qué nosotros no los usamos? ¿Qué hacen que nosotros no estemos haciendo?

La respuesta resumida es que se encargan de hartas cosas: performance, routing, estrategias de rendering, entre otras.

Por ahora quiero enfocarme en ese último punto, y para entender qué es creo que es útil pensar en lo que se ve en nuestro sitio y separarlo en dos grandes categorías: assets y datos.

Assets son lo que todos ven por igual (HTML, CSS, JS, imágenes, etc.)

<div style="text-align: center;">
  <img src="/frontend-2.png" alt="Assets" style="max-width: 600px;" />
</div>

Mientras que los datos son propios de cada usuario, como los saldos. O propios de un momento en el tiempo, como los precios. Pero en fin son más dinámicos, van cambiando.

<div style="text-align: center;">
  <img src="/frontend-1.png" alt="Datos" style="max-width: 600px;" />
</div>

Al juntarlos el resultado es el contenido que finalmente mostramos:

<div style="text-align: center;">
  <img src="/frontend-3.png" alt="Sitio con assets y datos" style="max-width: 600px;" />
</div>

En las imágenes se ve que en nuestro sitio se carga primero la estructura visual y luego los datos. Por eso al principio mostramos las tarjetas en un estado de carga. Pero no necesariamente tiene que ser así.

Podemos cambiar tanto *el cuándo* en que los cargamos como también el *dónde* los cargamos, en el cliente o en el servidor. Siguiendo el ejemplo y de manera algo simplificada, en vez de cargar los datos en el cliente *después* de los assets, podríamos cargar todo junto en el servidor donde va a ser más rápido. 

Esto tiene consecuencias principalmente en el ciclo de vida de la carga del sitio, como por ejemplo cuánto demora y qué es lo que ve el usuario mientras está cargando. Acá entran en juego conceptos y métricas de preformance como FCP (First Contentful Paint, cuánto se demora en mostrar el contenido inicial), LCP (Largest Contentful Paint, cuánto se demora en mostrar el contenido final), y TTI (Time To Interactive, cuánto se demora en volverse interactivo).

Aplicar estrategias de rendering nos ayuda principalmente a ofrecer al usuario una experiencia más rápida [➕ *beneficios/UX/experiencia clara*] en términos de dichas métricas, pero también nos puede hacer la vida más fácil como desarrolladores simplificando los estados de carga de nuestro sitio [➕ *beneficios/DX/carga cognitiva*].

Para entender en más detalle recomendamos la 📖 Sección *Bouncing Back and Forth* de [este post](https://www.joshwcomeau.com/react/server-components/#bouncing-back-and-forth-2), que explica de manera muy pedagógica las distintas estrategias y cómo afectan las métricas de performance. Luego habla de React Server Components, otra implementación que finalmente apunta a conseguir mejoras en la misma línea.

# Parte 4: Type safety

### Compile time type checking

En este momento nuestra principal herramienta de type safety es `Typescript`. Cuando lo usamos bien nos permite entender más fácilmente qué info viene en las variables [➕ *beneficios/DX/carga cognitiva*] y nos avisa inmediatamente si nos equivocamos [➕ *beneficios/DX/ciclo de feedback*]. Con esto mejoramos el rendimiento del equipo porque podemos trabajar más rápido [➕ *beneficios/equipo/rapidez*] y cometer menos errores [➕ *beneficios/equipo/errores*].

Podemos avanzar hacia un mejor *type checking* mediante tres estrategias principales. En primer lugar, ir gradualmente avanzado hacia ser más estrictos en nuestro uso de TS.

Las siguientes dos propuestas son hacer una **definición formal de la API** y **parsing en runtime**.

### Definición formal de la API

La interfaz entre el frontend y el backend –es decir la API –es un punto crítico de nuestra arquitectura porque siempre estamos interactuando con ella en nuestro desarrollo y por ser una posible fuente de errores en caso de inconsistencias entre ambas partes de esta interfaz.

🔜 [`OpenAPI`](https://www.openapis.org/) es un estándar de especificación de APIs HTTP que nos permite tener una única fuente de la verdad sobre cómo debería estar implementada la interacción entre el backend y el frontend. Un contrato entre ambas partes. Por ejemplo, acá puedes ver un 👀 [esquema de OpenAPI para el endpoint del ticker](https://github.com/budacom/surbtc/blob/monorepo2/frontend/packages/api/schemas/openapi.yaml).

A partir de esta definición podemos:

- Escribir queries en front que calcen con lo que efectivamente se va a recibir, incluyendo el typing correcto de los valores que retornan.
- Escribir tests de backend que verifiquen que la implementación cumple con la especificación.
- Definir los valores de los handlers que usamos en MSW para los tests de front.
- Documentar la API, tanto la pública en [api.buda.com](http://api.buda.com/) [➕ *beneficios/UX/experiencia clara*] como los endpoints de uso interno.
- Configurar las herramientas que nos permiten usar el back sin el front (como Postman) o usar el front sin el back (como MSW en ambiente de desarrollo).

Lo mejor es que hay herramientas como [`Kubb`](https://kubb.dev/) que nos permiten automatizar la generación de código para todos estos puntos [➕ *beneficios/equipo/rapidez*]. Así en teoría cualquier cambio en la API implicaría actualizar la especificación de OpenAPI y luego correr los scripts correspondientes, y así tendríamos gran parte del trabajo ya avanzado. Además de ahorrarnos tiempo nos ayudaría a encontrar posibles inconsistencias en cualquiera de los puntos mencionados [➕ *beneficios/equipo/errores*].

### Parsing

Sumado a lo anterior, todo lo que sale de los archivos de interacción con el backend – es decir de las carpetas `/api` [🔖 temas/patrones]– debe venir correctamente parseado.

Con parsing nos referimos a dos conceptos principales (para entrar más en detalles recomendamos el artículo 📖 [Parse, don’t validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)). Primero, la respuesta del backend debe pasar por una validación, y si no cumple la validación hay que levantar un error [➕ *beneficios/DX/ciclo de feedback*]. No tiene sentido que una variable incorrecta llegue a la interfaz.

En segundo lugar, la respuesta del backend debe quedar correctamente tipada. De esta manera la validación que hicimos nos sirve no sólo en runtime sino que también al momento de desarrollar.

Teniendo estos dos puntos bien implementados, cuando desarrollamos la interfaz podemos confiar en que lo que nos llega de la API está bien y simplemente preocuparnos de… temas de interfaz [➕ *beneficios/DX/carga cognitiva*].

La manera de implementar este parseo es usando [`Zod`](https://zod.dev/), idealmente automatizando la generación de schemas a partir de la especificación de la API, con herramientas como Kubb del punto anterior.
