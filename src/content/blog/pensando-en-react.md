---
title: 'Pensando en React'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Jul 14 2023'
heroImage: '/blog-placeholder-5.jpg'
---

En este artículo repaso la guía 🔗 [Guía Thinking In React](https://react.dev/learn/thinking-in-react), aplicándola a la vista simple de Buda.com. Acá aprenderás como usar correctamente el hook `useState` para guardar el estado de una aplicación.

A partir de esto, la guía indica cómo construir una aplicación en React en 5 pasos:
1. Separar la interfaz en una jerarquía de componentes
2. Construir una versión estática
3. Encontrar la representación completa más simple posible del estado
4. Identificar dónde vive el estado
5. Agregar flujo inverso de datos

Voy a usar como ejemplo una vista en el flujo de compra en Buda.com:

<div style="text-align: center;">
  <img src="/bug-1.gif" alt="Flujo completo" style="max-width: 500px;" />
</div>

## 1. Separar la interfaz en una jerarquía de componentes

Digamos que esta es la vista que queremos construir:

<div style="text-align: center;">
  <img src="/react-1.png" alt="" style="max-width: 600px;" />
</div>

La podemos dividir en componentes así:

<div style="text-align: center;">
  <img src="/react-2.png" alt="" style="max-width: 600px;" />
</div>

De esta manera construimos una jerarquía de componentes:

<div style="text-align: center;">
  <img src="/react-3.png" alt="" style="max-width: 600px;" />
</div>

\* Podemos notar que la capa de abajo también son componentes, pero no propios de esta feature sino que son los compartidos del design system.

## 2. Construir una versión estática

El siguiente paso es construir una versión hardcodeada de la aplicación. Es decir, sin guardar estado.

Esto significa que no tendremos ni llamadas al backend (por lo que debemos usar información falsa) ni interacción con el usuario.

En este caso esta primera versión se vería simplemente así:

<div style="text-align: center;">
  <img src="/react-1.png" alt="" style="max-width: 600px;" />
</div>


## 3. Encontrar la representación completa más simple del estado

En esta etapa sí agregamos estado.

Pero primero, partamos con una definición básica: ¿Qué es el estado de una aplicación?
_Estado_ es información que tiene que se debe guardar y que puede ir cambiando. En la práctica, estos estados se guardan en hooks como `useState` y sus cambios gatillan re-renderizaciones.

En el artículo 🔗 [Application State Management with React
](https://kentcdodds.com/blog/application-state-management-with-react), Kent C. Dodds dice:

> Every type of state is either server cache or UI state.

Esto tiene sentido si pensamos que el frontend es como una capa entre el backend y el usuario, y el estado que debe guardar es producto de la interacción con esas dos partes.

Para datos del servidor podemos usar una librería como React Query, con lo cual no necesitamos preocuparnos de definir directamente esos estados.

De lo que sí tenemos que preocuparnos es de los datos propios de la interacción con el usuario. A modo general, cada input del usuario se guarda como un estado.

En el ejemplo de la vista que queremos construir, tendríamos que guardar los siguientes datos:

1. Paso actual (ej: paso 2)
2. Moneda base (ej: BTC)
3. Moneda de cotización (ej: CLP)
4. Monto (ej: $10.000)
5. Origen (ej: Billetera CLP)
6. Destino (ej: Billetera BTC)
7. Tipo de movimiento (ej: compra)

Estos estados son los principales a nivel global, pero cada subcomponente también va a tener sus propios estados.

---

Ahora bien, un par de cosas más sobre el estado. La 🔗 [Guía Thinking In React](https://react.dev/learn/thinking-in-react) nos un par de indicadores adicionales sobre cuándo usar estados. El primero:

> Is it passed in from a parent via props? If so, it isn’t state.

Por ejemplo, el resumen lateral `SummarySidebar` necesita saber qué moneda mostrar:

<div style="text-align: center;">
  <img src="/react-6.png" alt="" style="max-width: 600px;" />
</div>

En principio podríamos pensar un usar `useEffect`. Pero esa información viene del componente padre `Transactions`, entonces no es necesario definirla como estado en `SummarySidebar`.
Si hay un cambio de estado en el componente padre, toda la jerarquía hacia abajo se actualizará.

> Can you compute it based on existing state or props? If so, it definitely isn’t state.

Por ejemplo, podríamos agregar _moneda de origen_, para calcular el monto máximo que puede ingresar el usuario:

<div style="text-align: center;">
  <img src="/react-8.png" alt="" style="max-width: 600px;" />
</div>

Pero moneda de origen no es necesario porque puedo calcularla en base a estados ya existentes:
- Para compras, la moneda de origen es la moneda de cotización.
- Para ventas, la moneda de origen es la moneda base.

> The most important principle for structuring state is to keep it DRY (Don’t Repeat Yourself).

Esto es muy parecido al concepto de evitar redundancia en BDD. La diferencia es que en BDD tenemos `constraints` que nos permiten agregar redundancia y a la vez evitar inconsistencia. En React no tenemos constraints, entonces tenemos que tener mucho más ojo con la redundancia.

En otras palabras, mantener estado redundante nos obliga a mantenerlo sincronizado. Esto hace nuestros componentes más complejos, aumenta el riesgo de errores, y empeora la experiencia de usuario. Puedes leer más en el artículo 🔗 [Don't Sync State, Derive It](https://kentcdodds.com/blog/dont-sync-state-derive-it).

## 4. Identificar dónde vive el estado.

Una vez que identificamos _cuáles_ son los datos que vamos a guardar como estado, el siguiente paso es determinar _dónde_ viven.

Los estados principales que mencionamos antes viven en el componente padre. Pero otros solo necesitan vivir en su propio componente hijo. Por ejemplo, el saldo solo lo usan un par de pasos en el flujo, entonces vive en esos pasos:

<div style="text-align: center;">
  <img src="/react-5.png" alt="" style="max-width: 600px;" />
</div>

La idea acá es dejar el estado lo más abajo posible, porque tener los datos y el código más cercanos a donde se usan hace que los componentes sean más fáciles de mantener.

## 5. Agregar flujo inverso de datos

El último paso

Los componentes tienen estado que pasan como props a sus hijos. Pero a veces los hijos necesitan modificar estado de los padres. Es el caso del componente `SelectCurrencyStep`, en que una vez que el usuario elige la moneda, lo debe guardar a nivel global (en el componente `Transactions`) para que lo use el resto del flujo.

En este caso `Transactions` puede pasar como prop a `SelectCurrencyStep` el setter del estado `setBaseCurrency`.