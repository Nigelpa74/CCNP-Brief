# BGP:

## Fundamentos de BGP:

RFC 1654 define el Protocolo de Puerta de Enlace Fronteriza (BGP) como un protocolo de enrutamiento estandarizado EGP que proporciona escalabilidad, flexibilidad y estabilidad de red. 
BGP es el único protocolo utilizado para intercambiar redes en Internet que al momento cuenta con más de 940.000 prefijos de red IPv4 y más de 180.000 prefijos de red IPv6 y continúa creciendo.
En la perspectiva de BGP es una coleccion de routers dentro del control de una misma organizacion.

### Numeros de sistemas autonomos (ASN):

Los ASN eran originalmente de 2 bytes (rango de 16 bits), lo que hizo posible 65.535 ASN. Despues de agotarse con ayuda del `RFC 4893` se expandio a 4 bytes (32 bits) 4,294,967,295 ASN.

Los ASN 64.512–65.534 son ASN privados en el rango de ASN de 16 bits y en 32 bits es 4,200,000,000–4,294,967,294.

La Autoridad de Números Asignados de Internet (IANA) es responsable de asignar todos los ASN públicos para garantizar que sean globalmente únicos. Donde si deseas formar parte es necesario:

- Prueba de un rango de red asignado públicamente.
- Prueba de que la conectividad a Internet se proporciona a través de múltiples conexiones.
- Necesidad de una política de enrutamiento única por parte de los proveedores.

### Atributos de ruta:

BGP utiliza atributos de ruta (PA) asociados a cada ruta de red. Los PA proporcionan a BGP granularidad y control de las políticas de enrutamiento dentro de BGP.

- Well-known mandatory
- Well-known discretionary
- Optional transitive
- Optional non-transitive

Según RFC 4271, los WELL-KNOW deben ser reconocidos por todas las implementaciones de BGP, para el DISCRETIONARY pueden o no incluirse con un anuncio de prefijo.
Los atributos opcionales no tienen que ser reconocidos por todas las implementaciones de BGP.

### Prevention de LOOP

BGP es un protocolo de enrutamiento por vector de ruta y no contiene una topología completa de la red.

El atributo BGP AS_Path es un atributo obligatorio bien conocido e incluye una lista completa de todos los ASN que el anuncio de prefijo ha recorrido desde su AS de origen.

![Image Alt]()

### Address Families:

Originalmente BGP se diseñó para el enrutamiento de prefijos IPv4. 

> [!NOTE]
> Pero la RFC 2858 añadió la capacidad de BGP multiprotocolo (MP-BGP) mediante una extensión denominada `identificador de familia de direcciones` (AFI).

Una familia de direcciones se correlaciona con un protocolo de red específico como IPv4 o IPv6 y se proporciona granularidad adicional a través de un `subsequent addressfamily identifier` (SAFI) como unicast or multicast.
En BGP, `Network Layer Reachability Information ` (NLRI) es la longitud del prefijo y el prefijo.

### Comunicación entre ROUTERS:

BGP no utiliza paquetes de saludo para descubrir vecinos. BGP se diseñó como un protocolo de enrutamiento autónomo. Los vecinos BGP se definen por dirección IP.

> [!IMPORTANT]
> BGP utiliza el puerto TCP 179 para comunicarse con otros enrutadores. TCP permite gestionar la fragmentación.

Las implementaciones más recientes de BGP establecen el `do-not-fragment` (DF) para evitar la fragmentación y dependen del descubrimiento de la MTU de la ruta.

BGP utiliza TCP, que es capaz de cruzar límites de red (es decir, tiene capacidad para múltiples saltos). También puede formar adyacencias que están a varios saltos de distancia.

![Image Alt]()

> [!NOTE]
> Los vecinos BGP conectados a la misma red utilizan la tabla ARP para localizar la dirección IP del router. Las sesiones BGP multisalto requieren información de la tabla de enrutamiento para encontrar la dirección IP del router.

BGP puede considerarse como un protocolo de enrutamiento del plano de control.

Tipos de sesiones BGP: 

- Internal BGP (iBGP): Sesiones establecidas con un enrutador iBGP que se encuentran en el mismo AS. Distancia administrativa (AD) de 200 al momento de la instalación en la RIB del enrutador.
- BGP externo (eBGP): sesiones establecidas con un enrutador BGP que se encuentra en un AS diferente. Con AD de 20

### iBGP:

La necesidad de BGP dentro de un sistema autónomo (AS) suele surgir cuando se requieren múltiples políticas de enrutamiento o cuando se proporciona conectividad de tránsito entre sistemas autónomos. 

En la Figura 11-3, el AS 65200 proporciona conectividad de tránsito al AS 65100 y al AS 65300. El AS 65100 se conecta en R2 y el AS 65300 en R4. 

![Image Alt]()

R2 podría establecer una sesión iBGP directamente con R4, pero R3 no sabría a dónde reenviar el tráfico para llegar al AS 65100 o al AS 65300 cuando el tráfico de cualquiera de los AS llegue a R3, como se muestra en la Figura, porque R3 no tendría la información de reenvío de ruta adecuada para el tráfico de destino.

![Image Alt]()

Se podría suponer que redistribuir la tabla BGP en un IGP soluciona el problema, pero esta no es una solución viable por varias razones:

- Escalabilidad: Al momento de escribir este artículo, Internet contaba con más de 940 000 prefijos de red IPv4 y su tamaño seguía creciendo. Los IGP no podían escalar a ese nivel de rutas.
- Enrutamiento personalizado: Los protocolos de estado de enlace y los protocolos de enrutamiento por vector de distancia utilizan la métrica como método principal para la selección de rutas. Los protocolos IGP siempre utilizan este patrón de enrutamiento para la selección de rutas. BGP utiliza varios pasos para identificar la mejor ruta y permite que los atributos de ruta BGP manipulen la ruta de un prefijo.
- Atributos de ruta: No se pueden mantener todos los atributos de ruta BGP dentro de los protocolos IGP.

### eBGP:

Los puntos eBGP son el componente central de BGP en Internet. Tienen los siguientes comportamietos en comparacion de iBGP.

- El tiempo de vida (TTL) de los paquetes eBGP está configurado en 1 de forma predeterminada. Los paquetes eBGP se descartan en tránsito si se intenta una sesión BGP multisalto. (El TTL de los paquetes iBGP está configurado en 255, lo que permite sesiones multisalto).
- El enrutador publicitario modifica la dirección del siguiente salto de BGP a la dirección IP que origina la conexión BGP.
- El enrutador de publicidad antepone su ASN al atributo AS_Path existente.
- El enrutador receptor verifica que el atributo AS_Path no contenga un ASN que coincida con el ASN local. Sino provoca LOOP.

Las configuraciones para las sesiones eBGP e iBGP son fundamentalmente las mismas, excepto que el ASN en la declaración remota es diferente del ASN definido en el proceso BGP.

La Figura 11-5 muestra las sesiones eBGP e iBGP necesarias entre los enrutadores para permitir la accesibilidad entre AS 65100 y AS 65300. Observe que todos los routers en AS 65200 establecen sesiones iBGP en una malla completa para permitir un reenvío adecuado entre AS 65100 y AS 65300.

![Image Alt]()

### Mensajes BGP:

