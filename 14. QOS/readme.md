La QoS es una tecnología de infraestructura de red que utiliza un conjunto de herramientas y mecanismos para asignar diferentes niveles de prioridad a los distintos flujos de tráfico IP y proporciona un tratamiento preferencial a los flujos de tráfico IP de mayor prioridad. Para estos flujos, reduce la pérdida de paquetes durante la congestión de red y ayuda a controlar la latencia y la variación de latencia (jitter). Para los flujos de tráfico IP de baja prioridad, ofrece un servicio de entrega con el mejor esfuerzo posible. Esto es similar al funcionamiento de un carril preferencial para vehículos con alta ocupación: un carril especial de alta prioridad está reservado para vehículos con varios pasajeros (tráfico de alta prioridad), lo que permite que estos vehículos circulen sin problemas, evitando la congestión del tráfico en los carriles generales.

Los principales objetivos de implementar la QoS en una red son:
- Acelerar la entrega de aplicaciones en tiempo real
- Garantizar la continuidad operativa de las aplicaciones críticas para el negocio
- Garantizar la equidad para las aplicaciones no críticas en caso de congestión
- Establecer un límite de confianza en el borde de la red para aceptar o rechazar las marcas de tráfico introducidas por los dispositivos finales

La QoS utiliza los siguientes mecanismos para lograr estos objetivos:
- Clasificación y marking.  
- Control y modelado de tráfico
- Gestión y prevención de la congestión

Todos estos mecanismos de QoS se describen en este capítulo.

# La necesidad de QoS: 

Las aplicaciones multimedia en tiempo real modernas, como la telefonía IP, la telepresencia, la transmisión de vídeo, Cisco Webex y la videovigilancia IP, son muy sensibles a las demoras en la transmisión y generan exigencias específicas de _quality of service_ (QoS) en la red. Cuando los paquetes se envían mediante un modelo de transmisión por mejor esfuerzo, pueden no llegar en orden ni a tiempo, e incluso pueden perderse. En el caso del vídeo, esto puede provocar pixelización, congelación de la imagen, reproducción discontinua, desincronización de audio y vídeo, o incluso la ausencia total de imagen. En el caso del audio, puede causar eco, superposición de voces (efecto de walkie-talkie, donde solo una persona puede hablar a la vez), voz ininteligible o distorsionada, interrupciones en la señal de voz, largos silencios y desconexiones. Las principales causas de estos problemas de calidad son:
- Falta de ancho de banda
- Latencia y Jitter de latencia
- Pérdida de paquetes

## Falta de ancho de banda:

El ancho de banda disponible en el camino de datos desde el origen al destino es igual a la capacidad del enlace con menor ancho de banda. Cuando se supera la capacidad máxima de este enlace, se produce congestión, lo que genera pérdida de tráfico. La solución más evidente a este problema es aumentar el ancho de banda del enlace, pero esto no siempre es posible debido a limitaciones presupuestarias o tecnológicas. Otra opción es implementar mecanismos de calidad de servicio (QoS), como el control de tráfico y la gestión de colas, para priorizar el tráfico según su importancia. El tráfico de voz, vídeo y las aplicaciones críticas para el negocio deben tener prioridad y un ancho de banda suficiente para satisfacer sus requisitos, mientras que el tráfico menos importante utilizará el ancho de banda restante.

## Latencia y Jitter

La _latencia de red_, también conocida como retardo de extremo a extremo, es el tiempo que tardan los paquetes en recorrer la red desde el origen hasta el destino. La Recomendación G.114 de la UIT establece que, independientemente del tipo de aplicación, la latencia de red no debería superar los 400 ms, y para el tráfico en tiempo real, debería ser inferior a 150 ms. Sin embargo, la UIT y Cisco han demostrado que la calidad del tráfico en tiempo real no se ve afectada significativamente hasta que la latencia de red supera los 200 ms. Para poder implementar estas recomendaciones, es importante comprender las causas de la latencia de red. Esta latencia puede dividirse en latencia fija y latencia variable:

- Retardo de propagación (fijo)
- Retardo de serialización (fijo)
- Retardo de procesamiento (fijo)
- Variación del retardo (variable)

### Retraso de propagación

El retraso de propagación es el tiempo que tarda un paquete en viajar desde el origen al destino a la velocidad de la luz a través de un medio como cables de fibra óptica o cables de cobre. La velocidad de la luz es de 299.792.458 metros por segundo en el vacío. La ausencia de vacío en un cable de fibra óptica o de cobre reduce la velocidad de la luz en función de un valor conocido como índice de refracción; cuanto mayor sea este índice, más lenta será la propagación de la luz.
La velocidad media de la luz en un cable de fibra óptica con un índice de refracción de 1,5 es de aproximadamente 200.000.000 metros por segundo (es decir, 300.000.000 / 1,5). Si un cable de fibra óptica con un índice de refracción de 1,5 se extendiera a lo largo de la circunferencia ecuatorial de la Tierra (unos 40.075 km), el retraso de propagación sería de aproximadamente 200 ms, un valor aceptable incluso para el tráfico en tiempo real.
Cabe recordar que los cables de fibra óptica no siempre se instalan físicamente siguiendo la ruta más corta entre dos puntos. Pueden ser cientos o incluso miles de kilómetros más largos de lo previsto. Además, otros componentes necesarios en los cables de fibra óptica, como repetidores y amplificadores, pueden introducir retrasos adicionales. El service-level agreement (SLA) del proveedor puede consultarse para estimar y planificar la latencia mínima, máxima y media del circuito.

> [!NOTE]
> A veces es necesario utilizar la comunicación por satélite para llegar a zonas de difícil acceso. El retardo de propagación en los circuitos de satélite es el tiempo que tarda una señal de radio, viajando a la velocidad de la luz, desde la superficie terrestre hasta un satélite (lo que puede implicar varios saltos entre satélites) y de vuelta a la superficie terrestre; dependiendo del número de saltos, este retardo puede superar los 400 ms recomendados. En estos casos, no hay forma de reducir el retardo, salvo buscando un proveedor de servicios por satélite que ofrezca menores tiempos de propagación.

### Serialization Delay
Serialization delay is the time it takes to place all the bits of a packet onto a link. It is a
fixed value that depends on the link speed; the higher the link speed, the lower the delay.
The serialization delay s is equal to the packet size in bits divided by the line speed in bits
per second. For example, the serialization delay for a 1500-byte packet over a 1 Gbps
interface is 12 µs and can be calculated as follows:
````
s = packet size in bits / line speed in bps
s = (1500 bytes × 8) / 1 Gbps
s = 12,000 bits / 1,000,000,000 bps = 0.000012 s × 1000 = .012 ms × 1000 = 12 µs
````

### Retraso de procesamiento
El retraso de procesamiento es el tiempo fijo que tarda un dispositivo de red en recibir un paquete por una interfaz de entrada y colocarlo en la cola de salida de la interfaz correspondiente. Este retraso depende de factores como:
- Velocidad de la CPU (en plataformas basadas en software)
- Utilización de la CPU (carga)
- Modo de switching de paquetes IP (switching por software, CEF por software o CEF por hardware)
- Arquitectura del router (centralizada o distribuida)
- Configuración de las interfaces de entrada y salida

### Variación de la latencia
La variación de la latencia, también conocida como _jitter_, es la diferencia de tiempo de tránsito entre paquetes de un mismo flujo. Por ejemplo, si un paquete tarda 50 ms en recorrer la red desde el origen al destino, y el siguiente paquete tarda 70 ms, el jitter es de 20 ms. Los principales factores que afectan la variación de la latencia son la demora en la cola de espera, los búferes de compensación de jitter y el tamaño variable de los paquetes. El _jitter_ se produce debido a la demora en la cola de espera que experimentan los paquetes durante la congestión de red. Esta demora depende del número y tamaño de los paquetes en cola, la velocidad de la red y el mecanismo de encolamiento. El encolamiento introduce demoras desiguales en los paquetes de un mismo flujo, lo que genera jitter. Los dispositivos de voz y vídeo suelen incorporar búferes de compensación de jitter para suavizar las variaciones en el tiempo de llegada de los paquetes. Estos búferes suelen ser dinámicos y pueden compensar variaciones de hasta 30 ms. Si un paquete no se recibe dentro de este margen de 30 ms, se descarta, lo que afecta la calidad de la voz o el vídeo. Para evitar el _jitter_ en el tráfico en tiempo real de alta prioridad, se recomienda utilizar mecanismos de encolamiento como LLQ (**Low Latency Queuing**), que prioriza el envío de los paquetes de dicha prioridad durante la congestión de red.

## Pérdida de paquetes
La pérdida de paquetes puede estar causada por diversos problemas de red, como fallos de hardware o errores en el cableado o el circuito de la capa física. Otra causa común de pérdida de paquetes es la congestión, que se puede prevenir mediante las siguientes medidas:

- Aumentar la velocidad de la conexión.
- Implementar mecanismos de gestión y prevención de la congestión (QoS).
- Implementar el control de tráfico para descartar los paquetes de baja prioridad y permitir el paso del tráfico de alta prioridad.
- Implementar el modelado de tráfico para retrasar los paquetes en lugar de descartarlos, ya que el tráfico puede experimentar picos y superar la capacidad del búfer de la interfaz. El modelado de tráfico no se recomienda para el tráfico en tiempo real, ya que se basa en la cola de espera, lo que puede provocar fluctuaciones de latencia.

> [!NOTE]
> El control de tráfico estándar no está diseñado para gestionar los picos de tráfico que ocurren en intervalos de tiempo de microsegundos (es decir, microráfagas). En los casos en que sea necesario suavizar estos picos de tráfico, se requiere un sistema de control de tráfico específico para microsegundos o para ráfagas de baja duración.

# Modelos de QoS: 

Existen tres modelos de implementación de QoS:
- Best effort: En este modelo no se habilita la QoS. Se utiliza para el tráfico que no requiere un tratamiento especial.
- Servicios integrados (IntServ): Las aplicaciones solicitan a la red la reserva de ancho de banda y especifican que requieren un tratamiento QoS específico.
- Servicios diferenciados (DiffServ): La red identifica las clases de tráfico que requieren un tratamiento QoS específico.

El modelo IntServ fue creado para aplicaciones en tiempo real, como las de voz y vídeo, que requieren garantías de ancho de banda, latencia y pérdida de paquetes para asegurar un nivel de servicio predecible y confiable. En este modelo, las aplicaciones notifican a la red sus requisitos para reservar los recursos de extremo a extremo (como el ancho de banda) necesarios para garantizar una buena experiencia de usuario. IntServ utiliza el `Resource Reservation Protocol` (RSVP) para reservar recursos en toda la red para una aplicación específica y para implementar el `call admission control` (CAC), garantizando que ningún otro tráfico IP pueda utilizar el ancho de banda reservado. El ancho de banda reservado por una aplicación que no se está utilizando se desperdicia. Para proporcionar una calidad de servicio (QoS) de extremo a extremo, todos los nodos, incluidos los terminales que ejecutan las aplicaciones, deben soportar, configurar y mantener el estado de la ruta RSVP para cada flujo. Este es el principal inconveniente de IntServ, ya que dificulta su escalabilidad en redes grandes con miles o millones de flujos, debido al gran número de flujos RSVP que habría que gestionar. 

La figura 14-1 ilustra cómo los hosts RSVP realizan las reservas de ancho de banda.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/0e109f4d8d93fac0d8cdbbf269cf5c0ccf2fb55d/14.%20QOS/IMG/QOS%20MOQ/QOS%20MOQ%201.JPG)

En la figura 14-1, cada host del lado izquierdo (emisores) intenta establecer una reserva de ancho de banda unidireccional con cada host del lado derecho (receptores). Los emisores comienzan enviando mensajes RSVP PATH a los receptores a través del mismo camino que utilizan los paquetes de datos. Estos mensajes contienen la dirección IP del emisor, la dirección IP del receptor y el ancho de banda que se desea reservar. Esta información se almacena en el estado de ruta RSVP de cada nodo. Una vez que los mensajes RSVP PATH llegan a los receptores, estos envían mensajes de solicitud de reserva RSVP (RESV) en sentido inverso, de nodo en nodo. En cada salto, la dirección IP de destino de un mensaje RESV es la dirección IP del nodo anterior, obtenida del estado de ruta RSVP. Al atravesar cada nodo, los mensajes RSVP RESV reservan ancho de banda en cada enlace para el tráfico que fluye desde los receptores a los emisores.

Si se requieren reservas de ancho de banda en el sentido inverso, los hosts del lado derecho deben seguir el mismo procedimiento de envío de mensajes RSVP PATH, lo que duplica el estado RSVP en cada dispositivo de red. Esto demuestra cómo el estado RSVP puede aumentar rápidamente a medida que más hosts reservan ancho de banda. Además de los problemas de escalabilidad, la distancia entre los hosts puede provocar largas demoras en la reserva de ancho de banda.

DiffServ fue diseñado para superar las limitaciones de los modelos de mejor esfuerzo e IntServ. Este modelo no requiere un protocolo de señalización ni mantiene el estado de flujo RSVP en cada nodo, lo que lo hace altamente escalable. Las características de calidad de servicio (como ancho de banda y latencia) se gestionan de nodo a nodo mediante políticas de calidad de servicio definidas independientemente en cada dispositivo de la red. DiffServ no se considera una solución de calidad de servicio de extremo a extremo, ya que no garantiza la calidad de servicio en toda la ruta.

DiffServ clasifica el tráfico IP según las necesidades empresariales y lo etiqueta para asignar a cada clase un nivel de servicio diferente. Al atravesar la red, cada dispositivo identifica la clase del paquete según su etiqueta y lo procesa en consecuencia. DiffServ permite seleccionar diversos niveles de servicio. Por ejemplo, el tráfico de voz por IP es muy sensible a la latencia y la fluctuación, por lo que debe tener siempre prioridad sobre otros tipos de tráfico. El correo electrónico, en cambio, puede tolerar mayor latencia y podría recibir un servicio de mejor esfuerzo, mientras que el tráfico no crítico, como el de YouTube, podría tener un límite de velocidad o incluso bloquearse. El modelo DiffServ es el más popular y el más implementado para la gestión de la calidad de servicio, y se describe con detalle en este capítulo.

# Modular QoS CLI: 

Modular QoS CLI (MQC) es el enfoque de Cisco para implementar la QoS en los routers de Cisco. MQC proporciona un marco de CLI modular para crear políticas de QoS que permiten clasificar el tráfico y realizar acciones de QoS, como marcación, control de tráfico, modelado de tráfico, gestión de congestión y prevención de congestión. Las políticas MQC se implementan mediante los siguientes tres componentes:

- Mapas de clases: Estos definen los criterios de clasificación del tráfico, identificando el tipo de tráfico que requiere tratamiento de QoS (por ejemplo, tráfico de voz). Normalmente se requieren varios mapas de clases para identificar todos los tipos de tráfico que deben clasificarse. Se definen mediante el comando `class-map [match-any | match-all] class-map-name`, y cada mapa de clases puede incluir uno o más comandos de `match` para la clasificación del tráfico.
- Mapas de políticas: Estos definen las acciones de QoS para cada clase de tráfico. Cada mapa de políticas puede incluir una o más clases de tráfico, y cada clase puede tener una o más acciones de QoS. Se configuran mediante el comando `policy-map policy-map-name`.
- Políticas de servicio: Estas se utilizan para aplicar los mapas de políticas a las interfaces, en dirección de entrada y/o salida. Se aplican con el comando `servicepolicy {input | output} policy-map-name`, y la misma política se puede reutilizar en varias interfaces.
Un mapa de políticas consta de dos elementos principales:
- Una clase de tráfico para clasificar el tráfico, identificada con el comando `class class-map-name`
- La(s) acción(es) de QoS a aplicar al tráfico que coincide con la clase de tráfico

Dado que los mapas de políticas utilizan clases de tráfico para aplicar las acciones de QoS, estas acciones se denominan acciones de QoS basadas en clases. Las siguientes acciones de QoS basadas en clases, compatibles con MQC, pueden aplicarse al tráfico que coincide con una clase de tráfico:

- `Class-based weighted fair queuing` (CBWFQ)
- Class-based policing
- Class-based shaping
- Class-based marking
  
Todo tráfico que no coincida con ninguna de las clases de tráfico definidas por el usuario en el mapa de políticas se considera tráfico no clasificado. Este tráfico se asigna a una clase predeterminada, denominada `class-default`, que se configura automáticamente. El tráfico no clasificado suele ser tráfico de mejor esfuerzo que no requiere garantías de QoS; sin embargo, si es necesario, también se pueden aplicar acciones de QoS basadas en clases a esta clase predeterminada.

Los mapas de políticas no tienen efecto hasta que se aplican a una interfaz, ya sea en dirección de entrada o de salida, mediante el comando `service-policy {input | output} policy-map-name`.

Las políticas de servicio permiten aplicar el mismo mapa de políticas a múltiples interfaces. También se pueden utilizar para aplicar un mapa de políticas de entrada _AND/OR_ uno de salida a una única interfaz. La figura 14-2 muestra un ejemplo del marco de trabajo MQC. 

La sintaxis de los comandos y la descripción de cada uno de ellos se explican en secciones posteriores de este capítulo.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/0e109f4d8d93fac0d8cdbbf269cf5c0ccf2fb55d/14.%20QOS/IMG/QOS%20MOQ/QOS%20MOQ%202.JPG)

> [!NOTE]
> Los nombres class map y policy map son casos sensitivos. Se recomienda utilizar únicamente letras mayúsculas para el nombre, ya que facilita la lectura de la configuración.

La figura 14-2 muestra un mapa de políticas aplicado a una interfaz, pero los mapas de políticas también se pueden aplicar a otras políticas (también denominadas parent policies) para crear mapas de políticas de QoS jerárquicos (también llamados nested policy maps). El comando `service-policy policy-map-name` se utiliza para aplicar un mapa de políticas secundario dentro de un mapa de políticas principal.

La figura 14-3 muestra un mapa de políticas denominado CHILD-POLICY que se aplica a la default class de otro mapa de políticas llamado PARENT-POLICY, mediante el comando `service-policy policy-map-name`. El mapa de políticas PARENT-POLICY es el que se aplica a la interfaz.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/0e109f4d8d93fac0d8cdbbf269cf5c0ccf2fb55d/14.%20QOS/IMG/QOS%20MOQ/QOS%20MOQ%204.JPG)

# Clasificación y Marking: 

Antes de poder aplicar cualquier mecanismo de QoS, el tráfico IP debe identificarse y categorizarse en diferentes clases, según las necesidades de la empresa. Los dispositivos de red utilizan la clasificación para determinar a qué clase pertenece el tráfico IP. Una vez clasificado el tráfico IP, se puede utilizar el marcado para identificar paquetes individuales, de modo que otros dispositivos de red puedan aplicar mecanismos de QoS a dichos paquetes a medida que transitan por la red. Esta sección introduce los conceptos de clasificación y marcado, explica las diferentes opciones de marcado disponibles para los paquetes de capa 2 y 3, y describe dónde deben utilizarse las herramientas de clasificación y marcado en una red.

## Clasificación:

La _clasificación de paquetes_ es un mecanismo de calidad de servicio (QoS) que permite diferenciar entre los distintos flujos de tráfico. Para ello, utiliza descriptores de tráfico que categorizan cada paquete IP en una clase específica. La clasificación de paquetes debe realizarse en el borde de la red, lo más cerca posible del origen del tráfico. Una vez clasificado el paquete IP, se pueden aplicar diversas acciones, como marcarlo, ponerlo en cola, controlar su velocidad, limitar su ancho de banda o realizar cualquier combinación de estas y otras acciones. Los descriptores de tráfico más comunes para la clasificación son:

- Internos: Grupos de QoS (relevantes para el router)
- Capa 1: Interfaz física, subinterfaz o puerto
- Capa 2: Dirección MAC y bits de clase de servicio (CoS) de 802.1Q/p
- Capa 2.5: Bits experimentales (EXP) de MPLS
- Capa 3: _Differentiated Services Code Points (DSCP)_, IP Precedence (IPP) y direcciones IP de origen/destino
- Capa 4: Puertos TCP o UDP
- Capa 7: Next-Generation Network-Based Application Recognition (NBAR2)

En el caso de las redes empresariales, los descriptores de tráfico más utilizados para la clasificación incluyen los descriptores de los niveles 2, 3, 4 y 7, que se enumeran a continuación.

### Clasificación de capa 7:

NBAR2 es un motor de inspección profunda de paquetes que puede clasificar e identificar una amplia variedad de protocolos y aplicaciones utilizando datos de las capas 3 a 7, incluyendo aplicaciones difíciles de clasificar que asignan dinámicamente numeros de puertos Transmission Control Protocol (TCP) o User Datagram
Protocol (UDP).

NBAR2 puede reconocer casi 1500 aplicaciones, y se proporcionan paquetes de protocolos mensuales para el reconocimiento de nuevas aplicaciones emergentes, sin necesidad de actualizar el sistema operativo ni reiniciar el router.
NBAR2 tiene dos modos de operación independientes:

- Detección de protocolos: Esta función permite a NBAR2 detectar y obtener estadísticas en tiempo real sobre las aplicaciones que se están ejecutando en la red.
- Modular QoS CLI (MQC): Mediante MQC, el comando «match protocol» en el modo de configuración de mapa de clases se utiliza para la clasificación de tráfico de NBAR2. NBAR2 clasifica el tráfico no solo por los números de puerto TCP/UDP, sino también mediante la clasificación de subpuertos, es decir, la clasificación del tráfico basada en el contenido de la carga útil TCP/UDP. Ejemplos incluyen cadenas URL, Multipurpose Internet Mail Extension (MIME), métodos de solicitud HTTP, tráfico P2P, análisis de comportamiento, etc.

## Configuración de clasificación MQC

La clasificación de tráfico mediante MQC se realiza mediante el comando `match` en el modo de configuración de `class-map`. Si el tráfico clasificado cumple con la condición especificada por el comando `match`, este devuelve un resultado de coincidencia; en caso contrario, devuelve un resultado de no coincidencia.
El comando match en un mapa de clases tiene las siguientes características:

- Compara las características de los paquetes, como CoS, DSCP o ACL (redes o puertos de origen/destino).
- Si no se especifica ningún comando `match` en el `class map`, el valor predeterminado es `match none`.
- El comando match any en un mapa de clases coincide con todos los paquetes.
- El comando `match protocol` en un mapa de clases se utiliza para la clasificación de tráfico NBAR.

Cuando hay más de un comando de `match` en una clase, las opciones de coincidencia en el comando `class-map [match-any | match-all] class-map-name` determinan los criterios de selección:

- Match any (operación lógica OR): Debe cumplirse al menos una de las condiciones de coincidencia definidas en la clase. Esta opción se configura con el comando `class-map match-any`.
- Match all (operación lógica AND): Deben cumplirse todas las condiciones de coincidencia definidas en la clase. Esta es la opción de coincidencia predeterminada si no se especifica el comando `class-map match-any`. También se puede configurar explícitamente con el comando `class-map match-all`.

La tabla 14-2 muestra los comandos de `match` más comunes utilizados con MQC para la clasificación del tráfico.

Tabla 14-2: Comandos de coincidencia de MQC más utilizados y su descripción
Comando|Descripcion
:---|:---
`match access-group {access-listnumber \x`| name access-list name}`|Busca coincidencias con una ACL por número o nombre.
`match any`|Coincide con cualquier paquete.
`match cos cos-value-list`|Compara el valor CoS de capa 2 de un paquete con uno de los valores especificados. Se pueden especificar hasta ocho valores CoS diferentes en la lista.
`match [ip] dscp dscp-value-list`|Con la palabra clave opcional "ip", se realiza la comparación con uno de los valores DSCP especificados en los paquetes IPv4. Sin la palabra clave "ip", la comparación se realiza con uno de los valores DSCP especificados en los paquetes IPv4 e IPv6. Se pueden especificar hasta ocho valores DSCP diferentes en la lista.
`match [ip] precedence precedence-value-list`|Con la palabra clave opcional "ip", se realiza la comparación con uno de los valores de IP Precedence en los paquetes IPv4. Sin esta palabra clave, la comparación se realiza con uno de los valores de IP Precedence en los paquetes IPv4 e IPv6. Se pueden especificar hasta cuatro valores de IP Precedence diferentes en la lista.
`match ip rtp starting-port-number port-number-range`|Coincide con cualquier número de puerto RTP dentro del rango especificado.
`match protocol protocol-name`|Coincide con el nombre de protocolo NBAR especificado.
`match qos-group qos-group-value`|Coincide con el valor del grupo de QoS especificado. Solo se puede especificar un valor de grupo de QoS.

La tabla 14-3 muestra los comandos de coincidencia de protocolos NBAR2 más utilizados.
Comando|Descripcion
:---|:---
`match protocol http [url url-string \| host hostname-string \| mime MIME-type]`| Identifica el tráfico HTTP según la URL, el nombre de host o el tipo MIME
`match protocol rtp [audio \| video \| payloadtype payload-string]`| Identifica correspondencia RTP según el tipo de carga útil de audio o vídeo, o según un tipo de carga útil más específico.
`match protocol peer-to-peer-application`|Identifica aplicaciones peer-to-peer (P2P) como Soulseek, BitTorrent y Skype.

# Policing y Shaping: 
# Gestión y prevención de la congestión: 


