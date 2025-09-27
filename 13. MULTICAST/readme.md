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

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/87bc6da98b3dbc414b074b77522a1a73382844eb/13.%20MULTICAST/IMG/MULTI%20IGMP/MULTI%20IGMP%201.PNG)

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

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/87bc6da98b3dbc414b074b77522a1a73382844eb/13.%20MULTICAST/IMG/MULTI%20IGMP/MULTI%20IGMP%202.PNG)

Si otra estación de trabajo envía un paquete con la dirección MAC de la estación A como destino, dicho paquete ya no se retransmite a todas las estaciones, ya que su dirección MAC está registrada en la tabla de direcciones MAC, y se envía únicamente a la estación A, como se muestra en la figura 13-9.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/87bc6da98b3dbc414b074b77522a1a73382844eb/13.%20MULTICAST/IMG/MULTI%20IGMP/MULTI%20IGMP%203.PNG)

En el caso del tráfico multicast, una dirección MAC multicast nunca se utiliza como dirección MAC de origen. Por defecto, los switches consideran las direcciones MAC multicast como paquetes multicast desconocidos y los envían a todos los puertos. Luego, corresponde a las estaciones de trabajo seleccionar qué paquetes procesar y cuáles descartar. Las estaciones de trabajo que no están configuradas para recibir tráfico multicast lo descartan a nivel de la tarjeta de red, ya que no están programadas para ello. El envío indiscriminado de tráfico multicast por un switch genera un uso ineficiente del ancho de banda en cada segmento de red.

Los switches Cisco ofrecen dos métodos para reducir el tráfico multicast en un segmento de red:

- IGMP snooping 
- Entradas estáticas de direcciones MAC

El protocolo IGMP Snooping, definido en la RFC 4541, es el método más utilizado. Funciona analizando los mensajes IGMP de solicitud de grupo enviados por los receivers y manteniendo una tabla de asignación entre las interfaces y los grupos IGMP. Cuando el conmutador recibe un paquete multicast dirigido a un grupo específico, lo reenvía únicamente por los puertos que han recibido solicitudes de pertenencia a dicho grupo. La figura 13-10 muestra cómo las estaciones de trabajo A y C envían solicitudes de pertenencia al grupo `239.255.1.1`, que corresponde a la dirección MAC multicast `01:00:5E:7F:01:01` (véase la figura 13-5 para un ejemplo de la asignación de direcciones IP multicast a direcciones MAC). El conmutador 1 tiene habilitado IGMP Snooping y actualiza su tabla de direcciones MAC con esta información.

> [!NOTE]
> Incluso con la función de supervisión IGMP activada, algunos grupos de difusión siguen transmitiéndose en todos los puertos (por ejemplo, las direcciones reservadas 224.0.0.0/24).

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/87bc6da98b3dbc414b074b77522a1a73382844eb/13.%20MULTICAST/IMG/MULTI%20IGMP/MULTI%20IGMP%204.PNG)

La figura 13-11 ilustra cómo el origen envía tráfico a la dirección `239.255.1.1` `(01:00:5E:7F:01:01)`. El conmutador 1 recibe este tráfico y lo reenvía únicamente por las interfaces G0/0 y G0/2, ya que son los únicos puertos que recibieron solicitudes de membership IGMP para ese grupo. Si bien es posible configurar manualmente una entrada estática para multicast en la tabla de direcciones MAC, esta solución no es escalable, ya que no permite adaptarse a los cambios dinámicos; por lo tanto, no se recomienda su uso.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/87bc6da98b3dbc414b074b77522a1a73382844eb/13.%20MULTICAST/IMG/MULTI%20IGMP/MULTI%20IGMP%205.PNG)

# Protocolo multicast independiente (PIM): 

Los receivers utilizan IGMP para unirse a un grupo de difusión, lo cual resulta suficiente si la fuente del grupo está conectada al mismo enrutador que el receiver. Para que los enrutadores puedan localizar y solicitar flujos de datos de difusión a otros enrutadores, es necesario un protocolo de enrutamiento multicast que dirija el tráfico de difusión a través de la red. Existen varios protocolos de enrutamiento multicast, pero Cisco solo ofrece soporte completo para PIM (Protocol Independent Multicast), que es el más popular y un estándar de la industria definido en la RFC 4601.

PIM es un protocolo de enrutamiento multicast que dirige el tráfico de difusión entre segmentos de red. Puede utilizar cualquier protocolo de enrutamiento unicast para determinar la ruta entre la fuente y los receivers.

## PIM Distribution Trees

Los routers multicast crean árboles de distribución que definen la ruta que sigue el tráfico multicast IP a través de la red para llegar a los receivers. Los dos tipos principales de multicast distribution trees son conocidos como **source trees**, también conocidos como **shortest path trees** (SPTs), y los **shared trees**.

### Source tree:

Un _source tree_ es un árbol de distribución multicast en el que el origen actúa como raíz, y las ramas forman una red de distribución que llega hasta los receivers.

Una vez creado este árbol, utiliza la ruta más corta a través de la red desde la fuente hasta las hojas del árbol; por esta razón, también se le denomina shortest path tree (SPT). El estado de reenvío del SPT se representa mediante la notación (S,G), donde S representa la fuente del flujo multicast (servidor) y G la dirección del grupo multicast. Usando esta notación, el estado del SPT en el ejemplo de la figura 13-12 es (10.1.1.2, 239.1.1.1), donde la fuente multicast S es 10.1.1.2 y el grupo multicast G es 239.1.1.1, al que se unen los receivers A y B.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/87bc6da98b3dbc414b074b77522a1a73382844eb/13.%20MULTICAST/IMG/MULTI%20IGMP/MULTI%20IGMP%206.PNG)

Dado que cada árbol SPT tiene su origen en la fuente S, cada fuente que envía datos a un grupo de difusión requiere un árbol SPT.

### Shared tree:

Un Shared tree es un árbol de distribución multicast en el que la raíz no es la fuente, sino un router designado como **punto de encuentro RP (rendezvous point)**. Por este motivo, estos árboles también se denominan árboles de punto de encuentro (**rendezvous point trees RPTs**). El tráfico multicast se reenvía a través del shared tree de distribución según la dirección de grupo G a la que están dirigidos los paquetes, independientemente de la dirección de origen. Por este motivo, el estado de reenvío en el árbol de distribución compartido se representa mediante la notación (*,G), que se lee como “asterisco, coma, G”. La figura 13-13 muestra un árbol de distribución compartido donde R2 es el RP, y la dirección (*,G) es (*,239.1.1.1).

> [!NOTE]
> En el any-source multicast (ASM), el estado (S,G) requiere un nodo padre (*,G). Por esta razón, la figura 13-13 muestra que los routers R1 y R2 tienen el estado (*,G). En cambio, R3 y R4 aún no se han unido al árbol de origen, como se indica por la ausencia de la entrada (S,G) en su tabla de enrutamiento multicast.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/87bc6da98b3dbc414b074b77522a1a73382844eb/13.%20MULTICAST/IMG/MULTI%20IGMP/MULTI%20IGMP%207.PNG)

Una de las ventajas de los **shared trees** de distribución frente a los **source trees** es que requieren menos entradas de multidifusión (por ejemplo, S,G y *,G). Por ejemplo, al añadir más fuentes a la red que envían tráfico al mismo grupo de multidifusión, el número de entradas de multidifusión para R3 y R4 permanece invariable: (*,239.1.1.1).

El principal inconveniente de los árboles de distribución compartida es que los receivers reciben tráfico de todas las fuentes que envían datos al mismo grupo de multidifusión. Si bien las aplicaciones de los receivers pueden filtrar el tráfico no deseado, esto genera un tráfico de red innecesario, desperdiciando ancho de banda. Además, dado que los árboles de distribución compartida permiten múltiples fuentes en un grupo de multidifusión IP, existe un riesgo de seguridad, ya que fuentes no autorizadas podrían enviar paquetes no deseados a los receivers.

## Terminología de PIM

La figura 13-14 muestra una topología de referencia para ilustrar algunos conceptos de enrutamiento multicast.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/87bc6da98b3dbc414b074b77522a1a73382844eb/13.%20MULTICAST/IMG/MULTI%20IGMP/MULTI%20IGMP%208.PNG)

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

Actualmente existen cinco modos de operación de PIM:
- PIM Dense Mode (PIM-DM)
- PIM Sparse Mode (PIM-SM)
- PIM Sparse Dense Mode
- PIM Source Specific Multicast (PIM-SSM)
- PIM Bidirectional Mode (Bidir-PIM)

> [!NOTE]
> PIM-DM y PIM-SM también se conocen comúnmente como multicast de origen arbitrario (ASM).

Todos los mensajes de control de PIM utilizan el número de protocolo IP 103; pueden ser de tipo unicast (es decir, mensajes de registro y de parada de registro enviados con un TTL mayor que 1) o multicast, con un TTL de 1 a la dirección 224.0.0.13, que corresponde a todos los routers PIM. 

La tabla 13-4 muestra la lista de los mensajes de control de PIM.
Tipo|Tipo de mensaje|Destino|PIM protocolo
:---|:---|:---|:---
0| Hola| 224.0.0.13 (todos los routers PIM)| PIM-SM, PIM-DM, Bidir-PIM y SSM
1| Registro| Dirección RP (unicast)| PIM-SM
2| Detener registro| First-hop router (unicast)| PIM-SM
3| Join/prune| 224.0.0.13 |(todos los routers PIM)| PIM-SM, Bidir-PIM y SSM
4| Inicialización/Bootstrap| 224.0.0.13| (todos los routers PIM)| PIM-SM y Bidir-PIM
5| Afirmación| 224.0.0.13 |(todos los routers PIM)| PIM-SM, PIM-DM y Bidir-PIM
8| Anuncio de RP candidato| Dirección del router de inicialización (BSR Bootstrap router) (unicast a BSR)| PIM-SM y Bidir-PIM
9| Actualización de estado| 224.0.0.13 |(todos los routers PIM)| PIM-DM
10| Elección de DF| 224.0.0.13 |(todos los routers PIM)| Bidir-PIM

Por defecto, los mensajes de saludo PIM se envían cada 30 segundos desde cada interfaz habilitada para PIM, con el fin de detectar los routers PIM vecinos en dicha interfaz. Estos mensajes se envían a la dirección de grupo de routers PIM, tal como se muestra en la tabla 13-4. Los mensajes de saludo también se utilizan para elegir un `designated router (DR)`, como se describe más adelante en este capítulo, y para negociar funcionalidades adicionales. Todos los routers PIM deben registrar la información de los mensajes de saludo recibidos de cada router PIM vecino.

## PIM Dense mode (PIM en modo denso): 

Los routers PIM pueden configurarse para el `PIM Dense Mode` (PIM-DM) cuando se puede suponer que los receptores de un grupo de multidifusión están presentes en todas las subredes de la red; es decir, cuando los receptores de multidifusión de un grupo están distribuidos de forma uniforme en toda la red.

En PIM-DM, el árbol de multidifusión se crea mediante la propagación del tráfico a través de todas las interfaces, desde la fuente hasta todos los routers en modo denso de la red. El árbol se construye desde la raíz hacia las hojas. Al recibir el tráfico del grupo de multidifusión, cada router decide si ya tiene receptores activos que desean recibir dicho tráfico. Si los tiene, el router permanece inactivo y permite que el tráfico de multidifusión continúe. Si ningún receptor ha solicitado el flujo de multidifusión para ese grupo en el router de última conexión (LHR), el router envía un mensaje de prune hacia la fuente. De esta forma, esa rama del árbol se elimina para evitar el tráfico innecesario. El árbol resultante es un árbol de origen, ya que es único desde la fuente hasta los receptores. 

La figura 13-15 muestra el funcionamiento de la propagación y prune en modo denso. El tráfico de multidifusión se propaga por toda la red. Al recibir el tráfico de multidifusión de su vecino aguas arriba a través de su interfaz RPF, cada router lo reenvía a todos sus vecinos PIM-DM. Esto puede provocar que parte del tráfico llegue a través de una interfaz que no sea RPF, como ocurre con R3 que recibe tráfico de R2 a través de una interfaz no RPF. Los paquetes que llegan por una interfaz no RPF se descartan. 

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c2bfe3bf9a2ab0a7843b70a88c36ef177ef30348/13.%20MULTICAST/IMG/MULTI%20PIM/MULTI%20PIM%201.PNG)

Estos flujos de multidifusión no RPF son normales durante la propagación inicial del tráfico de multidifusión y se corrigen mediante el mecanismo de prune PIM-DM. Este mecanismo se utiliza para detener el tráfico no deseado. Las señales de prune (indicadas por las flechas discontinuas) se envían a través de la interfaz RPF cuando el enrutador no tiene dispositivos receptores que necesiten el tráfico de multidifusión, como ocurre con R4, que no tiene ningún receptor interesado. También se envían a través de las interfaces no RPF para detener el tráfico de multidifusión que llega por dicha interfaz, como ocurre con R3, donde el tráfico de multidifusión llega a través de una interfaz no RPF desde R2, lo que genera una señal de prune.

La figura 13-16 muestra la topología resultante después de eliminar los enlaces innecesarios. Esto crea un árbol de distribución más corto (SPT) desde la fuente al receptor. Aunque el tráfico de multidifusión ya no llega a la mayoría de los enrutadores de la red, el estado (S,G) se mantiene en todos ellos. Este estado (S,G) persiste hasta que la fuente deja de transmitir.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c2bfe3bf9a2ab0a7843b70a88c36ef177ef30348/13.%20MULTICAST/IMG/MULTI%20PIM/MULTI%20PIM%202.PNG)

En PIM-DM, Prune caducan después de tres minutos. Esto provoca que el tráfico multicast se vuelva a distribuir a todos los routers, tal como ocurrió durante la distribución inicial. Este comportamiento de distribución y filtrado periódico (cada tres minutos) es normal y debe tenerse en cuenta al diseñar una red que utilice PIM-DM. PIM-DM es adecuado para redes pequeñas donde hay receptores activos en cada subred. Dado que este escenario es poco común, PIM-DM no se implementa ampliamente y no se recomienda para entornos de producción.

## Modo disperso de PIM (PIM Sparse Mode):

El protocolo PIM en modo disperso (PIM-SM) fue diseñado para redes con receptores de aplicaciones multicast distribuidos de forma dispersa, es decir, cuando los receptores de un grupo multicast están distribuidos de forma irregular en la red. Sin embargo, PIM-SM también funciona bien en redes con alta densidad de receptores. Este protocolo asume que ningún receptor está interesado en el tráfico multicast a menos que lo solicite explícitamente. Al igual que PIM-DM, PIM-SM utiliza la tabla de enrutamiento unicast para realizar las comprobaciones RPF, y no depende del protocolo de enrutamiento utilizado para poblar dicha tabla (incluidas las rutas estáticas); por lo tanto, es independiente del protocolo.

### PIM Shared y Source Path Trees:

IM-SM utiliza un modelo de incorporación explícita, donde los receptores envían un mensaje IGMP de incorporación a su router local, también conocido como **last hop router (LHR)**. Este mensaje provoca que el LHR envíe un mensaje PIM de incorporación en dirección a la raíz del árbol, que puede ser el RP en el caso de un shared tree (RPT) o el **first hop router (FHR)** al que está conectado el origen que envía el tráfico multicast en el caso de un árbol de origen (SPT).

Como resultado de estas incorporaciones explícitas, se crea un estado de reenvío multicast, lo cual difiere significativamente del comportamiento de inundación y prune, o de la incorporación implícita de PIM-DM, donde el estado de reenvío lo determina el paquete multicast que llega al router.

La figura 13-17 ilustra un origen multicast que envía tráfico multicast al FHR. Este, a su vez, envía dicho tráfico al RP, lo que permite al RP identificar el origen multicast. También muestra un receptor que envía un mensaje IGMP de incorporación al LHR para unirse al grupo multicast. El LHR envía entonces un mensaje PIM de incorporación (\*,G) al RP, creando un árbol compartido entre el RP y el LHR. El RP, a continuación, envía un mensaje PIM de incorporación (S,G) al FHR, formando un árbol de origen entre el origen y el RP. En resumen, se crean dos árboles: un SPT del FHR al RP (S,G) y un árbol compartido del RP al LHR (*,G).

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c2bfe3bf9a2ab0a7843b70a88c36ef177ef30348/13.%20MULTICAST/IMG/MULTI%20PIM/MULTI%20PIM%203.PNG)

En este punto, el tráfico multicast comienza a fluir desde la fuente hacia el RP, y desde el RP hacia el LHR, para finalmente llegar al receptor. Esta es una explicación muy simplificada de cómo PIM-SM realiza el enrutamiento multicast. Las siguientes secciones lo explican con mayor detalle.

### Shared Tree Join:

La figura 13-17 muestra al receptor A, conectado al LHR, uniéndose al grupo de multidifusión G. El LHR conoce la dirección IP del RP para el grupo G y, a continuación, envía un mensaje de solicitud de adhesión PIM (\*,G) a dicho RP. Si el RP no estuviera conectado directamente, esta solicitud PIM (*,G) recorrería los nodos intermedios hasta llegar al RP, creando una rama del árbol de multidifusión que se extendería desde el RP hasta el LHR. A partir de ese momento, el tráfico de multidifusión del grupo G que llegue al RP podrá fluir a través del árbol de multidifusión hasta el receptor.

### Source Registration:

En la figura 13-17, tan pronto como la fuente de un grupo G envía un paquete, el FHR asociado a dicha fuente se encarga de registrarla en el RP y solicitarle que cree un árbol de distribución hacia ese router.

El FHR crea una interfaz de túnel de registro PIM unidireccional que encapsula los datos de multidifusión recibidos de la fuente en un mensaje PIM-SM especial, llamado _register messagge_. Estos datos de multidifusión encapsulados se envían por unicast al RP mediante el túnel de registro PIM.

Cuando el RP recibe un mensaje de registro, desencapsula el paquete de datos de multidifusión. Si no existe un árbol de distribución activo (porque no hay receptores interesados), el RP envía un mensaje de parada de registro por unicast directamente al FHR, sin utilizar el túnel de registro PIM, indicándole que deje de enviar mensajes de registro.

Si existe un árbol de distribución activo para el grupo, el RP reenvía el paquete de multidifusión por dicho árbol y envía un mensaje de adhesión (S,G) hacia la red de origen S para crear un SPT (S,G). Si hay varios routers entre el RP y la fuente, se crea un estado (S,G) en todos los routers del SPT, incluido el RP. También habrá un estado (*,G) en R1 y en todos los routers entre el FHR y el RP.

Una vez creado el SPT desde el router origen al RP, el tráfico de multidifusión fluye directamente desde la fuente S al RP.

Cuando el RP empieza a recibir datos directamente (a través del SPT) de la fuente S, envía un mensaje de parada de registro al FHR para indicarle que puede dejar de enviar los mensajes de registro por unicast. En este punto, el tráfico de multidifusión fluye por el SPT hacia el RP y, desde allí, por el árbol de distribución (RPT) hacia el receptor.

El túnel de registro PIM entre el FHR y el RP permanece activo, incluso sin tráfico de multidifusión, mientras exista una ruta RPF válida para el RP.

### PIM SPT Switchover:

PIM-SM permite al LHR cambiar del árbol compartido (RPT) a un árbol específico (SPT) para una fuente determinada. En los routers de Cisco, este es el comportamiento predeterminado y ocurre inmediatamente después de recibir el primer paquete multicast del RP a través del RPT, incluso si la ruta más corta a la fuente pasa por el RP. La figura 13-18 ilustra este proceso de cambio a SPT. Cuando el LHR recibe el primer paquete multicast del RP, identifica la dirección IP de la fuente. A continuación, consulta su tabla de enrutamiento unicast para determinar la ruta más corta a la fuente y envía un mensaje de adhesión (S,G) PIM a la fuente, paso a paso, hasta llegar al FHR y crear el SPT. Si la interfaz RPF del SPT difiere de la del RPT, el LHR comenzará a recibir tráfico multicast duplicado. En ese momento, cambiará la interfaz RPF a la del SPT y enviará un mensaje de prune (S,G) PIM al RP para detener el tráfico multicast duplicado que llega por el RPT. En la figura 13-18, la ruta más corta a la fuente es entre R1 y R3; si ese enlace estuviera inactivo o no existiera, la ruta más corta sería a través del RP. En este caso, también se produciría el cambio a SPT, aunque la ruta del SPT sea la misma que la del RPT.

> [!NOTE]
> El mecanismo de conmutación de PIM SPT se puede desactivar para todos los grupos o para grupos específicos.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c2bfe3bf9a2ab0a7843b70a88c36ef177ef30348/13.%20MULTICAST/IMG/MULTI%20PIM/MULTI%20PIM%204.PNG)

Si el RP no tiene ninguna otra interfaz interesada en el tráfico multicast, envía un mensaje de prune PIM hacia el FHR. Si existen routers entre el RP y el FHR, este mensaje de prune se transmitirá de router en router hasta llegar al FHR.

### Designated Routers:

Cuando existen varios routers PIM-SM en un segmento de LAN, los mensajes de saludo PIM se utilizan para elegir un _Designated router_ (DR) y evitar el envío de tráfico multicast duplicado a la LAN o al RP. Por defecto, el valor de prioridad del DR en todos los routers PIM es 1, y se puede modificar para forzar a un router específico a convertirse en DR durante el proceso de elección. Cuanto mayor sea la prioridad del DR, mejor. Si un router de la subred no admite la opción de prioridad del DR o si todos los routers tienen la misma prioridad, se utiliza la dirección IP más alta de la subred como criterio de desempate.

En un FHR, el router designado encapsula en mensajes de registro unicast los paquetes multicast originados por una fuente y destinados al RP. En un LHR, el router designado envía mensajes de incorporación y prune PIM al RP para informar sobre la pertenencia a grupos de hosts y realiza la conmutación de árbol SPT.

Sin DR, todos los LHR del mismo segmento de LAN podrían enviar incorporaciones PIM, lo que generaría tráfico multicast duplicado en la LAN. En el origen, si hay varios FHR en la LAN, todos envían mensajes de registro al RP simultáneamente.

El tiempo de retención del DR por defecto es 3,5 veces el intervalo de saludo (105 segundos). Si no se reciben saludos en ese intervalo, se elige un nuevo DR. Para reducir el tiempo de conmutación del DR, se puede reducir el intervalo de consulta de saludo, aunque esto implica un mayor tráfico de control y mayor uso de la CPU del router.

## Reverse Path Forwarding:

El algoritmo de Reenvío de Ruta Inversa (RPF) se utiliza para prevenir bucles y garantizar que el tráfico multicast llegue a la interfaz correcta. Su funcionamiento es el siguiente:

- Si un router recibe un paquete multicast en una interfaz que utiliza para enviar paquetes unicast al origen, se considera que ha llegado por la interfaz RPF.
- Si el paquete llega por la interfaz RPF, el router lo reenvía por las interfaces de la lista de interfaces de salida (OIL) de la entrada de la tabla de enrutamiento multicast.
- Si el paquete no llega por la interfaz RPF, se descarta para evitar bucles.

PIM utiliza árboles de origen multicast entre el origen y el LHR, y entre el origen y el RP. También utiliza árboles compartidos multicast entre el RP y los LHR. El control RPF se realiza de forma diferente en cada caso:

- Si un router PIM tiene una entrada (S,G) en la tabla de enrutamiento multicast (estado SPT), realiza el control RPF con la dirección IP del origen del paquete multicast.
- Si un router PIM no tiene un estado de árbol de origen explícito, se considera un estado de árbol compartido. El router realiza el control RPF con la dirección del RP, que se conoce cuando los miembros se unen al grupo.

PIM-SM utiliza la función de búsqueda RPF para determinar a dónde enviar las solicitudes de adhesión y exclusión. Las solicitudes de adhesión (S,G) (estado SPT) se envían hacia el origen. Las solicitudes de adhesión (*,G) (estado de árbol compartido) se envían hacia el RP.

La topología de la izquierda en la figura 13-19 muestra un fallo en el control RPF en R3 para la entrada (S,G), ya que el paquete llega por una interfaz no RPF. La topología de la derecha muestra el tráfico multicast llegando a la interfaz correcta en R3; luego se reenvía por todas las interfaces de salida.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c2bfe3bf9a2ab0a7843b70a88c36ef177ef30348/13.%20MULTICAST/IMG/MULTI%20PIM/MULTI%20PIM%205.PNG)

## PIM Forwarder

En ciertos escenarios, los paquetes multicast duplicados podrían circular por una red de acceso múltiple. El mecanismo de selección de ruta PIM evita este problema.

La figura 13-20 muestra cómo los routers R2 y R3 reciben el mismo tráfico (S,G) a través de sus interfaces RPF y lo reenvían al segmento de red. Por lo tanto, R2 y R3 reciben un paquete (S,G) a través de su interfaz de salida (OIF), que coincide con la OIF de su entrada (S,G). En otras palabras, detectan un paquete multicast para un grupo (S,G) específico que entra por la misma interfaz de salida por la que también sale. Esto activa el mecanismo de selección de ruta.

R2 y R3 envían mensajes de selección de ruta PIM a la red. Estos mensajes incluyen la distancia administrativa (AD) y la métrica de ruta, que se envían al origen para determinar qué router debe reenviar el tráfico multicast a ese segmento de red.

Cada router compara sus propios valores con los recibidos. Se da preferencia al mensaje PIM con la menor distancia administrativa al origen. En caso de empate, se elige la métrica de ruta más baja; y, como último criterio, se utiliza la dirección IP más alta.

El router perdedor elimina la interfaz como si hubiera recibido una orden de eliminación, y el router ganador se convierte en el router PIM responsable del reenvío en la red.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c2bfe3bf9a2ab0a7843b70a88c36ef177ef30348/13.%20MULTICAST/IMG/MULTI%20PIM/MULTI%20PIM%206.PNG)

> [!NOTE]
> El temporizador de _prune_ expira a los tres minutos en el router perdedor, lo que hace que este router vuelva a reenviar paquetes por la interfaz. Esto desencadena la repetición del proceso de selección. Si el router ganador dejara de estar operativo, el router perdedor asumiría la función de reenviar paquetes a ese segmento de red una vez que expire el temporizador de _prune_.

El concepto de enrutador PIM se aplica tanto a PIM-DM como a PIM-SM. Si bien PIM-DM lo utiliza frecuentemente, PIM-SM rara vez lo requiere, ya que los paquetes duplicados solo llegan a la LAN en caso de inconsistencias en la configuración de enrutamiento.

Con la topología que se muestra en la Figura 13-20, PIM-SM no enviaría flujos duplicados a la LAN, a diferencia de PIM-DM, debido a su funcionamiento. Por ejemplo, suponiendo que R1 es el RP, cuando R4 envía un mensaje de adhesión PIM hacia R1, lo envía a la dirección 224.0.0.13 (dirección de grupo de enrutadores PIM), y R2 y R3 lo reciben. Uno de los campos del mensaje de adhesión PIM incluye la dirección IP del vecino aguas arriba, también conocido como vecino RPF. Si R3 es el vecino RPF, solo R3 enviará un mensaje de adhesión PIM a R1. R2 no lo hará, ya que el mensaje no estaba dirigido a él. En este punto, se crea un árbol de distribución compartido entre R1, R3 y R4, sin que se produzca duplicación de tráfico.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c2bfe3bf9a2ab0a7843b70a88c36ef177ef30348/13.%20MULTICAST/IMG/MULTI%20PIM/MULTI%20PIM%207.PNG)

# Puntos de encuentro (RP rendezvous points): 

En PIM-SM, es obligatorio seleccionar uno o más routers para que funcionen como _rendezvous points_ (RP). Un RP es un nodo raíz común ubicado en un punto específico del árbol de distribución compartido, tal como se describió anteriormente en este capítulo. Un RP puede configurarse de forma estática en cada router o mediante un mecanismo dinámico. Un router PIM puede configurarse para funcionar como RP de forma estática en cada router del dominio de multidifusión, o de forma dinámica mediante la configuración de Auto-RP o un router de arranque PIM (BSR), como se describe en las siguientes secciones.

> [!NOTE]
> BSR y Auto-RP no fueron diseñados para funcionar juntos y su implementación simultánea en la misma red puede generar complejidades innecesarias. Por lo tanto, se recomienda no utilizarlos de forma simultánea.

## Static RP (rendezvous points router): 

Es posible configurar de forma estática el RP para un rango de grupos multicast, configurando la dirección del RP en cada router del dominio multicast. La configuración de un RP estático es relativamente sencilla y se puede realizar con una o dos líneas de configuración en cada router. Si la red no tiene muchos RP definidos o si estos no cambian con frecuencia, este puede ser el método más simple para configurarlos. También puede ser una opción atractiva para redes pequeñas.

Sin embargo, la configuración estática puede aumentar la carga administrativa en redes grandes y complejas. Todos los routers deben tener la misma dirección de RP. Esto significa que cambiar la dirección del RP requiere la reconfiguración de todos los routers. Si hay varios RP activos para diferentes grupos, todos los routers deben conocer qué RP gestiona cada grupo multicast. Para garantizar que esta información sea completa, podrían ser necesarios varios comandos de configuración. Si un RP configurado manualmente falla, no existe un procedimiento de conmutación por error para que otro router asuma la función del RP que ha fallado, y este método por sí solo no proporciona ningún tipo de equilibrio de carga.

## AUTO-RP (AUTO Rendezvous Points router):

Auto-RP es un mecanismo propio de Cisco que automatiza la distribución de la asignación de grupos a los puntos de encuentro (RP) en una red PIM. Sus ventajas son las siguientes:
- Permite utilizar varios puntos de encuentro (RP) en una red para gestionar diferentes rangos de grupos.
- Distribuye la carga de trabajo entre los diferentes puntos de encuentro (RP).
- Simplifica la ubicación de los puntos de encuentro (RP) según la ubicación de los miembros del grupo.
- Evita configuraciones manuales inconsistentes de los puntos de encuentro (RP), que podrían causar problemas de conectividad.
- Se pueden utilizar varios puntos de encuentro (RP) para gestionar diferentes rangos de grupos o como redundancia entre sí.
- El mecanismo Auto-RP funciona mediante dos componentes principales: los `candidate RPs` (C-RPs) y `RP mapping agents` (MAs)

### Candidate RPs (C-RPs):

Un C-RP anuncia su disponibilidad para actuar como RP mediante mensajes de anuncio de RP. Estos mensajes se envían por defecto cada 60 segundos (intervalo de anuncio de RP) al grupo de multidifusión reservado 224.0.1.39 (Cisco-RP-Announce). Los mensajes de anuncio de RP incluyen el rango de direcciones predeterminado (224.0.0.0/4), la dirección IP del C-RP y el tiempo de retención, que es tres veces el intervalo de anuncio de RP. Si hay varios C-RP, se dará prioridad al que tenga la dirección IP más alta.

> [!NOTE]
> La configuración del anuncio de RP permite especificar los grupos multicast concretos que se van a anunciar, en lugar de utilizar el rango de grupos predeterminado 224.0.0.0/4. Esto posibilita tener varios servidores RP en la red que se encarguen de distintos grupos multicast, lo cual resulta útil para el diseño de la red.

### RP Mapping Agents (RP MA):

Los agentes de mapeo de RP (RP MA) se unen al grupo 224.0.1.39 para recibir los anuncios de RP. Almacenan la información de estos anuncios en una caché de mapeo de grupos a RP, junto con los tiempos de retención. Si varios RP anuncian el mismo rango de grupos, se selecciona el RP con la dirección IP más alta.

Los RP MA anuncian el mapeo de RP a otra dirección de grupo multicast conocida, 224.0.1.40 (Cisco-RP-Discovery). Estos mensajes se envían por defecto cada 60 segundos o cuando se detectan cambios. Los anuncios de los MA contienen los RP seleccionados y el mapeo de grupos a RP. Todos los routers con PIM habilitado se unen al grupo 224.0.1.40 y almacenan el mapeo de RP en su propia caché.

Se pueden configurar varios RP MA en la misma red para proporcionar redundancia en caso de fallo. No existe un mecanismo de selección entre ellos; funcionan de forma independiente y todos anuncian la misma información de mapeo de grupos a RP a todos los routers de la red PIM.

La figura 13-22 ilustra el mecanismo Auto-RP: el MA recibe periódicamente los anuncios de RP de Cisco para crear la caché de mapeo de grupos a RP y, posteriormente, envía esta información periódicamente a todos los routers PIM de la red mediante mensajes de descubrimiento de RP de Cisco.

![Image Alt]()

Con Auto-RP, todos los routers aprenden automáticamente la información del RP, lo que facilita su administración y actualización. Auto-RP permite configurar servidores RP de respaldo, lo que posibilita un mecanismo de conmutación de fallos para el RP. 

## PIM Bootstrap Router:

El mecanismo del router de arranque (BSR), descrito en la RFC 5059, es un mecanismo no propietario que proporciona un método automatizado y tolerante a fallos para el descubrimiento y la distribución de los routers de punto de encuentro (RP).

PIM utiliza el BSR para descubrir e informar a todos los routers del dominio PIM sobre la información del conjunto de RP para cada prefijo de grupo. Esta función es similar a la de Auto-RP, pero el BSR se implementa de forma diferente y no es compatible con Auto-RP. El BSR es un estándar IETF que forma parte de la especificación PIM versión 2, definida en la RFC 4601.

El conjunto de RP es un mapeo de grupo a RP que contiene los siguientes elementos:
- Rango de grupo multicast
- Prioridad del RP
- Dirección IP del RP
- Longitud de la máscara de hash
- Indicador SM/Bidir

Los mensajes de arranque (BSM) se originan en el BSR y son propagados por los routers intermedios. Cuando un BSM se reenvía, se envía por todas las interfaces PIM habilitadas que tienen vecinos PIM (incluida la interfaz por la que se recibió el mensaje). Los BSM utilizan la dirección de grupo de routers PIM (224.0.0.13) con un TTL de 1.

Para evitar un punto único de fallo, se pueden implementar varios routers candidatos a BSR (C-BSR) en un dominio PIM. Todos los C-BSR participan en la elección del BSR enviando mensajes de arranque PIM con su propia prioridad por todas las interfaces.

El C-BSR con mayor prioridad se elige como BSR y envía mensajes de arranque a todos los routers PIM del dominio. Si las prioridades de los C-BSR son iguales o si la prioridad no está configurada, se elige como BSR el C-BSR con la dirección IP más alta.

### Candidate RPs:

Un router configurado como candidato a RP (C-RP) recibe los mensajes de bootstrap, que contienen la dirección IP del BSR activo. Al conocer la dirección IP del BSR, el C-RP puede enviar mensajes de anuncio de candidato a RP (C-RP-Adv) directamente a este. Un mensaje C-RP-Adv incluye una lista de pares de direcciones de grupo y máscaras de red. Esto permite al C-RP especificar los rangos de grupos para los que está dispuesto a actuar como RP.

El BSR activo almacena todos los anuncios C-RP recibidos en su caché de asignación de grupo a RP. Posteriormente, el BSR envía esta lista completa de C-RPs a todos los routers PIM de la red mediante mensajes de bootstrap, cada 60 segundos por defecto. Al recibir estos mensajes, los routers actualizan su propia caché de asignación de grupo a RP, lo que les permite conocer las direcciones IP de todos los C-RPs de la red.

A diferencia de Auto-RP, donde el agente de asignación elige el RP activo para un rango de grupo y anuncia el resultado a la red, el BSR no elige el RP activo. Esta tarea queda a cargo de cada router individual de la red.

Cada router de la red utiliza un algoritmo de hash well-known para seleccionar el RP activo para un rango de grupos específico. Dado que todos los routers ejecutan el mismo algoritmo con la misma lista de candidatos a RP, seleccionarán el mismo RP para ese rango de grupos. Se da preferencia a los candidatos a RP con menor prioridad. Si las prioridades son iguales, se selecciona como RP el candidato con la dirección IP más alta. 

La figura 13-23 ilustra el mecanismo BSR: el BSR seleccionado recibe los mensajes de anuncio de los candidatos a RP de todos los routers del dominio y, a continuación, envía mensajes de configuración con la información del RP a través de todas las interfaces PIM habilitadas, los cuales se distribuyen a todos los routers de la red.

![Image Alt]()
