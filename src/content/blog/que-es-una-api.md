---
title: '¿Qué es una API?'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Oct 18 2021'
heroImage: '/blog-placeholder-3.jpg'
---

Si te ha tocado trabajar con algún computín, seguramente has escuchado por ahí el término "API". API es una sigla en inglés, una abreviación de ***A**pplication **P**rogramming **I**nterface*. Es un término técnico que puede sonar algo complicado (y en algunos casos puede llegar a serlo), pero el concepto básico que hay detrás no tiene por qué ser difícil de entender. 

Vamos a revisar qué es una API usando una analogía sobre cómo navegamos por internet día a día, y usaremos, por supuesto, la API de [Buda.com](http://buda.com/) como ejemplo. Voy a incluir algunos términos técnicos, pero si no eres desarrollador/a no te preocupes, te prometo que los voy a ir explicando todos.

Primero una definición rápida. ¿Qué son las APIs?

<aside>
💡 Las APIs son la manera en que los computadores se comunican entre sí.
</aside>

Y con computador me refiero a cualquier programa, aplicación, bot, página web, servidor, etc. En esencia todos esos son computadores, y para entender cómo se comunican entre sí, vamos a analizar cómo se comunican con los humanos.

### Comunicando humanos con computadores

Imaginemos la siguiente situación. Convencido/a por el potencial de las criptomonedas de ser el futuro del dinero, decides entrar a tu cuenta [Buda.com](http://buda.com/) y comprar bitcoin. El flujo se vería más o menos así:

![Flujo de compra por UI](/que-es-una-api-1.gif)

Podríamos entender todos estos pasos como un intercambio de información entre tú –un humano–, y el sitio web –un computador–. Podríamos incluso imaginarlo como una conversación:

**Tú:** Hola [Buda.com](http://buda.com/), quisiera comprar criptomonedas.

**Buda.com** Buenos días 👋. ¡Excelente! por favor indícame qué criptomoneda quieres comprar, qué medio de pago vas a usar, y cuánto quieres comprar. El monto lo puedes especificar en la misma criptomoneda o en tu moneda local.

**Tú:** Quiero comprar bitcoin usando el saldo de mi billetera, y quiero gastar $30.000.

**Buda.com:** Bien, por tu compra recibirás 0,00062876 BTC. Aquí también puedes ver el precio y la comisión. ¿Quieres confirmar la compra?

**Tú:** Sí, confirmo.

**Buda.com:** Listo, aquí puedes ver tu historial de compras y ventas, que incluye la compra que realizaste recién.

¿Y cómo se realizó toda esta comunicación? Pues no conversando, eso fue solo un ejemplo. Pero si te fijas, el mismo propósito de la conversación podemos conseguirlo mediante distintos elementos gráficos que se muestran en la página. Como usuario, utilizaste entradas desplegables para seleccionar la moneda y el método de pago, una entrada de texto para ingresar el monto, y un botón para confirmar tu intención de compra.

Por otro lado, el sitio web utiliza una combinación de texto, iconos, gráficos, entre otros elementos para indicar el precio, el estado de tu compra, etc.

Si analizamos un poco más en detalle la comunicación entre [Buda.com](http://buda.com/) y el usuario, podemos distinguir dos tipos de interacción. En primer lugar, puedes **obtener información** como el precio actual de bitcoin, el estado de tu último abono, o el saldo en tu billetera. Y por otro lado también puedes **realizar acciones** como comprar bitcoin, actualizar tus datos, o realizar un retiro.

De igual manera podemos distinguir dos categorías principales en cuanto a la información que obtienes del sitio. Ejemplos de **información pública** incluyen el precio actual de una criptomoneda o el valor de las comisiones. Esa información está disponible para que la consulte cualquier usuario porque no es propia de ninguna persona. Pero también hay información que sí corresponde a alguien y por tanto solo esa persona podrá verla. Esto lo consideramos **información privada** y algunos ejemplos incluyen el valor de tu última compra o tu saldo actual.

### Comunicando computadores con computadores

Ahora bien, ¿qué pasa si en vez de comunicar un humano con un computador, quiero comunicar dos computadores entre sí? Como ya mencionamos, esta comunicación se va a realizar por medio de una API, y no mediante los elementos gráficos que tiene la página. Básicamente, en [Buda.com](http://buda.com/) ofrecemos dos principales interfaces, o métodos para conectarte a nuestro sistema:

1. Para usuarios humanos, tenemos una página con una GUI (interfaz gráfica de usuario, del inglés *graphical user interface*). Aquí la interacción se hace mediante elementos gráficos como los que mencionamos antes.
2. Para usuarios robots, tenemos una API (interfaz de programación de aplicación, del inglés *application programming interface*). Aquí la interacción se hace mediante *requests*.

¿Mediante qué?

*Requests*, que en inglés significa solicitudes, aunque también hablamos de llamadas. Es un término importante así que lo quiero destacar:

<aside>
💡 Un <b>request</b> se refiere a una sola interacción entre dos computadores mediante una API.

</aside>

Esto es análogo a las acciones que puedes realizar tú como usuario en la página, tales como enviar un formulario, realizar una venta de criptomonedas, o revisar el saldo en tu billetera.

¿En qué consiste un *request*? De manera un poco simplificada, la estructura básica de un *request* incluye una acción y un recurso. Para entenderlo mejor, veamos un ejemplo de cómo sería una llamada para ver tu historial de órdenes (compras y ventas). El request se vería más o menos así:

```json
GET www.buda.com/api/orders
```

La primera parte es la acción, que especifica si solo queremos obtener información, o si queremos realizar un cambio permanente (la misma distinción que hicimos antes). En este caso usamos GET, que indica que solo queremos obtener información. La segunda parte es el recurso, que indica la dirección de la API y a qué parte específica queremos acceder. En este caso queremos acceder al historial de órdenes, *orders*.

Al realizar la llamada vamos a obtener una respuesta por parte de la API. Nuevamente simplificando un poco, esta respuesta podría verse algo así:

```json
{
	"orders": [
		{
			"id": 2,
			"type": "bid",
			"amount": ["0.001", "BTC"],
			"date": "2021-10-05",
			"state": "pending",
		},
		{
			"id": 1,
			"type": "ask",
			"amount": ["0.02", "ETH"],
			"date": "2021-09-22",
			"state": "confirmed",
		}
	]
}

```

No es una respuesta muy bonita, pero eso es porque no está diseñada para humanos, sino para computadores. Y a los computadores les importa que esté bien estructurada y que el formato de la respuesta sea conocido.

En caso de que queramos realizar una operación en vez de simplemente obtener información, vamos a tener que hacer una llamada tipo POST e incluir algunos datos adicionales. Por ejemplo, digamos que ahora queremos comprar un poco de bitcoin. El mismo ejemplo que vimos en la página, ahora queremos hacerlo por API. En ese caso nuestro *request* sería más o menos así:

```json
POST www.buda.com/api/orders
{
	"type": "bid",
	"amount": ["0.003", "BTC"],
}

```

Aquí se indica que la orden que queremos ingresar va a ser una compra, además del monto que queremos comprar. La respuesta de la API nos diría si todo salió bien o si hubo un error al intentar ingresar la compra (como por ejemplo que no tienes saldo suficiente), además de indicarnos toda la información correspondiente a la comprar que acabamos de ingresar. Podría ser algo así:

```json
201 CREATED
{
	"id": 2,
	"type": "bid",
	"amount": ["0.001", "BTC"],
	"date": "2021-10-05",
	"state": "pending",
}

```

Como dato adicional, el número 201 en la respuesta es un código HTTP que nos indica que todo salió bien. Otro código HTTP que seguramente has visto es el 404, que indica que no se pudo encontrar una página.

### La API de [Buda.com](http://buda.com/)

Pero ya basta de ejemplos simplificados, pasemos a ver una API de verdad. Y cuál mejor que la de [Buda.com](http://buda.com/). Si sabes programar y conoces nuestra API, podrías hacer tu propio *bot* que haga trading, o que te avise cuando el precio de bitcoin bajó (para comprar más, obvio), o que sincronice la información de tu cuenta [Buda.com](http://buda.com/) con tu planilla de finanzas personales. En el mundo de la programación, el límite es tu imaginación (casi). Si no sabes programar, de todas maneras esto te va a ayudar a profundizar tu conocimiento.

La forma más fácil de probar la API de [Buda.com](http://buda.com/) es ir al siguiente link en tu navegador:

https://www.buda.com/api/v2/markets/BTC-CLP/ticker

Y así de fácil, ya puedes decir que has usado la API de [Buda.com](http://buda.com/). La llamada que hicimos recién nos muestra el estado actual del mercado BTC-CLP, lo cual llamamos *ticker*. Como puedes ver, y como era de esperar, lo que te muestra el navegador no es muy bonito. Pero incluye información útil y actualizada como el precio actual, el volumen transado, y la variación de precio como porcentaje de las últimas 24 horas, entre otras cosas.

Si te interesa saber más, puedes revisar la documentación de la API acá: [https://api.buda.com](https://api.buda.com/). Aquí explicamos en detalle las llamadas que es posible hacer, qué información es necesario especificar, y qué información se incluirá en la respuesta, para que puedas comunicarte con nuestro sitio como un verdadero robot 🤖. El documento está hecho para desarrolladores, así que puede que algunas cosas puedan ser algo complicadas de entender, pero con lo que aprendiste hoy por lo menos tienes el primer paso resuelto.
