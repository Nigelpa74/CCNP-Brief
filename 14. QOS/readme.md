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

La _latencia de red_, también conocida como retardo de end-to-end, es el tiempo que tardan los paquetes en recorrer la red desde el origen hasta el destino. La Recomendación G.114 de la UIT establece que, independientemente del tipo de aplicación, la latencia de red no debería superar los 400 ms, y para el tráfico en tiempo real, debería ser inferior a 150 ms. Sin embargo, la UIT y Cisco han demostrado que la calidad del tráfico en tiempo real no se ve afectada significativamente hasta que la latencia de red supera los 200 ms. Para poder implementar estas recomendaciones, es importante comprender las causas de la latencia de red. Esta latencia puede dividirse en latencia fija y latencia variable:

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

El modelo IntServ fue creado para aplicaciones en tiempo real, como las de voz y vídeo, que requieren garantías de ancho de banda, latencia y pérdida de paquetes para asegurar un nivel de servicio predecible y confiable. En este modelo, las aplicaciones notifican a la red sus requisitos para reservar los recursos de end-to-end (como el ancho de banda) necesarios para garantizar una buena experiencia de usuario. IntServ utiliza el `Resource Reservation Protocol` (RSVP) para reservar recursos en toda la red para una aplicación específica y para implementar el `call admission control` (CAC), garantizando que ningún otro tráfico IP pueda utilizar el ancho de banda reservado. El ancho de banda reservado por una aplicación que no se está utilizando se desperdicia. Para proporcionar una calidad de servicio (QoS) de end-to-end, todos los nodos, incluidos los terminales que ejecutan las aplicaciones, deben soportar, configurar y mantener el estado de la ruta RSVP para cada flujo. Este es el principal inconveniente de IntServ, ya que dificulta su escalabilidad en redes grandes con miles o millones de flujos, debido al gran número de flujos RSVP que habría que gestionar. 

La figura 14-1 ilustra cómo los hosts RSVP realizan las reservas de ancho de banda.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/0e109f4d8d93fac0d8cdbbf269cf5c0ccf2fb55d/14.%20QOS/IMG/QOS%20MOQ/QOS%20MOQ%201.JPG)

En la figura 14-1, cada host del lado izquierdo (emisores) intenta establecer una reserva de ancho de banda unidireccional con cada host del lado derecho (receptores). Los emisores comienzan enviando mensajes RSVP PATH a los receptores a través del mismo camino que utilizan los paquetes de datos. Estos mensajes contienen la dirección IP del emisor, la dirección IP del receptor y el ancho de banda que se desea reservar. Esta información se almacena en el estado de ruta RSVP de cada nodo. Una vez que los mensajes RSVP PATH llegan a los receptores, estos envían mensajes de solicitud de reserva RSVP (RESV) en sentido inverso, de nodo en nodo. En cada salto, la dirección IP de destino de un mensaje RESV es la dirección IP del nodo anterior, obtenida del estado de ruta RSVP. Al atravesar cada nodo, los mensajes RSVP RESV reservan ancho de banda en cada enlace para el tráfico que fluye desde los receptores a los emisores.

Si se requieren reservas de ancho de banda en el sentido inverso, los hosts del lado derecho deben seguir el mismo procedimiento de envío de mensajes RSVP PATH, lo que duplica el estado RSVP en cada dispositivo de red. Esto demuestra cómo el estado RSVP puede aumentar rápidamente a medida que más hosts reservan ancho de banda. Además de los problemas de escalabilidad, la distancia entre los hosts puede provocar largas demoras en la reserva de ancho de banda.

DiffServ fue diseñado para superar las limitaciones de los modelos de mejor esfuerzo e IntServ. Este modelo no requiere un protocolo de señalización ni mantiene el estado de flujo RSVP en cada nodo, lo que lo hace altamente escalable. Las características de calidad de servicio (como ancho de banda y latencia) se gestionan de nodo a nodo mediante políticas de calidad de servicio definidas independientemente en cada dispositivo de la red. DiffServ no se considera una solución de calidad de servicio de end-to-end, ya que no garantiza la calidad de servicio en toda la ruta.

DiffServ clasifica el tráfico IP según las necesidades empresariales y lo etiqueta para asignar a cada clase un nivel de servicio diferente. Al atravesar la red, cada dispositivo identifica la clase del paquete según su etiqueta y lo procesa en consecuencia. DiffServ permite seleccionar diversos niveles de servicio. Por ejemplo, el tráfico de voz por IP es muy sensible a la latencia y la fluctuación, por lo que debe tener siempre prioridad sobre otros tipos de tráfico. El correo electrónico, en cambio, puede tolerar mayor latencia y podría recibir un servicio de mejor esfuerzo, mientras que el tráfico no crítico, como el de YouTube, podría tener un límite de velocidad o incluso bloquearse. El modelo DiffServ es el más popular y el más implementado para la gestión de la calidad de servicio, y se describe con detalle en este capítulo.

# Modular QoS CLI: 

Modular QoS CLI (MQC) es el enfoque de Cisco para implementar la QoS en los routers de Cisco. MQC proporciona un marco de CLI modular para crear políticas de QoS que permiten clasificar el tráfico y realizar acciones de QoS, como marcación, control de tráfico, modelado de tráfico, gestión de congestión y prevención de congestión. Las políticas MQC se implementan mediante los siguientes tres componentes:

- Mapas de clases: Estos definen los criterios de clasificación del tráfico, identificando el tipo de tráfico que requiere tratamiento de QoS (por ejemplo, tráfico de voz). Normalmente se requieren varios mapas de clases para identificar todos los tipos de tráfico que deben clasificarse. Se definen mediante el comando `class-map [match-any | match-all] class-map-name`, y cada mapa de clases puede incluir uno o más comandos de `match` para la clasificación del tráfico.
- Mapas de políticas: Estos definen las acciones de QoS para cada clase de tráfico. Cada policy map puede incluir una o más clases de tráfico, y cada clase puede tener una o más acciones de QoS. Se configuran mediante el comando `policy-map policy-map-name`.
- Políticas de servicio: Estas se utilizan para aplicar los mapas de políticas a las interfaces, en dirección de entrada y/o salida. Se aplican con el comando `servicepolicy {input | output} policy-map-name`, y la misma política se puede reutilizar en varias interfaces.
Un policy map consta de dos elementos principales:
- Una clase de tráfico para clasificar el tráfico, identificada con el comando `class class-map-name`
- La(s) acción(es) de QoS a aplicar al tráfico que coincide con la clase de tráfico

Dado que los mapas de políticas utilizan clases de tráfico para aplicar las acciones de QoS, estas acciones se denominan acciones de QoS basadas en clases. Las siguientes acciones de QoS basadas en clases, compatibles con MQC, pueden aplicarse al tráfico que coincide con una clase de tráfico:

- `Class-based weighted fair queuing` (CBWFQ)
- Class-based policing
- Class-based shaping
- Class-based marking
  
Todo tráfico que no coincida con ninguna de las clases de tráfico definidas por el usuario en el policy map se considera tráfico no clasificado. Este tráfico se asigna a una clase predeterminada, denominada `class-default`, que se configura automáticamente. El tráfico no clasificado suele ser tráfico de mejor esfuerzo que no requiere garantías de QoS; sin embargo, si es necesario, también se pueden aplicar acciones de QoS basadas en clases a esta clase predeterminada.

Los mapas de políticas no tienen efecto hasta que se aplican a una interfaz, ya sea en dirección de entrada o de salida, mediante el comando `service-policy {input | output} policy-map-name`.

Las políticas de servicio permiten aplicar el mismo policy map a múltiples interfaces. También se pueden utilizar para aplicar un policy map de entrada _AND/OR_ uno de salida a una única interfaz. La figura 14-2 muestra un ejemplo del marco de trabajo MQC. 

La sintaxis de los comandos y la descripción de cada uno de ellos se explican en secciones posteriores de este capítulo.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/0e109f4d8d93fac0d8cdbbf269cf5c0ccf2fb55d/14.%20QOS/IMG/QOS%20MOQ/QOS%20MOQ%202.JPG)

> [!NOTE]
> Los nombres class map y policy map son casos sensitivos. Se recomienda utilizar únicamente letras mayúsculas para el nombre, ya que facilita la lectura de la configuración.

La figura 14-2 muestra un policy map aplicado a una interfaz, pero los mapas de políticas también se pueden aplicar a otras políticas (también denominadas parent policies) para crear mapas de políticas de QoS jerárquicos (también llamados nested policy maps). El comando `service-policy policy-map-name` se utiliza para aplicar un policy map secundario dentro de un policy map principal.

La figura 14-3 muestra un policy map denominado CHILD-POLICY que se aplica a la default class de otro policy map llamado PARENT-POLICY, mediante el comando `service-policy policy-map-name`. El policy map PARENT-POLICY es el que se aplica a la interfaz.

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

El ejemplo 14-1 muestra varios mapas de clasificación, cada uno con diferentes comandos de coincidencia. El VOIPTELEPHONY class map está configurado con la opción **match-all** (operación lógica AND), mientras que el CONTROL class map está configurado con la opción **match-any** (operación lógica OR). Para que el VOIP-TELEPHONY class map sea matched, el paquete analizado debe tener la marca DSCP EF y, además, debe estar permitido por la ACL VOICE-TRAFFIC. Para que haga match con CONTROL class map, el paquete analizado debe coincidir con alguno de los valores DSCP especificados en la lista o estar permitido por la ACL CALL-CONTROL. Los maps class que se utilizan se muestran con el comando **match protocol** muestranndo como clasificar el tráfico mediante NBAR2.

Ejemplo 14-1: Ejemplo de class-map
````
ip access-list extended VOICE-TRAFFIC
  10 permit udp any any range 16384 32767
  20 permit udp any range 16384 32767 any

ip access-list extended CALL-CONTROL
  10 permit tcp any any eq 1719
  20 permit tcp any eq 1719 any

class-map match-all VOIP-TELEPHONY
  match dscp ef
  match access-group name VOICE-TRAFFIC

class-map match-any CONTROL
  match dscp cs3 af31 af32 af33
  match access-group name CALL-CONTROL

class-map match-any HTTP-VIDEO
  match protocol http mime "video/*"

class-map match-any P2P
  match protocol bittorrent
  match protocol soulseek

class-map match-all RTP-AUDIO
  match protocol rtp audio

class-map match-all HTTP-WEB-IMAGES
  match protocol http url "*.jpeg|*.jpg"
````

## Marking de paquetes

El marcado de paquetes es un mecanismo de QoS que permite identificar un paquete mediante la modificación de un campo en su cabecera, utilizando un descriptor de tráfico. Esto facilita la diferenciación de dicho paquete respecto a otros durante la aplicación de otros mecanismos de QoS (como la reasignación de prioridad, el control de tráfico, la gestión de colas o la prevención de congestión). Los siguientes descriptores de tráfico se utilizan para el marcado:

- Interno: Grupos de QoS
- Capa 2: Bits de clase de servicio (CoS) 802.1Q/p
- Capa 2.5: Bits experimentales (EXP) de MPLS
- Capa 3: Differentiated Services Code Points (DSCP) e IP Precedence (IPP)

> [!NOTE]
> Los grupos de QoS se utilizan para etiquetar los paquetes al recibirlos y procesarlos internamente en el router, y se eliminan automáticamente cuando los paquetes salen del mismo. Se utilizan únicamente en casos especiales en los que los descriptores de tráfico marcados o recibidos en una interfaz de entrada no serían visibles para la clasificación de paquetes en las interfaces de salida debido a la encapsulación o desencapsulación.

En las redes empresariales, los descriptores de tráfico más utilizados para la clasificación del tráfico incluyen los descriptores de capa 2 y capa 3 mencionados en la lista anterior. Ambos se describen en las siguientes secciones.

### Marking de capa 2

El estándar 802.1Q es una especificación de IEEE para la implementación de VLAN en redes conmutadas de capa 2. Esta especificación define dos campos de 2 bytes: el Tag Protocol Identifier (TPID) y Tag Control Information (TCI), que se insertan en el marco Ethernet, después del campo de dirección de origen, tal como se muestra en la figura 14-4.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/b70467a4b8f9d8e0158cf3f6c7e56055e4d68baf/14.%20QOS/IMG/QOS%20MARK/QOS%20MARK%201.JPG)

El valor TPID es un campo de 16 bits que tiene asignado el valor 0x8100 y que identifica el paquete como un paquete etiquetado según el estándar 802.1Q.

El campo TCI consta de 16 bits y está compuesto por los siguientes tres subcampos:
- Campo de Priority Code Point (PCP) (3 bits)
- Campo de Drop Eligible Indicator (DEI) (1 bit)
- Campo de VLAN Identifier (VLAN ID) (12 bits)

#### Priority Code Point (PCP)

Las especificaciones del campo PCP de 3 bits están definidas en la norma IEEE 802.1p. Este campo se utiliza para identificar los paquetes según su nivel de calidad de servicio (QoS). La marcación QoS permite asignar a cada trama Ethernet de capa 2 uno de los ocho niveles de prioridad (del 0 al 7), siendo 0 la prioridad más baja y 7 la más alta. La tabla 14-4 muestra la definición estándar de la norma IEEE 802.1p para cada nivel de QoS.

Tabla 14-4 Definiciones de CoS de IEEE 802.1p
PCP Valor/Prioridad|Acronimo|Tipo de trafico
:---|:---|:---
0 (nivel más bajo) | BK| Background
1 (predeterminado) | BE| Best effort
2 | EE| Excellent effort
3 | CA| Critical applications
4 | VI| Vídeo con latencia y jitter < 100 ms
5 | VO| Voz con latencia y jitter < 10 ms
6 | IC| Internetwork control
7 (nivel más alto) | NC| Network control

Una desventaja del uso de la marcación CoS es que los paquetes pierden esta información al atravesar un enlace que no cumple con el estándar 802.1Q o una red de capa 3. Por ello, es recomendable marcar los paquetes con otros indicadores de capa superior para preservar la información de priorización de end-to-end. Esto se suele lograr mediante la asignación de un valor CoS a otro indicador. Por ejemplo, los niveles de prioridad CoS corresponden directamente a los valores de **type of service (ToS)** de IPv4, por lo que se pueden asignar directamente.

#### Indicador de paquete descartable (DEI Drop Eligible Indicator):

El campo DEI es de 1 bit y puede usarse de forma independiente o junto con PCP para indicar qué paquetes pueden descartarse en caso de congestión. Por defecto, su valor es 0 (paquete no descartable); se puede establecer en 1 para indicar que el paquete es descartable.

#### Identificador de VLAN (VLAN ID)

El campo VLAN ID es de 12 bits y define la VLAN utilizada por 802.1Q. Al ser de 12 bits, limita el número de VLANs soportadas por 802.1Q a 4096, lo cual puede resultar insuficiente para redes empresariales grandes o de proveedores de servicios.

### Marcación de capa 3

Al viajar desde su origen a su destino, un paquete puede atravesar enlaces troncales no 802.1Q o enlaces no Ethernet que no soportan el campo CoS. La marcación en capa 3 proporciona una información de priorización más persistente, que se conserva de end-to-end.

La figura 14-5 ilustra el campo ToS/DiffServ en el encabezado IPv4.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/b70467a4b8f9d8e0158cf3f6c7e56055e4d68baf/14.%20QOS/IMG/QOS%20MARK/QOS%20MARK%202.JPG)

El campo ToS, definido en la RFC 791, es un campo de 8 bits, donde solo los 3 primeros bits, denominados IP Precedence (IPP), se utilizan para la clasificación del tráfico, mientras que los restantes permanecen sin usar. Los valores de IPP, que van de 0 a 7, permiten clasificar el tráfico en hasta seis clases de servicio; los valores 6 y 7 están reservados para uso interno de red.

La RFC 2474 redefinió los campos ToS de IPv4 y Clase de tráfico de IPv6 como un campo DiffServ de 8 bits (DS field). Este campo utiliza los mismos 8 bits que antes se utilizaban para los campos ToS de IPv4 y Clase de tráfico de IPv6, lo que garantiza la compatibilidad con la IP Precedence. El campo DS consta de un campo Differentiated Services Code Point (DSCP) de 6 bits, que permite clasificar el tráfico en hasta 64 valores (0 a 63), y un campo Explicit Congestion Notification (ECN) de 2 bits.

## DSCP Per-Hop Behaviors:

Los paquetes se clasifican y marcan para recibir un comportamiento de reenvío específico en cada nodo de la red a lo largo de su ruta hacia el destino (es decir, prioridad, retraso o descarte). El campo DS se utiliza para marcar los paquetes según su clasificación en DiffServ Behavior Aggregates (BAs). Una DiffServ BA es un conjunto de paquetes con el mismo valor DiffServ que atraviesan un enlace en una dirección determinada. El **Per-hop behavior (PHB)**  es el comportamiento de reenvío observable externamente (tratamiento de reenvío) aplicado en un DiffServ-compliant node a un conjunto de paquetes con el mismo valor DiffServ que atraviesan un enlace en una dirección determinada (BA DiffServ).

En otras palabras, el PHB consiste en priorizar, retrasar o descartar un conjunto de paquetes mediante uno o varios mecanismos de QoS, en función del valor DSCP. Una BA DiffServ puede incluir varias aplicaciones, por ejemplo, SSH, Telnet y SNMP, todas agrupadas y marcadas con el mismo valor DSCP. De esta manera, el núcleo de la red realiza un PHB simple, basado en las BA DiffServ, mientras que el borde de la red realiza la clasificación, la marcación, el control de tráfico y la conformación. Esto hace que el modelo de QoS DiffServ sea altamente escalable.

Se han definido y caracterizado cuatro tipos de PHB para uso general:

- Class Selector (CS) PHB: Los 3 primeros bits del campo DSCP se utilizan como bits CS. Estos bits permiten la compatibilidad descendente de DSCP con la Precedencia IP, ya que esta última utiliza los mismos 3 bits para determinar la clase.
- Default Forwarding (DF) PHB: Se utiliza para el servicio de best-effort.
- Assured Forwarding (AF) PHB: Se utiliza para el servicio de ancho de banda garantizado.
- Expedited Forwarding (EF) PHB: Se utiliza para el servicio de baja latencia.

### Class Selector (CS) PHB:

El RFC 2474 declaró obsoleto el campo ToS al introducir el campo DS, y el Class Selector(CS) PHB se definió para garantizar la compatibilidad con versiones anteriores de DSCP y IP Precedence.

La figura 14-6 ilustra el funcionamiento del selector de clase PHB.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/b70467a4b8f9d8e0158cf3f6c7e56055e4d68baf/14.%20QOS/IMG/QOS%20MARK/QOS%20MARK%203.JPG)

Los paquetes con mayor  IP Precedence deben reenviarse más rápidamente que los paquetes con menor IP Precedence.

Los tres últimos bits del DSCP (bits 2 a 4), cuando están en 0, identifican un selector de clase PHB, pero los bits 5 a 7 del selector de clase son los que determinan la IP Precedence. Los dispositivos que son non-DiffServ-compliant y que realizan la clasificación basada en la IP Precedence ignoran los bits 2 a 4.

Existen ocho clases CS, desde CS0 hasta CS7, que corresponden directamente con los ocho valores de IP Precedence.

### Default Forwarding (DF) PHB:

El reenvío predeterminado (DF) y Class Selector 0 (CS0) proporcionan un servicio de best-effort y utilizan el valor DS 000000. La figura 14-7 ilustra el DF PHB.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/b70467a4b8f9d8e0158cf3f6c7e56055e4d68baf/14.%20QOS/IMG/QOS%20MARK/QOS%20MARK%204.JPG)

El reenvío por defecto best-effort también se aplica a los paquetes que no pueden clasificarse mediante un mecanismo de QoS como _queuing, shaping, o policing_. Esto suele ocurrir cuando la política de QoS del nodo está incompleta o cuando los valores DSCP no se ajustan a los definidos para las clases de servicio CS, AF y EF.

### Assured Forwarding (AF) PHB:

La clase de servicio AF garantiza un ancho de banda mínimo y permite el acceso a ancho de banda adicional, si está disponible. Los paquetes que requieren la clase de servicio AF tienen una estructura DSCP de la forma aaadd0, donde aaa representa el valor binario de la clase AF (bits 5, 6 y 7), y dd (bits 2, 3 y 4) representa la Drop Probability (el bit 2 no se utiliza y siempre está en 0). La figura 14-8 ilustra la clase de servicio AF.

Existen cuatro clases AF definidas según el estándar: AF1, AF2, AF3 y AF4. El número de la clase AF no determina la prioridad; por ejemplo, la clase AF4 no recibe ningún tratamiento preferencial respecto a la clase AF1. Cada clase debe tratarse de forma independiente y asignarse a una cola diferente.

La tabla 14-5 muestra cómo se asigna una IP Precedence a cada clase AF (en la columna «Valor de clase AF») y las tres probabilidades de pérdida de paquetes: baja, media y alta.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/b70467a4b8f9d8e0158cf3f6c7e56055e4d68baf/14.%20QOS/IMG/QOS%20MARK/QOS%20MARK%205.JPG)

El nombre AF (AFxy) está compuesto por el valor de IP Precedence AF en decimal (x) y el valor de Drop Probability en decimal (y). Por ejemplo, AF41 combina la IP Precedence 4 con una Drop Probability 1.

Para convertir rápidamente el nombre AF en un valor DSCP decimal, utilice la fórmula 8x + 2y. Por ejemplo, el valor DSCP para AF41 es 8(4) + 2(1) = 34.

Tabla 14-5: Valores AF PHB con sus equivalentes decimales y binarios
DSCP Class|Valor DSCP Bin|Valor Decimal Dec|Drop Probability|Valor equivalente de IP Precedence
:---|:---|:---|:---|:---
DF (CS0)| 000 000| 0||0
CS1| 001 000| 8 ||1
AF11| 001 010| 10| Low| 1
AF12| 001 100| 12| Medium| 1
AF13| 001 110| 14| High |1
CS2| 010 000| 16|| 2
AF21|010 010| 18| Low |2
AF22| 010 100 |20| Medium| 2
AF23 |010 110| 22| High| 2
CS3| 011 000| 24|| 3
AF31| 011 010| 26| Low| 3
AF32| 011 100| 28| Medium| 3
AF33| 011 110| 30| High| 3
CS4| 100 000| 32|| 4
AF41| 100 010 |34| Low| 4
AF42| 100 100| 36| Medium| 4
AF43| 100 110| 38| High| 4
CS5| 101 000| 40|| 5
EF| 101 110| 46|| 5
CS6 |110 000 |48|| 6
CS7| 111 000 |56|| 7

## Clase de tráfico de baja prioridad (Scavenger class):

La scavenger class está diseñada para ofrecer un servicio con prioridad inferior al de calidad óptima. Las aplicaciones asignadas a esta clase aportan poco o ningún valor a los objetivos empresariales de la organización y suelen ser aplicaciones de entretenimiento. Entre ellas se incluyen aplicaciones peer-to-peer (como los clientes de torrent), juegos (por ejemplo, Minecraft, Fortnite) y aplicaciones de vídeo (por ejemplo, YouTube, Vimeo, Netflix). Este tipo de aplicaciones suelen tener un límite de velocidad de transmisión muy bajo o se bloquean por completo. 

Una característica particular de esta clase es que tiene una prioridad inferior a la del tráfico de calidad óptima. El tráfico de calidad óptima utiliza un código de prioridad (DSCP) de 000000 (CS0). Dado que no existen valores DSCP negativos, se decidió utilizar CS1 como marcador para el tráfico de baja prioridad. Este uso está definido en la RFC 4594.

## Límites de confianza (Trust Boundary)
Para garantizar una experiencia de QoS escalable y de end-to-end, los paquetes deben etiquetarse en el dispositivo final o lo más cerca posible de este. Cuando un dispositivo final etiqueta un paquete con un valor CoS o DSCP, el puerto del switch al que está conectado puede configurarse para aceptar o rechazar dichos valores. Si el switch acepta los valores, significa que confía en el dispositivo final y no necesita reetiquetar ni reclasificar los paquetes recibidos. Si el switch no confía en el dispositivo final, rechaza la etiqueta y reetiqueta los paquetes recibidos con el valor CoS o DSCP apropiado.

Por ejemplo, en una red de campus con telefonía IP y dispositivos finales, los teléfonos IP etiquetan por defecto el tráfico de voz con un valor CoS de 5 y un valor DSCP de 46 (EF), mientras que el tráfico entrante de un dispositivo final (como un PC) conectado al puerto del switch se reetiqueta con un valor CoS de 0 y un valor DSCP de 0. Aunque el dispositivo final envíe paquetes etiquetados con un valor CoS o DSCP específico, el comportamiento predeterminado de los teléfonos IP de Cisco es no confiar en el dispositivo final y restablecer los valores CoS y DSCP a 0 antes de enviar los paquetes al switch. Cuando el teléfono IP envía tráfico de voz y datos al switch, este puede clasificar el tráfico de voz como de mayor prioridad gracias a las etiquetas CoS y DSCP de alta prioridad.

Para garantizar la escalabilidad, la clasificación de los límites de confianza debe realizarse lo más cerca posible del dispositivo final. La figura 14-9 ilustra los límites de confianza en diferentes puntos de una red de campus, donde los puntos 1 y 2 son óptimos, y el punto 3 es aceptable solo si el switch de acceso no puede realizar la clasificación.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/b70467a4b8f9d8e0158cf3f6c7e56055e4d68baf/14.%20QOS/IMG/QOS%20MARK/QOS%20MARK%206.JPG)

## Configuración de marcación basada en clases

La marcación basada en clases mediante MQC se puede realizar de dos maneras:

1. Mediante el comando `set` dentro de una clase de tráfico de un policy map.
2. Marcando el tráfico que supera un umbral configurado mediante el comando `police` en un policy map.

En esta sección, solo se aborda la opción del comando `set`. La marcación mediante control de tráfico se explica en la sección “Policing and Shaping".

La tabla 14-7 muestra los comandos `set` que se utilizan para la configuración de marcación basada en clases. Se admiten varios comandos `set` dentro de una misma clase de tráfico.

Tabla 14-7: Comandos y descripciones del Class-Based Marking `set`:
Comando|Descripcion
:---|:---
`set qos-group <id_grupo_QoS>` | Marca el tráfico clasificado con un ID de grupo QoS interno
`set cos <valor_CoS>` | Marca el valor CoS de la capa 2
`set [ip] precedence <valor_precedencia_IP>` | Marca el valor de prioridad para paquetes IPv4 e IPv6
`set [ip] dscp <valor_DSCP>` | Marca el valor DSCP para paquetes IPv4 e IPv6

El ejemplo 14-2 muestra un policy map de entrada que utiliza el comando `set` para realizar el marcado del tráfico entrante.

Ejemplo 14-2: Ejemplo de política de marcado de tráfico entrante
````
policy-map INBOUND-MARKING-POLICY
  class VOIP-TELEPHONY
    set dscp ef
  class VIDEO
    set dscp af31
  class CONFERENCING
    set dscp af41
    set cos 4
  class class-default
    set dscp default
    set cos 0
````

## Ejemplo práctico: QoS en redes inalámbricas

Una red inalámbrica puede configurarse para aprovechar los mecanismos de QoS descritos en este capítulo. Por ejemplo, el wireless LAN controller (WLC) se sitúa en la frontera entre las redes inalámbricas y cableadas, lo que lo convierte en un punto ideal para implementar la gestión de QoS. El tráfico que entra y sale del WLC puede clasificarse y etiquetarse para que se procese adecuadamente durante su transmisión por el aire y a través de la red cableada.

La QoS en redes inalámbricas se puede configurar de forma específica para cada red LAN inalámbrica (WLAN), utilizando las cuatro categorías de tráfico que se muestran en la tabla 14-8. Cabe destacar que los nombres de las categorías son descriptivos y se corresponden con valores específicos de 802.1p y DSCP.

Tabla 14-8: Categorías y marcadores de la política de QoS para redes inalámbricas
QoS Category| Traffic Type| 802.1p Tag| DSCP Value
:---|:---|:---|:---
Platinum| Voice| 5| 46 (EF)
Gold| Video| 4| 34 (AF41)
Silver| Best effort (default)| 0| 0
Bronze| Background| 1| 10| (AF11)

Al crear una nueva red WLAN, la política de QoS predeterminada es "Silver", que garantiza un servicio de calidad estándar. En la figura 14-10, se ha creado una WLAN denominada "voice" para el tráfico de voz, por lo que su política de QoS se ha configurado en "Platinum". De esta manera, el tráfico de voz inalámbrico se clasifica para minimizar la latencia y la fluctuación, y se etiqueta con un valor CoS 802.1p de 5 y un valor DSCP de 46 (EF).

# Policing y Shaping: 
# Gestión y prevención de la congestión: 



