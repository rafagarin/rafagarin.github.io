---
title: 'Guía de estados distribuídos'
description: 'Esta guía contiene un breve resumen de mis aprendizajes trabajando con estados distribuídos en múltiples servicios'
pubDate: 'Oct 7 2023'
heroImage: '/blog-placeholder-2.jpg'
---

Esta guía contiene un breve resumen de mis aprendizajes trabajando con estados distribuídos en múltiples servicios

### Contexto

Digamos que un *Servicio Principal* debe realizar dos acciones:

1. Generar un side effect en un *Servicio Externo.*
2. Actualizar su base de datos.

Dado que no es posible asegurar disponibilidad del servicio externo, queremos asegurar:

1. Atomicidad: se ejecutan ambos o ninguno.
2. Reintentos idempotentes.

### Versión ingenua

```ruby
# no se asegura atomicidad
# no se reintenta, porque no sería idempotente
ExternalMovement.create!
local_movement.mark_as_confirmed!
```

### Versión mejorada

Guardamos un UUID en el servicio externo para asegurar idempotencia del side effect. Luego en el servicio principal:

```ruby
ActiveRecord::Base.transaction do
  PerformExternalMovementJob.perform_later(local_movement)
  local_movement.mark_as_confirming!
end
```

```ruby
class PerformExternalMovementJob
  def perform(local_movement)
	  # operación idempotente gracias al UUID
    external_movement = ExternalMovement.create! uuid: local_movement.transfer_uuid

		# finalmente realizamos un "cierre" en la bdd local
    local_movement.mark_as_confirmed! if external_movement.confirmed?
  end
end

```

Comentarios adicionales:
- Agregamos un estado intermedio `confirming` para dar visibilidad de operaciones con posibles problemas. Si usamos [Delayed Job](https://github.com/collectiveidea/delayed_job), la transacción asegura atomicidad de la primera etapa (`mark_as_confirming`).
- Si fuera necesario realizar múltiples side effects, simplemente usamos múltiples UUIDs. Así se mantiene la idempotencia de reintentos, y se mantiene la atomicidad tarde o temprano (*eventual consistency*) siempre que se siga reintentando. Alternativamente, en caso de múltiples fallos en uno de los side effects, podemos asegurarnos de permitir la revocación de los demás side effects y de las actualizaciones a la base de datos local.
- Si la confirmación del side effect fuera asíncrona, simplemente encolamos el job nuevamente hasta que se confirme el side effect. En ejecuciones posteriores la línea con el side effect no va a hacer nada, pero la línea con la actualización local sí tendrá efecto en caso que corresponda. Nuevamente, se mantiene la idempotencia de reintentos, y se mantiene la atomicidad tarde o temprano (*eventual consistency*) siempre que se siga reintentando.
- Si el side effect en el servicio externo no asegura idempotencia, debemos –antes de ejecutarlo– revisar si no se ejecutó antes. Por ejemplo, con `ExternalMovement.find_by(uuid:).present?`
- Parte de asegurar atomicidad es no ejecutar side effects dentro de transacciones, tal como propone la gema [isolator](https://github.com/palkan/isolator). Esta solución cumple con eso.