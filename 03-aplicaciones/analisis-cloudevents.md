# analisis-cloudevents.md
# Completa este archivo con tus respuestas al TODO 1 de la Fase 3.

## TODO 1: Análisis de trazabilidad (CloudEvents)

### 1. **`id`**: ¿Quién genera este UUID? ¿Por qué es importante para la idempotencia?

En nuestro laboratorio, el `id` lo genera automáticamente el sidecar de Dapr. El publisher solo envía `orderId` y `status`, pero en los logs del subscriber vimos que cada evento recibido también tenía un `id` diferente.

Ese identificador permite reconocer un mismo evento si vuelve a llegar. La aplicación podría comprobar si ya lo procesó y evitar repetir la misma acción; eso ayuda a lograr la idempotencia.

### 2. **`source`**: ¿Qué valor tiene? ¿De dónde lo toma Dapr?

En los logs del subscriber vimos que `source` tenía el valor `order-publisher`. Ese valor coincide con el identificador configurado en `publisher-job.yaml` mediante `dapr.io/app-id: "order-publisher"`.

### 3. **`traceparent`**: ¿Qué estándar implementa este atributo? ¿Cómo se relaciona con OpenTelemetry?

Al revisar la documentación entendimos que `traceparent` usa el estándar W3C Trace Context. Sirve para seguir una misma operación mientras pasa por distintos servicios. OpenTelemetry usa esa información para relacionar los pasos y mostrar el recorrido de la operación.

En nuestro laboratorio vimos que `traceparent` aparecía vacío en los logs del subscriber, así que no pudimos ver ese seguimiento en la prueba que hicimos.

### 4. **Valor de negocio**: Un equipo de auditoría necesita rastrear todos los eventos que pasaron por el sistema en las últimas 24h. ¿Qué atributos del sobre CloudEvents les serían más útiles y por qué?

Nos servirían el `id` para distinguir cada evento y el `source` para identificar de qué aplicación vino. En nuestra prueba vimos un identificador diferente para cada evento y todos tenían como origen `order-publisher`. También vimos en `data` el número de la orden y su estado, lo que nos permite relacionar el evento con lo que estaba pasando con esa orden.

Para revisar las últimas 24 horas también necesitaríamos la fecha y la hora. En nuestra salida aparecían en los registros del subscriber, pero esos registros no mostraban el atributo `time` del sobre CloudEvents.

El `traceparent` podría ayudar a seguir el recorrido entre servicios, según lo que consultamos, pero en nuestra prueba apareció vacío y no nos aportó esa información.
