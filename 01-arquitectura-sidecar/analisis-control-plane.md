# analisis-control-plane.md
# Completa este archivo con tus respuestas a los TODOs de la Fase 1.

## TODO 1: Análisis del Plano de Control

### 1. ¿Qué tipo de objeto de Kubernetes es el `dapr-sidecar-injector`? ¿Por qué necesita ser un **MutatingWebhookConfiguration**?

El dapr-sidecar-injector se despliega mediante un Deployment y se ejecuta dentro de un Pod. Su trabajo es agregar automáticamente el contenedor de Dapr, llamado daprd, junto al contenedor de nuestra aplicación.

Para que K8s sepa cuándo llamar al injector, se utiliza una MutatingWebhookConfiguration. Esta configuración permite que Kubernetes consulte al injector antes de terminar de crear un Pod. Si el Pod tiene la anotación dapr.io/enabled: "true", el injector agrega el sidecar.

Se llama Mutating porque permite modificar la definición del Pod. Así podemos tener el contenedor de Dapr sin agregarlo manualmente al manifiesto de nuestra aplicación.

### 2. ¿Cuántas réplicas del `dapr-operator` están corriendo? ¿Por qué el plano de control de Dapr es más liviano que el de Istio?

En nuestro clúster está corriendo una réplica del dapr-operator, porque en la salida del comando aparece un solo Pod con ese nombre. Además, está en estado Running y no tiene reinicios.

La idea de que Dapr tiene un plano de control más liviano se relaciona con las tareas que realiza. Dapr ofrece funciones como publicar y recibir eventos, que la aplicación utiliza cuando las necesita. Istio administra la comunicación de una malla de servicios, incluyendo reglas para dirigir el tráfico, seguridad y configuración de los proxies.

Dapr permite trabajar con una configuración más sencilla.

### 3. Describe con tus palabras el flujo de inyección del sidecar: ¿qué ocurre desde que `kubectl apply` un Deployment con `dapr.io/enabled: "true"` hasta que el sidecar está corriendo?

Cuando ejecutamos kubectl apply, enviamos a Kubernetes el Deployment con la configuración de nuestra aplicación. La anotación dapr.io/enabled: "true" debe estar en la plantilla de los Pods.

Cuando Kubernetes va a crear uno de esos Pods, consulta al injector mediante el webhook. El injector reconoce la anotación y agrega a la definición del Pod el contenedor daprd, junto con la configuración que necesita.

Después, Kubernetes asigna el Pod a un nodo, descarga las imágenes e inicia los contenedores. Así quedan funcionando nuestra aplicación y el sidecar de Dapr dentro del mismo Pod. Ambos pueden comunicarse mediante localhost, y la aplicación puede usar las funciones de Dapr.

---

## TODO 2: Exploración de los CRDs de Dapr

### 4. ¿Qué CRDs instala Dapr? Lista al menos 4 y describe brevemente para qué sirve cada uno.

Los CRDs permiten agregar nuevos tipos de recursos a Kubernetes. Cuatro de ellos son:

- components.dapr.io: permite configurar las tecnologías que utilizará Dapr, como Redis para mensajería o una base de datos para guardar estado.
- configurations.dapr.io: permite configurar el comportamiento de Dapr, por ejemplo, las métricas, las trazas y las opciones de seguridad.
- subscriptions.dapr.io: permite declarar suscripciones a tópicos e indicar a qué ruta de la aplicación deben llegar los mensajes.
- resiliencies.dapr.io: permite definir cómo actuar ante fallos, por ejemplo, cuántas veces reintentar una operación o cuánto tiempo esperar una respuesta.

### 5. ¿Cómo se relaciona el CRD `components.dapr.io` con el concepto de "Building Block"? ¿Es el componente la abstracción o la implementación concreta?

Un Building Block es una capacidad que Dapr ofrece a las aplicaciones, como publicar y recibir mensajes mediante Pub/Sub. Representa la abstracción, porque la aplicación utiliza la misma API sin tener que conocer los detalles del broker.

Un componente configura la tecnología concreta que permite realizar esa función. Por ejemplo, para usar el Building Block de Pub/Sub podemos configurar un componente de tipo pubsub.redis, que conecta Dapr con Redis.

El CRD components.dapr.io permite que Kubernetes reconozca y gestione esos recursos de tipo Component. Por eso, el Building Block representa la capacidad general y el componente selecciona y configura su implementación concreta. Documentación de Dapr.
