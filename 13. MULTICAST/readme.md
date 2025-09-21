El protocolo multicast se implementa en casi todos los tipos de red. Permite a un equipo origen enviar paquetes de datos a un grupo de equipos destino (_receivers_) de forma eficiente, optimizando el uso del ancho de banda y los recursos del sistema. Este capítulo describe la utilidad del protocolo multicast, así como los protocolos fundamentales necesarios para comprender su funcionamiento, como IGMP, PIM (modo denso/modo disperso) y los rendezvous points (RPs).

# Fundamentos de multicast: 

La comunicación IP tradicional entre dispositivos de red suele utilizar uno de los siguientes métodos de transmisión:
- Unicast (de uno a uno)
- Broadcast (de uno a todos)
- Multicast (de uno a varios)

La comunicación multicast es una tecnología que optimiza el uso del ancho de banda de la red y ahorra recursos del sistema. 

Para su funcionamiento en redes de capa 2, utiliza el `Internet Group Management Protocol (IGMP)`, mientras que en redes de capa 3 utiliza el `Protocol Independent Multicast (PIM)`.

La figura 13-1 ilustra cómo funciona IGMP entre los receivers y el enrutador multicast local, y cómo funciona PIM entre los enrutadores. Estas dos tecnologías trabajan conjuntamente para permitir que el tráfico multicast fluya desde la fuente a los `receivers`. 

La figura 13-2 muestra un ejemplo en el que cinco estaciones de trabajo visualizan el mismo vídeo, transmitido por un servidor mediante tráfico unicast (uno a uno). Cada flecha representa un flujo de datos del mismo vídeo dirigido a cinco dispositivos diferentes. Si cada flujo es de 10 Mbps, el enlace de red entre R1 y R2 necesita un ancho de banda de 50 Mbps. El enlace entre R2 y R4 requiere 30 Mbps, y el enlace entre R2 y R5, 20 Mbps. El servidor debe mantener la información de estado de sesión para todas las conexiones con los dispositivos. El ancho de banda y la carga del servidor aumentan a medida que más receivers solicitan el mismo flujo de vídeo.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%201.PNG)
![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%202.PNG)

Otra alternativa para que las cinco estaciones de trabajo reciban el vídeo es enviarlo desde el servidor mediante transmisión por Broadcast (uno a todos). La figura 13-3 muestra un ejemplo de cómo se transmite el mismo flujo de vídeo mediante difusión IP dirigida. La carga del servidor se reduce, ya que solo tiene que gestionar una única sesión en lugar de varias. Este flujo de vídeo consume solo 10 Mbps de ancho de banda en todos los enlaces de red. Sin embargo, este método presenta algunas desventajas:
- La función de Broadcast IP dirigida no está activada por defecto en los routers Cisco, y su activación expone el router a ataques de denegación de servicio distribuida (DDoS).
- Las tarjetas de red de las estaciones de trabajo que no reciben el vídeo deben procesar los paquetes de difusión y enviarlos a la CPU, lo que consume recursos del procesador. En la figura 13-3, la estación de trabajo F está procesando paquetes no deseados.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%203.PNG)

Por estas razones, el tráfico de difusión no suele ser recomendable. El tráfico multicast permite la comunicación de (uno a muchos), donde solo se envía un paquete de datos por enlace y, a continuación, se replica a través de los enlaces cuando los datos se bifurcan en un dispositivo de red a lo largo del `multicast distribution tree (MDT)`. Estos paquetes de datos constituyen un flujo que utiliza una dirección IP de destino especial, conocida como _group address_. Un servidor de flujo gestiona una única sesión, y los dispositivos de red solicitan selectivamente recibir dicho flujo. Los dispositivos receivers de un flujo multicast se denominan _receivers_. Entre las aplicaciones comunes que utilizan el tráfico multicast se incluyen Cisco TelePresence, vídeo en tiempo real, IPTV, cotizaciones bursátiles en directo, educación a distancia, videoconferencias, música de espera y juegos en línea. La figura 13-4 muestra un ejemplo del mismo flujo de vídeo utilizando multicast. Cada enlace de red consume solo 10 Mbps de ancho de banda, igual que con el tráfico de difusión, pero solo los receivers interesados ​​en el flujo de vídeo procesan el tráfico multicast. Por ejemplo, la estación de trabajo F descartaría el tráfico multicast a nivel de la tarjeta de red, ya que no estaría configurada para recibirlo.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%204.PNG)

> [!NOTE]
> La estación de trabajo F no recibiría ningún tráfico multicast si el conmutador del segmento de red tuviera activada la función de inspección de protocolo IGMP (Internet Group Management Protocol). El protocolo IGMP y la inspección (snooping) IGMP se explican en la siguiente sección.

# Direccionamiento multicast: 

La Autoridad de Asignación de Números de Internet (IANA) asignó el espacio de direcciones IP de clase D 224.0.0.0/4 para la comunicación multicast; este espacio abarca las direcciones desde 224.0.0.0 hasta 239.255.255.255. Los cuatro primeros bits de este rango comienzan con 1110. Dentro de este espacio de direcciones multicast, varios bloques de direcciones están reservados para propósitos específicos, como se muestra en la Tabla 13-2.

Tabla 13-2: Direcciones IP de difusión múltiple asignadas por la IANA
Designacion|Direcciones de Rango Multicast
:---|:---
Local network control block| 224.0.0.0 to 224.0.0.255
Internetwork control block| 224.0.1.0 to 224.0.1.255
Ad hoc block I| 224.0.2.0 to 224.0.255.255
Reserved| 224.1.0.0 to 224.1.255.255
SDP/SAP block| 224.2.0.0 to 224.2.255.255
Ad hoc block II| 224.3.0.0 to 224.4.255.255
Reserved| 224.5.0.0 to 224.251.255.255
DIS transient groups| 224.252.0.0 to 224.255.255.255
Reserved| 225.0.0.0 to 231.255.255.255
Source Specific Multicast (SSM) block| 232.0.0.0 to 232.255.255.255
GLOP block| 233.0.0.0 to 233.251.255.255
Ad hoc block III| 233.252.0.0 to 233.255.255.255
Unicast-Prefix-based IPv4 multicast addresses| 234.0.0.0 to 234.255.255.255
Reserved| 235.0.0.0 to 238.255.255.255
Organization-local scope (commonly known as the Administratively scoped block)|239.0.0.0 to 239.255.255.255

De los bloques de direcciones multicast mencionados en la Tabla 13-2, los más importantes son los siguientes:
- Bloque de control de red local (224.0.0.0/24): Las direcciones de este bloque se utilizan para el tráfico de control de protocolo que no se reenvía fuera del dominio de broadcast. Ejemplos de este tipo de tráfico de control multicast son: todas las máquinas de la subred (224.0.0.1), todos los routers de la subred (224.0.0.2) y todos los routers PIM (224.0.0.13).
- Bloque de control de red global (224.0.1.0/24): Las direcciones de este bloque se utilizan para el tráfico de control de protocolo que puede reenviarse a través de Internet. Ejemplos: Protocolo de Tiempo de Red (NTP) (224.0.1.1), Cisco-RP-Announce (224.0.1.39) y Cisco-RP-Discovery (224.0.1.40).

La ​​Tabla 13-3 muestra algunas de las direcciones multicast más comunes de los bloques de control de red local y global.
Direccion IP MULTICAST|Descripcion
:---|:---
224.0.0.0| Base address (reserved)
224.0.0.1| All hosts in this subnet (all-hosts group)
224.0.0.2| All routers in this subnet
224.0.0.5| All OSPF routers (AllSPFRouters)
224.0.0.6| All OSPF DRs (AllDRouters)
224.0.0.9| All RIPv2 routers
224.0.0.10| All EIGRP routers
224.0.0.13| All PIM routers
224.0.0.18| VRRP
224.0.0.22| IGMPv3
224.0.0.102| HSRPv2 and GLBP
224.0.1.1| NTP
224.0.1.39| Cisco-RP-Announce (Auto-RP)
224.0.1.40| Cisco-RP-Discovery (Auto-RP)

- Source Specific Multicast (SSM) Bloque (232.0.0.0/8): Es el bloque default de SSM. SSM es una extensión de PIM, descrita en la RFC 4607. SSM reenvía el tráfico a los receivers únicamente desde las fuentes multicast para las que los receivers han expresado interés explícito; está diseñado principalmente para aplicaciones de transmisión a múltiples destinatarios.
- Bloque GLOP (233.0.0.0/8): Las direcciones de este bloque son direcciones estáticas de ámbito global. Se asignan a dominios con un número de sistema autónomo (ASN) de 16 bits, mediante la asignación del ASN del dominio (expresado en octetos como X.Y) a los dos octetos centrales del bloque GLOP, resultando una dirección del tipo 233.X.Y.0/24. Esta asignación se define en la RFC 3180. Los dominios con un ASN de 32 bits pueden solicitar espacio en el bloque III o considerar el uso de direcciones multicast IPv6.
- Ámbito local de organización (239.0.0.0/8): Estas direcciones, descritas en la RFC 2365, están limitadas a un grupo u organización local. Son similares a los rangos de direcciones IP unicast reservados (como 10.0.0.0/8) definidos en la RFC 1918 y la IANA no las asignará a otros grupos o protocolos. En otras palabras, los administradores de red pueden usar direcciones multicast en este rango dentro de su dominio sin temor a conflictos con otros usuarios de Internet.

## Direcciones multicast de capa 2:

Históricamente, las NIC de un segmento de LAN podían recibir solo paquetes dirigidos a su dirección MAC o a la dirección MAC de difusión. Este método podía sobrecargar los recursos de enrutamiento durante la replicación de paquetes en segmentos de LAN. Para solucionar este problema, se creó un método alternativo para el tráfico multicast, que permitía replicar este tráfico sin manipular los paquetes y utilizando una dirección MAC de destino estandarizada. 

Una dirección MAC es un valor único asociado a una NIC, que sirve para identificarla de forma exclusiva en un segmento de LAN. Las direcciones MAC son números hexadecimales de 12 dígitos (48 bits), generalmente representados en segmentos de 8 bits separados por guiones (-) o dos puntos (:) (por ejemplo, `00-12-34-56-78-00` o `00:12:34:56:78:00`). 

Cada dirección IP de grupo multicast se asigna a una dirección MAC específica, que permite a las interfaces Ethernet identificar los paquetes multicast dirigidos a ese grupo. Un segmento de LAN puede tener varios flujos de tráfico, y el receiver determina qué tráfico enviar a la CPU para su procesamiento según la dirección MAC asignada al tráfico multicast. 

Los 24 primeros bits de una dirección MAC multicast siempre comienzan con `01:00:5E`. El bit menos significativo del primer byte (bit I/G o bit unicast/multicast) indica si el paquete es multicast (valor 1). El bit 25 siempre es 0. Los 23 bits menos significativos de la dirección MAC multicast se copian de los 23 bits menos significativos de la dirección IP del grupo multicast. 

La figura 13-5 muestra un ejemplo de la asignación de la dirección IP multicast `239.255.1.1` a la dirección MAC multicast `01:00:5E:7F:01:01`. Los 25 bits más significativos son fijos, y los 23 bits menos significativos se copian directamente de la dirección IP del grupo multicast.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%205.PNG)

De los 9 bits de la dirección IP multicast que no se copian en la dirección MAC multicast, los 4 bits más significativos (1110) son fijos; esto deja 5 bits variables que no se transfieren a la dirección MAC. Por lo tanto, existen 32 (2⁵) direcciones IP multicast que no son únicas y pueden corresponder a una misma dirección MAC; es decir, se superponen. La figura 13-6 muestra un ejemplo de dos direcciones IP multicast que se superponen porque se corresponden con la misma dirección MAC multicast.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%206.PNG)

Cuando un dispositivo receiver desea recibir una transmisión multicast específica, envía un mensaje IGMP de **join** al grupo, utilizando la dirección IP del grupo multicast correspondiente. El receiver reprograma su interfaz para aceptar la dirección MAC del grupo multicast asociada a dicha dirección IP. Por ejemplo, un PC podría enviar un mensaje IGMP de **join** al grupo 239.255.1.1 y reprogramaría su tarjeta de red para recibir la dirección MAC `01:00:5E:7F:01:01`. Si el PC recibiera una actualización OSPF dirigida a la dirección `224.0.0.5` y su dirección MAC multicast correspondiente `01:00:5E:00:00:05`, ignoraría dicha actualización, evitando así el procesamiento de tráfico multicast no deseado y optimizando el rendimiento de la CPU.

# Protocolo de gestión de grupos de Internet (IGMP): 

`Internet Group Management Protocol (IGMP)` es el protocolo que utilizan los dispositivos `receivers` para unirse a grupos de difusión y comenzar a recibir tráfico de esos grupos. Este protocolo debe estar habilitado tanto en los dispositivos receivers como en las interfaces de los routers que están conectados a ellos. Cuando un dispositivo receiver desea recibir tráfico de difusión de una fuente, envía una solicitud de **join** IGMP a su router. Si el router no tiene IGMP habilitado en la interfaz, la solicitud se ignora. Existen tres versiones de IGMP. El RFC 1112 define IGMPv1, una versión antigua que se usa con poca frecuencia. El RFC 2236 define IGMPv2, que es común en la mayoría de las redes de difusión, y el RFC 3376 define IGMPv3, utilizado por SSM. En este capítulo se describen únicamente las versiones IGMPv2 e IGMPv3.

## IGMPv2:

IGMPv2 Utiliza el formato de mensaje que se muestra en la figura 13-7. Este mensaje se encapsula en un `paquete IP con el número de protocolo 2`. Los mensajes se envían con la opción de alerta de router IP activada, lo que indica que los paquetes deben ser examinados con mayor detenimiento, y `con un valor de tiempo de vida (TTL) de 1`. El TTL es un campo de 8 bits en la cabecera del paquete IP, establecido por el emisor y decrementado por cada router a lo largo de la ruta hasta su destino. Si el TTL llega a 0 antes de alcanzar el destino, el paquete se descarta. Los paquetes IGMP no deben reenviarse más allá del segmento de red **local**, y por ello se envían con un TTL de 1. Esto garantiza que los paquetes IGMP sean procesados ​​únicamente por el/los router(s) locales y no sean reenviados a otros routers.

![Image Alt]()

Los campos del formato de los mensajes IGMP se definen de la siguiente manera:
- Tipo: Este campo describe cinco tipos de mensajes IGMP utilizados por los routers y los receivers:
  - Version 2 membership report (valor 0x16): Este tipo de mensaje, también conocido como solicitud de **join** a IGMP, permite a los receivers unirse a un grupo multicast o responder a una consulta de membership del router local.
  - Version 1 membership report (valor 0x12): Utilizado por los receivers para la compatibilidad con IGMPv1.
  - Version 2 leave group (valor 0x17): Permite a los receivers indicar que desean dejar de recibir tráfico multicast de un grupo al que estaban suscritos.
  - General membership query (valor 0x11): Se envía periódicamente a la dirección de grupo 224.0.0.1 para comprobar si hay receivers en la subred. El campo de dirección de grupo se establece en 0.0.0.0.
  - Group specific query (valor 0x11): Se envía en respuesta a un mensaje de salida de grupo, a la dirección del grupo que el receiver solicitó abandonar. La dirección de grupo es la dirección IP de destino del paquete IP.
- Tiempo máximo de respuesta: Este campo solo se establece en las consultas de membership a grupo (valor 0x11) y especifica el tiempo máximo permitido antes de enviar una respuesta, en décimas de segundo. En otros mensajes, el emisor lo establece en 0x00 y los receivers lo ignoran.
- Checksum: Este campo es el complemento a 1s de 16 bits de la suma de los complementos a 1s del mensaje IGMP. Es el algoritmo estándar de suma de comprobación de TCP/IP.
- Dirección de grupo: En los mensajes de consulta generales, este campo se establece en 0.0.0.0, mientras que en los mensajes específicos de grupo se establece con la dirección del grupo correspondiente. En los mensajes de informe de membresía, este campo contiene la dirección del grupo al que se refiere el informe; en los mensajes de salida de grupo, contiene la dirección del grupo del que se está saliendo.

Cuando un dispositivo receiver desea recibir una transmisión multicast, envía un informe no solicitado de memebresia (comúnmente conocido como solicitud IGMP join) al router local, indicando el grupo al que desea unirse (por ejemplo, 239.1.1.1). A continuación, el router local envía un mensaje PIM de **join** hacia el origen para solicitar la transmisión multicast. Una vez que el router local comienza a recibir la transmisión, la reenvía a la subred donde se encuentra el dispositivo receiver que la solicitó.

> [!NOTE]
> El término "IGMP join" no es un tipo de mensaje válido según las especificaciones de la RFC de IGMP, pero se utiliza comúnmente en la práctica como sinónimo de "IGMP membership report" porque es más sencillo de pronunciar y escribir.

A continuación, el router comienza a enviar periódicamente mensajes de consulta de membership a la subred, dirigidos a la dirección de grupo 224.0.0.1 (dirección de todos los hosts), para comprobar si hay algún miembro en la subred. Este mensaje de consulta contiene un campo de tiempo máximo de respuesta, que por defecto se establece en 10 segundos.

En respuesta a esta consulta, los receivers configuran un temporizador aleatorio interno entre 0 y 10 segundos (este valor puede cambiar si el tiempo máximo de respuesta no es el predeterminado). Cuando el temporizador expira, los receivers envían informes de membership para cada grupo al que pertenecen. Si un receiver recibe un informe de otro receiver para un grupo al que también pertenece mientras su temporizador está en funcionamiento, detiene su temporizador para ese grupo y no envía ningún informe; esto evita la duplicación de informes.

Cuando un receiver desea abandonar un grupo, si fue el último en responder a una consulta, envía un mensaje de abandono de grupo a la dirección de grupo 222.0.0.2 (dirección de todos los routers). De lo contrario, puede abandonar el grupo sin enviar ningún mensaje, ya que debe haber otro receiver en la subred.

Cuando el router recibe el mensaje de abandono de grupo, envía una consulta de membership específica del grupo a la dirección multicast del grupo para determinar si quedan receivers interesados ​​en ese grupo en la subred. Si no quedan receivers, el router elimina el estado IGMP para ese grupo.

Si hay más de un router en un segmento de LAN, se realiza una elección de router IGMP para determinar cuál será el router encargado de las consultas. Los routers IGMPv2 envían mensajes de consulta de membership con su dirección IP de interfaz como dirección IP de origen y dirigidos a la dirección multicast 224.0.0.1. Cuando un router IGMPv2 recibe este mensaje, comprueba la dirección IP de origen y la compara con su propia dirección IP de interfaz. El router con la dirección IP de interfaz más baja en la subred LAN es elegido como router de consulta IGMP. A partir de ese momento, todos los routers que no sean el router de consulta inician un temporizador que se reinicia cada vez que reciben un informe de consulta de membership del router de consulta.

Si el router de consulta deja de enviar consultas de membership por algún motivo (por ejemplo, si se apaga), se realiza una nueva elección de router de consulta. Un router que no actúa como agente de consulta espera el doble del intervalo de consulta (que por defecto es de 60 segundos) y, si no recibe ninguna consulta del agente de consulta IGMP, inicia el proceso de elección del nuevo agente de consulta IGMP.

## IGMPv3:

En IGMPv2, cuando un receiver envía un informe de membership para unirse a un grupo multicast, no especifica la fuente de la que desea recibir el tráfico multicast. IGMPv3 es una extensión de IGMPv2 que añade soporte para el filtrado de fuentes multicast, lo que permite a los receivers seleccionar la fuente de la que desean recibir el tráfico. IGMPv3 está diseñado para coexistir con IGMPv1 e IGMPv2. Soporta todos los tipos de mensajes IGMP de IGMPv2 y es compatible con versiones anteriores. Las diferencias con IGMPv2 radican en que IGMPv3 añadió nuevos campos a la solicitud de membership y un nuevo tipo de mensaje, el informe de membership versión 3, para implementar el filtrado de fuentes. IGMPv3 permite a las aplicaciones especificar las fuentes de las que desean recibir tráfico. Con IGMPv3, los receivers notifican su membership a una dirección de grupo multicast mediante un informe de membership en dos modos:

- Modo de inclusión: El receiver anuncia su membership a una dirección de grupo multicast y proporciona una lista de direcciones de origen (lista de inclusión) de las que desea recibir tráfico.
- Modo de exclusión: El receiver anuncia su membership a una dirección de grupo multicast y proporciona una lista de direcciones de origen (lista de exclusión) de las que no desea recibir tráfico. El receiver recibirá tráfico solo de las fuentes cuyas direcciones IP no estén en la lista de exclusión. Para recibir tráfico de todas las fuentes (comportamiento de IGMPv2), el receiver utiliza el modo de exclusión con una lista de exclusión vacía.

> [!NOTE]
> IGMPv3 se utiliza para proporcionar filtrado de origen para el Source Specific Multicast (SSM).

## IGMP Snooping:

Para optimizar el reenvío de paquetes y minimizar la propagación de tráfico innecesario, los conmutadores necesitan un método para enviar el tráfico únicamente a los receivers interesados. En el caso del tráfico unicast, los conmutadores Cisco aprenden las direcciones MAC de capa 2 y a qué puertos están asociadas analizando la dirección MAC de origen; esta información se almacena en la tabla de direcciones MAC. Si reciben un paquete de capa 2 con una dirección MAC de destino que no está en esta tabla, lo consideran un paquete unicast desconocido y lo envían a todos los puertos de la misma VLAN, excepto al puerto por el que lo recibieron. Las estaciones de trabajo que no estén interesadas en ese paquete, al detectar que la dirección MAC de destino no les pertenece, lo descartarán. 

En la figura 13-8, el conmutador SW1 comienza con una tabla de direcciones MAC vacía. Cuando la estación de trabajo A envía un paquete, el conmutador almacena su dirección MAC de origen y la interfaz en la tabla de direcciones MAC, y envía el paquete recibido a todos los puertos (excepto al puerto por el que lo recibió).

![Image Alt]()

Si otra estación de trabajo envía un paquete con la dirección MAC de la estación A como destino, dicho paquete ya no se retransmite a todas las estaciones, ya que su dirección MAC está registrada en la tabla de direcciones MAC, y se envía únicamente a la estación A, como se muestra en la figura 13-9.

![Image Alt]()

En el caso del tráfico multicast, una dirección MAC multicast nunca se utiliza como dirección MAC de origen. Por defecto, los switches consideran las direcciones MAC multicast como paquetes multicast desconocidos y los envían a todos los puertos. Luego, corresponde a las estaciones de trabajo seleccionar qué paquetes procesar y cuáles descartar. Las estaciones de trabajo que no están configuradas para recibir tráfico multicast lo descartan a nivel de la tarjeta de red, ya que no están programadas para ello. El envío indiscriminado de tráfico multicast por un switch genera un uso ineficiente del ancho de banda en cada segmento de red.

Los switches Cisco ofrecen dos métodos para reducir el tráfico multicast en un segmento de red:

- IGMP snooping 
- Entradas estáticas de direcciones MAC

El protocolo IGMP Snooping, definido en la RFC 4541, es el método más utilizado. Funciona analizando los mensajes IGMP de solicitud de grupo enviados por los receivers y manteniendo una tabla de asignación entre las interfaces y los grupos IGMP. Cuando el conmutador recibe un paquete multicast dirigido a un grupo específico, lo reenvía únicamente por los puertos que han recibido solicitudes de pertenencia a dicho grupo. La figura 13-10 muestra cómo las estaciones de trabajo A y C envían solicitudes de pertenencia al grupo `239.255.1.1`, que corresponde a la dirección MAC multicast `01:00:5E:7F:01:01` (véase la figura 13-5 para un ejemplo de la asignación de direcciones IP multicast a direcciones MAC). El conmutador 1 tiene habilitado IGMP Snooping y actualiza su tabla de direcciones MAC con esta información.

> [!NOTE]
> Incluso con la función de supervisión IGMP activada, algunos grupos de difusión siguen transmitiéndose en todos los puertos (por ejemplo, las direcciones reservadas 224.0.0.0/24).

![Image Alt]()

La figura 13-11 ilustra cómo el origen envía tráfico a la dirección `239.255.1.1` `(01:00:5E:7F:01:01)`. El conmutador 1 recibe este tráfico y lo reenvía únicamente por las interfaces G0/0 y G0/2, ya que son los únicos puertos que recibieron solicitudes de membership IGMP para ese grupo. Si bien es posible configurar manualmente una entrada estática para multicast en la tabla de direcciones MAC, esta solución no es escalable, ya que no permite adaptarse a los cambios dinámicos; por lo tanto, no se recomienda su uso.

![Image Alt]()

# Protocolo multicast independiente (PIM): 

Los receivers utilizan IGMP para unirse a un grupo de difusión, lo cual resulta suficiente si la fuente del grupo está conectada al mismo enrutador que el receiver. Para que los enrutadores puedan localizar y solicitar flujos de datos de difusión a otros enrutadores, es necesario un protocolo de enrutamiento multicast que dirija el tráfico de difusión a través de la red. Existen varios protocolos de enrutamiento multicast, pero Cisco solo ofrece soporte completo para PIM (Protocol Independent Multicast), que es el más popular y un estándar de la industria definido en la RFC 4601.

PIM es un protocolo de enrutamiento multicast que dirige el tráfico de difusión entre segmentos de red. Puede utilizar cualquier protocolo de enrutamiento unicast para determinar la ruta entre la fuente y los receivers.

## PIM Distribution Trees

Los routers multicast crean árboles de distribución que definen la ruta que sigue el tráfico multicast IP a través de la red para llegar a los receivers. Los dos tipos principales de multicast distribution trees son conocidos como **source trees**, también conocidos como **shortest path trees** (SPTs), y los **shared trees**.

### Source tree:

Un _source tree_ es un árbol de distribución multicast en el que el origen actúa como raíz, y las ramas forman una red de distribución que llega hasta los receivers.

Una vez creado este árbol, utiliza la ruta más corta a través de la red desde la fuente hasta las hojas del árbol; por esta razón, también se le denomina shortest path tree (SPT). El estado de reenvío del SPT se representa mediante la notación (S,G), donde S representa la fuente del flujo multicast (servidor) y G la dirección del grupo multicast. Usando esta notación, el estado del SPT en el ejemplo de la figura 13-12 es (10.1.1.2, 239.1.1.1), donde la fuente multicast S es 10.1.1.2 y el grupo multicast G es 239.1.1.1, al que se unen los receivers A y B.

![Image Alt]()

Dado que cada árbol SPT tiene su origen en la fuente S, cada fuente que envía datos a un grupo de difusión requiere un árbol SPT.

### Shared tree:

Un Shared tree es un árbol de distribución multicast en el que la raíz no es la fuente, sino un router designado como **punto de encuentro RP (rendezvous point)**. Por este motivo, estos árboles también se denominan árboles de punto de encuentro (**rendezvous point trees RPTs**). El tráfico multicast se reenvía a través del shared tree de distribución según la dirección de grupo G a la que están dirigidos los paquetes, independientemente de la dirección de origen. Por este motivo, el estado de reenvío en el árbol de distribución compartido se representa mediante la notación (*,G), que se lee como “asterisco, coma, G”. La figura 13-13 muestra un árbol de distribución compartido donde R2 es el RP, y la dirección (*,G) es (*,239.1.1.1).

> [!NOTE]
> En el any-source multicast (ASM), el estado (S,G) requiere un nodo padre (*,G). Por esta razón, la figura 13-13 muestra que los routers R1 y R2 tienen el estado (*,G). En cambio, R3 y R4 aún no se han unido al árbol de origen, como se indica por la ausencia de la entrada (S,G) en su tabla de enrutamiento multicast.

![Image Alt]()

Una de las ventajas de los **shared trees** de distribución frente a los **source trees** es que requieren menos entradas de multidifusión (por ejemplo, S,G y *,G). Por ejemplo, al añadir más fuentes a la red que envían tráfico al mismo grupo de multidifusión, el número de entradas de multidifusión para R3 y R4 permanece invariable: (*,239.1.1.1).

El principal inconveniente de los árboles de distribución compartida es que los receivers reciben tráfico de todas las fuentes que envían datos al mismo grupo de multidifusión. Si bien las aplicaciones de los receivers pueden filtrar el tráfico no deseado, esto genera un tráfico de red innecesario, desperdiciando ancho de banda. Además, dado que los árboles de distribución compartida permiten múltiples fuentes en un grupo de multidifusión IP, existe un riesgo de seguridad, ya que fuentes no autorizadas podrían enviar paquetes no deseados a los receivers.

## Terminología de PIM

La figura 13-14 muestra una topología de referencia para ilustrar algunos conceptos de enrutamiento multicast.

![Image Alt]()

La siguiente lista define la terminología común de PIM, ilustrada en la Figura 13-14:

- Interfaz de reenvío de ruta inversa RPF (Reverse Path Forwarding): La interfaz con la ruta de menor coste (basada en la distancia administrativa [AD] y la métrica) hacia la dirección IP del origen (SPT) o del punto de encuentro (RP), en el caso de árboles de distribución. Si varias interfaces tienen el mismo coste, se selecciona como criterio de desempate la interfaz con la dirección IP más alta. Un ejemplo de este tipo de interfaz es Te0/1/2 en R5, ya que representa la ruta más corta hacia el origen. Otro ejemplo es Te1/1/1 en R7, puesto que la ruta más corta hacia el origen pasa por R4.
- Vecino RPF (RPF neighbor): El vecino PIM en la interfaz RPF. Por ejemplo, si R7 utiliza el shared tree RPT, el vecino RPF sería R3, que representa el camino de menor costo al RP. Si utiliza el árbol SPT, R4 sería su vecino RPF, ya que ofrece el menor costo a la fuente.
- Upstream: Hacia la fuente del árbol, que puede ser la fuente real en árboles basados ​​en fuente o el RP en shared trees. Un mensaje de adhesión PIM se transmite en sentido ascendente hacia la fuente.
- Upstream interface: La interfaz que apunta hacia la fuente del árbol. También se conoce como interfaz RPF o interfaz de entrada (IIF). Un ejemplo de interfaz ascendente es la interfaz Te0/1/2 de R5, que puede enviar mensajes de adhesión PIM en sentido ascendente a su vecino RPF.
- Downstream: Alejándose de la fuente del árbol y hacia los receivers.
- Downstream interface: Cualquier interfaz que se utiliza para reenviar tráfico multicast a lo largo del árbol; también conocida como interfaz de salida (OIF). Un ejemplo de interfaz descendente es la interfaz Te0/0/0 de R1, que reenvía el tráfico multicast a la interfaz Te0/0/1 de R3.
- Interfaz de entrada (IIF Incoming interface): El único tipo de interfaz que puede recibir tráfico multicast proveniente de la fuente, lo cual es lo mismo que la interfaz RPF. Un ejemplo de este tipo de interfaz es Te0/0/1 en R3, ya que el camino más corto a la fuente se conoce a través de esta interfaz.
- Interfaz de salida (OIF Outgoing interface): Cualquier interfaz que se utiliza para reenviar tráfico multicast a lo largo del árbol; también conocida como interfaz descendente.
- Lista de interfaces de salida (OIL Outgoing interface list): Un grupo de OIF que reenvían tráfico multicast al mismo grupo. Un ejemplo de esto son las interfaces Te0/0/0 y Te0/0/1 de R1 que envían tráfico multicast a R3 y R4 para el mismo grupo multicast.
- Enrutador de última etapa (LHR Last-hop router): Un enrutador conectado directamente a los receivers; también conocido como enrutador **leaf**. Es responsable de enviar mensajes de adhesión PIM en sentido ascendente hacia el RP o la fuente.
- Router de primer salto (FHR First-hop router): Un router conectado directamente a la fuente, también conocido como router raíz. Es responsable de enviar los mensajes de registro al RP.
- Base de datos de información de enrutamiento multicast (MRIB Multicast Routing Information Base): Una tabla de topología, también conocida como tabla de rutas multicast (mroute). Se crea a partir de la información de la tabla de enrutamiento unicast y del protocolo PIM. La MRIB contiene la fuente (S), el grupo (G), las interfaces de entrada (IIF), las interfaces de salida (OIF) y la información del vecino RPF para cada ruta multicast, así como otra información relacionada con el enrutamiento multicast.
- Base de datos de información de reenvío multicast (MFIB Multicast Forwarding Information Base): Una tabla de reenvío que utiliza la MRIB para programar la información de reenvío multicast en hardware, lo que permite un procesamiento más rápido.
- Estado de multicast: El estado de reenvío de tráfico multicast que utiliza un router para reenviar el tráfico multicast. Este estado está compuesto por las entradas de la tabla mroute (S, G, IIF, OIF, etc.).

# Puntos de encuentro: Esta sección describe el propósito, la función y el funcionamiento de los puntos de encuentro en una red multicast.


