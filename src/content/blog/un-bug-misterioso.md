---
title: 'Un bug misterioso'
description: ''
pubDate: 'Jan 24 2024'
heroImage: '/blog-placeholder-4.jpg'
---

En Buda.com tenemos una interfaz donde puedes comprar y vender criptomonedas de manera fácil y rápida. Solo tienes que seguir algunos pasos donde eliges la moneda y la cantidad que quieres comprar. Se ve así:

<div style="text-align: center;">
  <img src="/bug-1.gif" alt="Flujo completo" style="max-width: 500px;" />
</div>


Antes de confirmar tu compra, te mostramos un resumen con el detalle: cuánto vas a pagar, cuánto es de comisión y cuánto de la criptomoneda vas a recibir según el precio actual:

<div style="text-align: center;">
  <img src="/bug-2.png" alt="Esperado" style="max-width: 500px;" />
</div>

También te reservamos el precio durante un minuto. Esto significa que si el precio de la criptomoneda cambia durante ese minuto, de todas maneras vas a pagar y recibir la cantidad que te indicamos. Cualquier diferencia la cubrimos nosotros. Para representar visualmente el paso del tiempo, hay un temporizador en forma de barra horizontal que se va vaciando.

Una vez que pasa el minuto, la página actualiza el precio, resetea el temporizador, y muestra un nuevo resumen. 

Un día, un usuario nos escribió por el chat de soporte para decirnos que la vista del resumen no le funcionaba. Se veía así:

<div style="text-align: center;">
  <img src="/bug-3.gif" alt="Bug" style="max-width: 500px;" />
</div>

Al principio no nos quedó claro por qué ocurría este problema. Intentamos replicarlo en nuestro ambiente de pruebas, pero no pudimos. Le pedimos al resto del equipo que revisara si tenían el mismo problema, pero a nadie más le pasaba. Incluso nos juntamos por videollamada con el usuario para obtener más información.

Descubrimos que:
-  Todas las llamadas y respuestas de nuestro backend funcionaban correctamente, por lo que el problema estaba relacionado con la implementación del componente visual en el frontend.
- El mismo usuario **no** tenía problemas al hacer la compra desde la app móvil, así que no parecía ser algo específico de su cuenta.
- Curiosamente, si intentaba hacer la compra desde el sitio web pero usando su teléfono, tampoco tenía el problema.

Esto nos dio una pista: el problema tenía algo que ver con el dispositivo actual.

Luego de varias pruebas e hipótesis nos dimos cuenta de que el error estaba en la forma en que calculábamos el tiempo para la barra horizontal. Este era el código:

```javascript
const currentTime = new Date();
// transaction.expiresAt es una fecha en formato ISO 8601,
// por ejemplo: 2024-01-24T15:00:00.000Z
const expirationTime = new Date(transaction.expiresAt);

const timerBarTime = expirationTime.getTime() - currentTime.getTime();

timerBar.start(timerBarTime);
```

¿Se te ocurre qué puede ser?

El problema es en la asignación de la variable `currentTime`. `new Date()` obtiene la *fecha y hora actual del dispositivo*. ¿Pero qué pasa si el dispositivo tiene una hora incorrecta?

Esto puede ocurrir al configurar la hora manualmente en vez de sincronizarla automáticamente. En los casos donde el dispositivo tiene una hora adelantada, el valor de `currentTime` queda "en el futuro", por lo que `timerBarTime` termina siendo negativo y el temporizador se reinicia apenas comienza.

La solución inmediata para el usuario fue corregir la hora de su dispositivo, y después implementamos una solución más robusta en que el tiempo del temporizador no dependía de la hora del dispositivo.

