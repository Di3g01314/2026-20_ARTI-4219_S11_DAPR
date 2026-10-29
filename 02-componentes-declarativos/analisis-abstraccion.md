# analisis-abstraccion.md
# Completa este archivo con tus respuestas a los TODOs de la Fase 2.

## TODO 2: Análisis de la Abstracción

### 1. **Migración de broker:** Si la empresa decide migrar de Redis a Apache Kafka mañana, ¿qué archivos del repositorio habría que modificar? ¿Cambiaría el código Python del subscriber o del publisher?

Cambiaríamos el archivo `02-componentes-declarativos/pubsub-redis.yaml`, porque ahí está configurado Redis. Habría que ajustarlo para conectar Dapr con Kafka.

El código Python del publisher y del subscriber podría mantenerse igual, conservando el nombre `order-pubsub` y el tópico `orders`. Al revisar el código vimos que las aplicaciones se comunican con Dapr y no tienen una conexión directa con Redis.

### 2. **Inventario de brokers soportados:** Consulta la [documentación de componentes Pub/Sub de Dapr](https://docs.dapr.io/reference/components-reference/supported-pubsub/). Lista **5 brokers distintos** que Dapr soporta como implementación. ¿Qué campo en el YAML cambia para apuntar a cada uno?

Al consultar la documentación encontramos estas cinco opciones:

| Broker | `spec.type` en el YAML |
|---|---|
| [Redis Streams](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-redis-pubsub/) | `pubsub.redis` |
| [Apache Kafka](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-apache-kafka/) | `pubsub.kafka` |
| [RabbitMQ](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-rabbitmq/) | `pubsub.rabbitmq` |
| [Google Cloud Pub/Sub](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-gcp-pubsub/) | `pubsub.gcp.pubsub` |
| [Azure Service Bus Topics](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-azure-servicebus-topics/) | `pubsub.azure.servicebus.topics` |

El campo que cambia para elegir el broker es `spec.type`. También debemos ajustar los datos de conexión en `spec.metadata`, porque cada broker necesita su propia configuración.

### 3. **Secretos en producción:** El valor `redisPassword: ""` funciona para el lab, pero en producción nunca se deben poner contraseñas en texto plano en un YAML de Git. ¿Qué mecanismo ofrece Dapr para referenciar secretos de forma segura? (Pista: busca "Dapr Secret Store component reference")

Dapr permite usar un almacén de secretos, llamado **Secret Store**. Allí se guarda la contraseña y en el YAML usamos `secretKeyRef` para indicar dónde encontrarla, sin escribir su valor directamente.

Por ejemplo, podemos guardar la contraseña en un Secret de Kubernetes y hacer que Dapr la consulte cuando la necesite. Así evitamos subir la contraseña al repositorio.
