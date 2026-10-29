# reflexion-arquitectonica.md
# Completa este archivo con tus respuestas al Reto 3 de la Fase 4.

## Reto 3: Reflexión Arquitectónica

### 1. **Comparación de enfoques:** En una arquitectura sin Dapr usando el SDK de Kafka directamente, ¿dónde estaría el código que gestiona el TTL? ¿Y el consumer group? ¿Quién sería responsable de mantenerlo (el equipo de la app o el equipo de plataforma)?

Sin Dapr, configuraríamos el grupo de consumidores en la aplicación que lee los mensajes de Kafka. Para evitar procesar órdenes vencidas, podríamos agregar una revisión de la antigüedad del mensaje antes de procesarlo.

Kafka también permite configurar cuánto tiempo conserva los mensajes, pero eso no equivale a comprobar si una orden todavía es válida para el negocio.

Repartiríamos la responsabilidad: el equipo de la aplicación mantendría la revisión de vencimiento y la configuración del consumidor. El equipo de plataforma mantendría Kafka y sus reglas de almacenamiento.

### 2. **El patrón sidecar como frontera de responsabilidad:** La decisión de usar `ttlInSeconds=120` es una decisión de negocio (las órdenes expiran en 2 minutos) pero se implementa en la capa de infraestructura (el Component YAML). ¿Cómo distribuirías esta responsabilidad entre el equipo de producto (que conoce el SLA de expiración) y el equipo de plataforma (que gestiona el broker)?

El equipo de producto debería definir cuánto tiempo sigue siendo válida una orden y explicar por qué debe vencer después de dos minutos. El equipo de plataforma se encargaría de revisar cómo configurar ese tiempo con las herramientas que se están usando.

Después, ambos deberían comprobar que el resultado cumple lo que necesita el negocio. En nuestro laboratorio configuramos `streamTTL` en `120s`, pero vimos que llegaron mensajes después de ese tiempo. Esa prueba nos mostró la importancia de revisar lo que ocurre realmente, además de guardar la configuración.

### 3. **Limitaciones del patrón:** Dapr es agnóstico al broker, pero ¿significa eso que puedes migrar de Redis a Kafka sin *ningún* cambio? Investiga si hay diferencias entre los brokers en cuanto a soporte de TTL y consumer groups, y documenta al menos una limitación concreta de la abstracción.

Dapr facilita cambiar de broker, pero todavía tenemos que ajustar la conexión y revisar las opciones que admite cada uno.

Por ejemplo, la documentación de Redis describe `streamTTL` como un tiempo aproximado de conservación de mensajes. Kafka tiene una configuración de retención por tópico llamada `retention.ms`. Además, Dapr permite indicar un TTL por mensaje al publicarlo, que es diferente de esas configuraciones de almacenamiento.

Con los grupos también hay diferencias: en Redis usamos `consumerID`, mientras que el componente de Kafka permite definir `consumerGroup`, que tiene prioridad sobre `consumerID`.

Esto muestra una limitación: usar Dapr no hace que todas las configuraciones y comportamientos sean iguales entre brokers. En nuestra prueba con Redis vimos mensajes recibidos después de los 120 segundos. Las diferencias con Kafka las consultamos en la documentación, porque no hicimos esa migración.

## Observación de la prueba de TTL

Configuramos `streamTTL` en `120s` y ejecutamos la prueba de la guía. Al encender nuevamente el subscriber, vimos que recibió mensajes que llevaban más de 120 segundos desde su publicación. Por eso, en nuestra prueba no observamos la expiración esperada.
