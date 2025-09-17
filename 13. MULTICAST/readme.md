El protocolo multicast se implementa en casi todos los tipos de red. Permite a un equipo origen enviar paquetes de datos a un grupo de equipos destino (_receivers_) de forma eficiente, optimizando el uso del ancho de banda y los recursos del sistema. Este capítulo describe la utilidad del protocolo multicast, así como los protocolos fundamentales necesarios para comprender su funcionamiento, como IGMP, PIM (modo denso/modo disperso) y los rendezvous points (RPs).

# Fundamentos de multicast: 

La comunicación IP tradicional entre dispositivos de red suele utilizar uno de los siguientes métodos de transmisión:
- Unicast (de uno a uno)
- Broadcast (de uno a todos)
- Multicast (de uno a varios)

La comunicación multicast es una tecnología que optimiza el uso del ancho de banda de la red y ahorra recursos del sistema. 

Para su funcionamiento en redes de capa 2, utiliza el `Internet Group Management Protocol (IGMP)`, mientras que en redes de capa 3 utiliza el `Protocol Independent Multicast (PIM)`.

La figura 13-1 ilustra cómo funciona IGMP entre los receptores y el enrutador multicast local, y cómo funciona PIM entre los enrutadores. Estas dos tecnologías trabajan conjuntamente para permitir que el tráfico multicast fluya desde la fuente a los `receivers`. 

La figura 13-2 muestra un ejemplo en el que cinco estaciones de trabajo visualizan el mismo vídeo, transmitido por un servidor mediante tráfico unicast (uno a uno). Cada flecha representa un flujo de datos del mismo vídeo dirigido a cinco dispositivos diferentes. Si cada flujo es de 10 Mbps, el enlace de red entre R1 y R2 necesita un ancho de banda de 50 Mbps. El enlace entre R2 y R4 requiere 30 Mbps, y el enlace entre R2 y R5, 20 Mbps. El servidor debe mantener la información de estado de sesión para todas las conexiones con los dispositivos. El ancho de banda y la carga del servidor aumentan a medida que más receptores solicitan el mismo flujo de vídeo.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%201.PNG)
![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%202.PNG)

Otra alternativa para que las cinco estaciones de trabajo reciban el vídeo es enviarlo desde el servidor mediante transmisión por Broadcast (uno a todos). La figura 13-3 muestra un ejemplo de cómo se transmite el mismo flujo de vídeo mediante difusión IP dirigida. La carga del servidor se reduce, ya que solo tiene que gestionar una única sesión en lugar de varias. Este flujo de vídeo consume solo 10 Mbps de ancho de banda en todos los enlaces de red. Sin embargo, este método presenta algunas desventajas:
- La función de Broadcast IP dirigida no está activada por defecto en los routers Cisco, y su activación expone el router a ataques de denegación de servicio distribuida (DDoS).
- Las tarjetas de red de las estaciones de trabajo que no reciben el vídeo deben procesar los paquetes de difusión y enviarlos a la CPU, lo que consume recursos del procesador. En la figura 13-3, la estación de trabajo F está procesando paquetes no deseados.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%203.PNG)

Por estas razones, el tráfico de difusión no suele ser recomendable. El tráfico multicast permite la comunicación de (uno a muchos), donde solo se envía un paquete de datos por enlace y, a continuación, se replica a través de los enlaces cuando los datos se bifurcan en un dispositivo de red a lo largo del `multicast distribution tree (MDT)`. Estos paquetes de datos constituyen un flujo que utiliza una dirección IP de destino especial, conocida como _group address_. Un servidor de flujo gestiona una única sesión, y los dispositivos de red solicitan selectivamente recibir dicho flujo. Los dispositivos receptores de un flujo multicast se denominan _receivers_. Entre las aplicaciones comunes que utilizan el tráfico multicast se incluyen Cisco TelePresence, vídeo en tiempo real, IPTV, cotizaciones bursátiles en directo, educación a distancia, videoconferencias, música de espera y juegos en línea. La figura 13-4 muestra un ejemplo del mismo flujo de vídeo utilizando multicast. Cada enlace de red consume solo 10 Mbps de ancho de banda, igual que con el tráfico de difusión, pero solo los receptores interesados ​​en el flujo de vídeo procesan el tráfico multicast. Por ejemplo, la estación de trabajo F descartaría el tráfico multicast a nivel de la tarjeta de red, ya que no estaría configurada para recibirlo.

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

- Source Specific Multicast (SSM) Bloque (232.0.0.0/8): Es el bloque default de SSM. SSM es una extensión de PIM, descrita en la RFC 4607. SSM reenvía el tráfico a los receptores únicamente desde las fuentes multicast para las que los receptores han expresado interés explícito; está diseñado principalmente para aplicaciones de transmisión a múltiples destinatarios.
- Bloque GLOP (233.0.0.0/8): Las direcciones de este bloque son direcciones estáticas de ámbito global. Se asignan a dominios con un número de sistema autónomo (ASN) de 16 bits, mediante la asignación del ASN del dominio (expresado en octetos como X.Y) a los dos octetos centrales del bloque GLOP, resultando una dirección del tipo 233.X.Y.0/24. Esta asignación se define en la RFC 3180. Los dominios con un ASN de 32 bits pueden solicitar espacio en el bloque III o considerar el uso de direcciones multicast IPv6.
- Ámbito local de organización (239.0.0.0/8): Estas direcciones, descritas en la RFC 2365, están limitadas a un grupo u organización local. Son similares a los rangos de direcciones IP unicast reservados (como 10.0.0.0/8) definidos en la RFC 1918 y la IANA no las asignará a otros grupos o protocolos. En otras palabras, los administradores de red pueden usar direcciones multicast en este rango dentro de su dominio sin temor a conflictos con otros usuarios de Internet.

## Direcciones multicast de capa 2:

Históricamente, las NIC de un segmento de LAN podían recibir solo paquetes dirigidos a su dirección MAC o a la dirección MAC de difusión. Este método podía sobrecargar los recursos de enrutamiento durante la replicación de paquetes en segmentos de LAN. Para solucionar este problema, se creó un método alternativo para el tráfico multicast, que permitía replicar este tráfico sin manipular los paquetes y utilizando una dirección MAC de destino estandarizada. 

Una dirección MAC es un valor único asociado a una NIC, que sirve para identificarla de forma exclusiva en un segmento de LAN. Las direcciones MAC son números hexadecimales de 12 dígitos (48 bits), generalmente representados en segmentos de 8 bits separados por guiones (-) o dos puntos (:) (por ejemplo, `00-12-34-56-78-00` o `00:12:34:56:78:00`). 

Cada dirección IP de grupo multicast se asigna a una dirección MAC específica, que permite a las interfaces Ethernet identificar los paquetes multicast dirigidos a ese grupo. Un segmento de LAN puede tener varios flujos de tráfico, y el receptor determina qué tráfico enviar a la CPU para su procesamiento según la dirección MAC asignada al tráfico multicast. 

Los 24 primeros bits de una dirección MAC multicast siempre comienzan con `01:00:5E`. El bit menos significativo del primer byte (bit I/G o bit unicast/multicast) indica si el paquete es multicast (valor 1). El bit 25 siempre es 0. Los 23 bits menos significativos de la dirección MAC multicast se copian de los 23 bits menos significativos de la dirección IP del grupo multicast. 

La figura 13-5 muestra un ejemplo de la asignación de la dirección IP multicast `239.255.1.1` a la dirección MAC multicast `01:00:5E:7F:01:01`. Los 25 bits más significativos son fijos, y los 23 bits menos significativos se copian directamente de la dirección IP del grupo multicast.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%205.PNG)

De los 9 bits de la dirección IP multicast que no se copian en la dirección MAC multicast, los 4 bits más significativos (1110) son fijos; esto deja 5 bits variables que no se transfieren a la dirección MAC. Por lo tanto, existen 32 (2⁵) direcciones IP multicast que no son únicas y pueden corresponder a una misma dirección MAC; es decir, se superponen. La figura 13-6 muestra un ejemplo de dos direcciones IP multicast que se superponen porque se corresponden con la misma dirección MAC multicast.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/cb05a97ea16650f27dd242a344a15df8565bacf6/13.%20MULTICAST/IMG/MULTI%20FUND/MULTI%20FUND%206.PNG)

Cuando un dispositivo receptor desea recibir una transmisión multicast específica, envía un mensaje IGMP de adhesión al grupo, utilizando la dirección IP del grupo multicast correspondiente. El receptor reprograma su interfaz para aceptar la dirección MAC del grupo multicast asociada a dicha dirección IP. Por ejemplo, un PC podría enviar un mensaje IGMP de adhesión al grupo 239.255.1.1 y reprogramaría su tarjeta de red para recibir la dirección MAC `01:00:5E:7F:01:01`. Si el PC recibiera una actualización OSPF dirigida a la dirección `224.0.0.5` y su dirección MAC multicast correspondiente `01:00:5E:00:00:05`, ignoraría dicha actualización, evitando así el procesamiento de tráfico multicast no deseado y optimizando el rendimiento de la CPU.

# Protocolo de gestión de grupos de Internet (IGMP): 

`Internet Group Management Protocol (IGMP)` es el protocolo que utilizan los dispositivos receivers para unirse a grupos de difusión y comenzar a recibir tráfico de esos grupos. Este protocolo debe estar habilitado tanto en los dispositivos receivers como en las interfaces de los routers que están conectados a ellos. Cuando un dispositivo receiver desea recibir tráfico de difusión de una fuente, envía una solicitud de adhesión IGMP a su router. Si el router no tiene IGMP habilitado en la interfaz, la solicitud se ignora. Existen tres versiones de IGMP. El RFC 1112 define IGMPv1, una versión antigua que se usa con poca frecuencia. El RFC 2236 define IGMPv2, que es común en la mayoría de las redes de difusión, y el RFC 3376 define IGMPv3, utilizado por SSM. En este capítulo se describen únicamente las versiones IGMPv2 e IGMPv3.

# Protocolo multicast independiente (PIM): 



# Puntos de encuentro: Esta sección describe el propósito, la función y el funcionamiento de los puntos de encuentro en una red multicast.

