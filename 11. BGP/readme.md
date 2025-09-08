# Fundamentos de BGP:

RFC 1654 define el Protocolo de Puerta de Enlace Fronteriza (BGP) como un protocolo de enrutamiento estandarizado EGP que proporciona escalabilidad, flexibilidad y estabilidad de red. 
BGP es el único protocolo utilizado para intercambiar redes en Internet que al momento cuenta con más de 940.000 prefijos de red IPv4 y más de 180.000 prefijos de red IPv6 y continúa creciendo.
En la perspectiva de BGP es una coleccion de routers dentro del control de una misma organizacion.

## Numeros de sistemas autonomos (ASN):

Los ASN eran originalmente de 2 bytes (rango de 16 bits), lo que hizo posible 65.535 ASN. Despues de agotarse con ayuda del `RFC 4893` se expandio a 4 bytes (32 bits) 4,294,967,295 ASN.

Los ASN 64.512–65.534 son ASN privados en el rango de ASN de 16 bits y en 32 bits es 4,200,000,000–4,294,967,294.

La Autoridad de Números Asignados de Internet (IANA) es responsable de asignar todos los ASN públicos para garantizar que sean globalmente únicos. Donde si deseas formar parte es necesario:

- Prueba de un rango de red asignado públicamente.
- Prueba de que la conectividad a Internet se proporciona a través de múltiples conexiones.
- Necesidad de una política de enrutamiento única por parte de los proveedores.

## Atributos de ruta:

BGP utiliza atributos de ruta (PA) asociados a cada ruta de red. Los PA proporcionan a BGP granularidad y control de las políticas de enrutamiento dentro de BGP.

- Well-known mandatory
- Well-known discretionary
- Optional transitive
- Optional non-transitive

Según RFC 4271, los WELL-KNOW deben ser reconocidos por todas las implementaciones de BGP, para el DISCRETIONARY pueden o no incluirse con un anuncio de prefijo.
Los atributos opcionales no tienen que ser reconocidos por todas las implementaciones de BGP.

## Prevention de LOOP

BGP es un protocolo de enrutamiento por vector de ruta y no contiene una topología completa de la red.

El atributo BGP AS_Path es un atributo obligatorio bien conocido e incluye una lista completa de todos los ASN que el anuncio de prefijo ha recorrido desde su AS de origen.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/628ca403d621dc3d5f0000ceed1dfffd160e84d8/11.%20BGP/BGP%201.PNG)

## Address Families:

Originalmente BGP se diseñó para el enrutamiento de prefijos IPv4. 

> [!NOTE]
> Pero la RFC 2858 añadió la capacidad de BGP multiprotocolo (MP-BGP) mediante una extensión denominada `identificador de familia de direcciones` (AFI).

Una familia de direcciones se correlaciona con un protocolo de red específico como IPv4 o IPv6 y se proporciona granularidad adicional a través de un `subsequent addressfamily identifier` (SAFI) como unicast or multicast.
En BGP, `Network Layer Reachability Information ` (NLRI) es la longitud del prefijo y el prefijo.

## Comunicación entre ROUTERS:

BGP no utiliza paquetes de saludo para descubrir vecinos. BGP se diseñó como un protocolo de enrutamiento autónomo. Los vecinos BGP se definen por dirección IP.

> [!IMPORTANT]
> BGP utiliza el puerto TCP 179 para comunicarse con otros enrutadores. TCP permite gestionar la fragmentación.

Las implementaciones más recientes de BGP establecen el `do-not-fragment` (DF) para evitar la fragmentación y dependen del descubrimiento de la MTU de la ruta.

BGP utiliza TCP, que es capaz de cruzar límites de red (es decir, tiene capacidad para múltiples saltos). También puede formar adyacencias que están a varios saltos de distancia.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/628ca403d621dc3d5f0000ceed1dfffd160e84d8/11.%20BGP/BGP%202.PNG)

> [!NOTE]
> Los vecinos BGP conectados a la misma red utilizan la tabla ARP para localizar la dirección IP del router. Las sesiones BGP multisalto requieren información de la tabla de enrutamiento para encontrar la dirección IP del router.

BGP puede considerarse como un protocolo de enrutamiento del plano de control.

Tipos de sesiones BGP: 

- Internal BGP (iBGP): Sesiones establecidas con un enrutador iBGP que se encuentran en el mismo AS. Distancia administrativa (AD) de 200 al momento de la instalación en la RIB del enrutador.
- BGP externo (eBGP): sesiones establecidas con un enrutador BGP que se encuentra en un AS diferente. Con AD de 20

## iBGP:

La necesidad de BGP dentro de un sistema autónomo (AS) suele surgir cuando se requieren múltiples políticas de enrutamiento o cuando se proporciona conectividad de tránsito entre sistemas autónomos. 

En la Figura 11-3, el AS 65200 proporciona conectividad de tránsito al AS 65100 y al AS 65300. El AS 65100 se conecta en R2 y el AS 65300 en R4. 

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/628ca403d621dc3d5f0000ceed1dfffd160e84d8/11.%20BGP/BGP%203.PNG)

R2 podría establecer una sesión iBGP directamente con R4, pero R3 no sabría a dónde reenviar el tráfico para llegar al AS 65100 o al AS 65300 cuando el tráfico de cualquiera de los AS llegue a R3, como se muestra en la Figura, porque R3 no tendría la información de reenvío de ruta adecuada para el tráfico de destino.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/628ca403d621dc3d5f0000ceed1dfffd160e84d8/11.%20BGP/BGP%204.PNG)

Se podría suponer que redistribuir la tabla BGP en un IGP soluciona el problema, pero esta no es una solución viable por varias razones:

- Escalabilidad: Al momento de escribir este artículo, Internet contaba con más de 940 000 prefijos de red IPv4 y su tamaño seguía creciendo. Los IGP no podían escalar a ese nivel de rutas.
- Enrutamiento personalizado: Los protocolos de estado de enlace y los protocolos de enrutamiento por vector de distancia utilizan la métrica como método principal para la selección de rutas. Los protocolos IGP siempre utilizan este patrón de enrutamiento para la selección de rutas. BGP utiliza varios pasos para identificar la mejor ruta y permite que los atributos de ruta BGP manipulen la ruta de un prefijo.
- Atributos de ruta: No se pueden mantener todos los atributos de ruta BGP dentro de los protocolos IGP.

## eBGP:

Los puntos eBGP son el componente central de BGP en Internet. Tienen los siguientes comportamietos en comparacion de iBGP.

- El tiempo de vida (TTL) de los paquetes eBGP está configurado en 1 de forma predeterminada. Los paquetes eBGP se descartan en tránsito si se intenta una sesión BGP multisalto. (El TTL de los paquetes iBGP está configurado en 255, lo que permite sesiones multisalto).
- El enrutador publicitario modifica la dirección del siguiente salto de BGP a la dirección IP que origina la conexión BGP.
- El enrutador de publicidad antepone su ASN al atributo AS_Path existente.
- El enrutador receptor verifica que el atributo AS_Path no contenga un ASN que coincida con el ASN local. Sino provoca LOOP.

Las configuraciones para las sesiones eBGP e iBGP son fundamentalmente las mismas, excepto que el ASN en la declaración remota es diferente del ASN definido en el proceso BGP.

La Figura 11-5 muestra las sesiones eBGP e iBGP necesarias entre los enrutadores para permitir la accesibilidad entre AS 65100 y AS 65300. Observe que todos los routers en AS 65200 establecen sesiones iBGP en una malla completa para permitir un reenvío adecuado entre AS 65100 y AS 65300.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/628ca403d621dc3d5f0000ceed1dfffd160e84d8/11.%20BGP/BGP%205.PNG)

## Mensajes BGP:

Nombre|Description general
:---|:---
OPEN|Establece la adyacencia BGP
UPDATE|Anuncia, actualiza o retira rutas
NOTIFICATION|Indica la condicion de error de un vecino BGP.
KEEPALIVE|Se asegura que los vecinos BGP estan activos.

- Open: Ambas partes negocian las capacidades de sesión antes de establecer el peering BGP. El mensaje OPEN contiene el número de versión de BGP, el ASN del router de origen, el `tiempo de espera` y el `identificador de BGP`.
  - Tiempo de espera: Establece el valor propuesto para el temporizador de retención en segundos para cada vecino BGP. El temporizador de espera, junto con los mensajes de keepalive, funciona como un mecanismo de latido para los vecinos BGP. El valor del tiempo de retención debe ser de al menos 3 segundos o establecerse en 0 para deshabilitar los mensajes de keepalive. En los routers Cisco, el temporizador de retención predeterminado es de 180 segundos.
  - Identificador BGP: El ID del enrutador BGP (RID) es un número único de 32 bits. El RID puede utilizarse como mecanismo para prevenir bucles. Evitar el valor 0.
- Keepalive: BGP no depende del estado de la conexión TCP. Los mensajes de mantenimiento activo se intercambian cada tercio del tiempo de espera acordado. Los dispositivos Cisco tienen un tiempo de espera predeterminado de 180 segundos, por lo que el intervalo de mantenimiento activo predeterminado es de 60 segundos. Si el tiempo de espera se establece en 0, no se envían mensajes de mantenimiento activo entre los vecinos BGP.
- Update: Un mensaje de ACTUALIZACIÓN anuncia las rutas viables. Incluye los prefijos anunciados en la `Network Layer Reachability Information` (NLRI). Los prefijos que deben retirarse se anuncian en el campo "RUTAS RETIRADAS" del mensaje de ACTUALIZACIÓN. Un mensaje de ACTUALIZACIÓN puede actuar como un mecanismo de `Keepalive` para reducir el tráfico innecesario. Al recibir una ACTUALIZACIÓN, el temporizador de retención se restablece a su valor inicial. Si el temporizador de retención llega a cero, la sesión BGP se interrumpe, las rutas de ese vecino se eliminan y se envía un mensaje de ACTUALIZACIÓN a otros vecinos BGP para retirar los prefijos afectados.
- Notification: Como un temporizador de retención que expira, un cambio en las capacidades del vecino o una solicitud de restablecimiento de sesión BGP.

## Estados de vecino BGP:

BGP usa el `finite state machine` (FSM) para mantener una tabla de puntos BGP en estatus operacional.

- Idle
- Connect
- Active
- OpenSent
- OpenConfirm
- Established

En la figura se muestra BGP FSM y los estados.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/628ca403d621dc3d5f0000ceed1dfffd160e84d8/11.%20BGP/BGP%206.PNG)

- Idle: Es la primera etapa del FSM de BGP. BGP detecta un evento de inicio e intenta iniciar un TCP.
- Connect: BGP inicia la conexion BGP, si el TCP 3 hand se completa resetea el `ConnectRetryTimer` y envia el mensaje `Open` al vecino, entonces cambia al estado `OpenSent`. Si el `ConnectRetryTimer` se agota antes de que el episodio se coplete una nueva conexion TCP es intentada. Si la conexion TCP falla el estado pasa a `Active`. BGP usa el TCP en puerto 179 para escuchar las conexiones entrantes.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/847616f49fbe52cf61b0662c0bb3be45fa2105b8/11.%20BGP/BGP%207.PNG)

- Active: BGP inicia un nuevo protocolo de enlace TCP 3 hand. Si se establece una conexión, se envía un mensaje de `Open`, el `temporizador de espera` se establece en 4 minutos y el estado cambia a `OpenSent`. Si este intento de conexión TCP falla, el estado vuelve a Conectar y el `ConnectRetryTimer` se reinicia.
- OpenSent: Se ha enviado un mensaje `Open` desde el enrutador de origen y se está esperando un mensaje `Open` del otro router. Se examina lo siguiente.
  - Versiones BGP deben ser iguales.
  - La dirección IP de origen del mensaje `Open` debe coincidir con la dirección IP configurada para el vecino.
  - ASN deben ser iguales cuando se configura con el vecino.
  - BGP identificador (RIDs) debe ser unico.
  - Parametros de seguridad como TTL y contrasena deben ser configurados apropiadamente.

Si los mensajes de APERTURA no contienen errores, se negocia el `hold time` (utilizando el valor más bajo) y se envía un mensaje de KEEPALIVE.
El estado de la conexión cambia a `OpenConfirm`. Si se encuentra un error en el mensaje de `Open`A, se envía un mensaje de `NOTIFICATION` y el estado vuelve a `Idle`.
- OpenConfirm: BGP espera un mensaje de `KEEPALIVE` o `NOTIFICATION`. Al recibir el mensaje de `KEEPALIVE` de un vecino, el estado cambia a `Established`. Si pasa un error se pasa a `Idle`.
- Established: Los vecinos BGP intercambian rutas mediante mensajes `UPDATE`. A medida que se reciben los mensajes `UPDATE` y `KEEPALIVE`, el `hold time` se reinicia. Si se acaba el tiempo pasa a `Idle`.

# BGP configuracion basica:

- Parámetros de sesión BGP: Los parámetros de sesión BGP proporcionan la configuración necesaria para establecer la comunicación con el vecino BGP remoto. La configuración de sesión incluye el ASN del par BGP, la autenticación y los temporizadores de keepalive.
- Inicialización de Address Family: La Address Family se inicializa en el modo de configuración del enrutador BGP. El anuncio y la sumarizacion de la red se realizan dentro de Address Family.
- Activar la Address Family en el punto BGP: Para iniciar una sesión, se debe activar una Address Family para un vecino. La dirección IP del router se añade a la tabla de vecinos.

Lo siguiente pasos se siguen para activacion de BGP:

- Paso 1: Inicialice el proceso de enrutamiento BGP con el comando global `router bgp (as-number)`.
- Paso 2: Define ESTATICAMENTE el RID router id con el comando `bgp router-id (router-id)` cuando el proceso de BGP de inicialice.
- Pase 3: Identifique la dirección IP y el número de sistema autónomo del vecino BGP con el comando de configuración del enrutador BGP `neighbor (ip-address) remote-as (as-number)`. Si el origen del paquete BGP no coincide con una entrada en la tabla de vecinos, el paquete no se puede asociar a un vecino y se descarta.

> [!NOTE]
> En IOS XE el address family esta por default en IPV4. El comando de configuración del enrutador BGP `no bgp default ipv4-unicast` deshabilita la activación automática de la AFI IPv4, por lo que se requieren los pasos 4 y 5.

- Paso 4: Inicializa el Address Family con la configuracion BGP. Para el `AFI` es ipv4 o ipv6 y para el `SAFI` es `unicast` o `multicast`.
- Paso 5: Activa el Address Family del vecino con el comando `neighbor (ip-address) activate`.

El ejemplo 11-2 muestra cómo configurar R1 y R2 mediante la sintaxis CLI del modificador AFI IPv4 predeterminado y opcional. R1 se configura con la Address Family IPv4 predeterminada habilitada, y R2 deshabilita la Address Family IPv4 predeterminada de IOS y la activa manualmente para el vecino específico 10.12.1.1. El comando `no bgp default ipv4-unicast` no es necesario en R2, y BGP funcionará correctamente con prefijos IPv4, pero la estandarización del comportamiento es más sencilla cuando se trabaja con otras familias de direcciones como IPv6.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/847616f49fbe52cf61b0662c0bb3be45fa2105b8/11.%20BGP/BGP%208.PNG)

## Verificar sesiones BGP:

La sesión BGP se verifica con el comando `show bgp afi safi summary`

> [!NOTE]
> El comando `show ip bgp summary` vino antes de MBGP y no proporcionan una estructura para las capacidades multiprotocolo actuales de BGP. El uso de la sintaxis AFI y SAFI garantiza la coherencia de los comandos, independientemente de la información intercambiada por BGP. Esto se hará más evidente a medida que los ingenieros trabajen con familias de direcciones como IPv6, VPNv4 y VPNv6.

Imagen pra verificar sesion BGP.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/847616f49fbe52cf61b0662c0bb3be45fa2105b8/11.%20BGP/BGP%209.PNG)

Campo|Description general
:---|:---
Neighbor|Dirección IP del punto BGP
V|Version BGP hablado por el punto BGP
AS|Numero de sistema autonomo del punto BGP
MsgRcvd|Contador de mensajes recibido del punto BGP
MsgSent|Contador de mensajes enviados al punto BGP
TblVer|Ultima version de base de datos BGP enviado al punto BGP
InQ|Número de mensajes recibidos del par y puestos en cola para ser procesados
OutQ|Número de mensajes en cola para ser enviados al par
Up/Down|Tiempo que la sesión BGP está establecida o el estado actual si la sesión no está en un estado establecido
State/PfxRcd|Estado actual del par BGP o la cantidad de prefijos recibidos del punto BGP

El estado de la sesión vecina BGP, los temporizadores y otra información esencial sobre el punto BGP están disponibles con el comando `show bgp afi safi neighbors ip-address`.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/1e5c4cf0df680752ed0a152e2d2445c5b2eb306f/11.%20BGP/BGP%2010.PNG)
![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/1e5c4cf0df680752ed0a152e2d2445c5b2eb306f/11.%20BGP/BGP%2011.PNG)
![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/1e5c4cf0df680752ed0a152e2d2445c5b2eb306f/11.%20BGP/BGP%2012.PNG)

## Anunciado de rutas:

Las declaraciones de red BGP no habilitan BGP para una interfaz específica; en cambio, identifican prefijos de red específicos que se instalarán en la tabla BGP, conocidos como `Loc-RIB`.
Despues de configurar un `BGP network` estado, el proceso BGP procesa busquedas en la `RIB` rutas de un prefijo que encaje con a red exacta. Despues de comprobarlo lo siguiente pasa.

- Connected Network (C): El atributo BGP del siguiente salto se establece en 0.0.0.0, el atributo de origen BGP se establece en i (IGP) y el peso BGP se establece en 32.768.
- Ruta estatica o procolo de enrutamiento: El atributo BGP del siguiente salto se establece en la dirección IP del siguiente salto en la RIB, el atributo de origen BGP se establece en i (IGP) y el peso BGP se establece en 32 768.

Una vez configurados los PA BGP, la ruta se instala en Loc-RIB (la tabla BGP). Solo si cumplen ciertas condiciones.
- Paso 1: Pase una comprobación de validez. Verifique que el NLRI sea válido y que la dirección del siguiente salto se pueda resolver en el RIB. SINO RIP.
- Paso 2: Procesar las políticas de rutas vecinas salientes. Tras el procesamiento, si una ruta no fue denegada por las políticas de salida, se conserva en Adj-RIB-Out para su posterior consulta.
- Paso 3: Anunciar las rutas a los pares BGP. Si el PA BGP del siguiente salto es 0.0.0.0 para los prefijos del PA NLRI, la dirección del siguiente salto se cambia a la dirección IP de la sesión BGP.

Imagen para el proceso mencionado.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/b6679a0a690e4e683bd05ded67b6b32198339221/11.%20BGP/BGP%2013.PNG)

> [!NOTE]
> BGP solo anuncia la mejor ruta de forma predeterminada a otros puntos BGP, independientemente de la cantidad de rutas en Loc-RIB.

La declaración de red reside en la Address Family correspondiente dentro de la configuración del enrutador BGP. El comando `network (network) mask (subnet-mask) [route-map routemap-name]` se utiliza para anunciar redes IPv4. El comando `route-map` opcional proporciona un método para configurar PA BGP específicos cuando la ruta se instala en la Loc-RIB. 

Imagen para mostrar como se anuncia desde 2 perspectivas del Address Family automatico y no auto.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/edcd51fda694f184d236405b5ac0121aaddc1869/11.%20BGP/BGP%2014.PNG)

## Recepción y visualización de rutas:

BGP utiliza tres tablas para mantener el prefijo de red y los atributos de ruta (PA) para una ruta.

- Adj-RIB-In: Contiene las rutas en su formato original (es decir, antes de procesar las políticas de ruta entrantes). Para ahorrar memoria, la tabla se purga después de procesar todas las políticas de ruta.
- Loc-RIB: Contiene todas las rutas originadas localmente o recibidas de otros puntos BGP. Una vez que las rutas superan la comprobación de validez y accesibilidad del siguiente salto, el algoritmo de mejor ruta de BGP selecciona la mejor ruta para un prefijo específico.
- Adj-RIB-Out: Contiene las rutas después de que se hayan procesado las políticas de ruta de salida.

No todas las rutas en el Loc-RIB se anuncian a un par BGP ni se instalan en el RIB cuando se reciben de un punto BGP. Se procesa:

- Paso 1: Almacena la ruta en la RIB Adjunta en su estado original y aplique la política de rutas entrantes según el vecino en el que se recibió.
- Paso 2. Actualice la Loc-RIB con la última entrada. La Adj-RIB se borra para ahorrar memoria.
- Paso 3. Realice una comprobación de validez para verificar que la ruta sea válida y que la dirección del siguiente salto se pueda resolver en la RIB. Si la ruta falla, permanece en la Adj-RIB, pero no se procesa.
- Paso 4. Identifique la mejor ruta BGP y pase solo la mejor ruta y sus atributos al paso 5.
- Paso 5. Instale la ruta de mejor ruta en la RIB, procese la política de rutas salientes, almacene las rutas no descartadas en la Adj-RIB-Out y anuncie a puntos BGP.

La figura 11-9 muestra la lógica completa de procesamiento de ruta BGP.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/edcd51fda694f184d236405b5ac0121aaddc1869/11.%20BGP/BGP%2015.PNG)

El comando show bgp afi safi muestra el contenido de la tabla BGP (Loc-RIB) en el router.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/edcd51fda694f184d236405b5ac0121aaddc1869/11.%20BGP/BGP%2016.PNG)

Explicacion de los campos.

Campo|Description general
:---|:---
Network|Una lista de los prefijos instalados en BGP. Si existen varias rutas para el mismo prefijo, solo se muestra el primero; las demás rutas dejan un espacio en blanco en la salida. Las válidas se indican con *.La meJor ruta se indica con un corchete angular (>).
Next-Hop|Un atributo de ruta BGP obligatorio bien conocido que define la dirección IP para el siguiente salto para esa ruta específica.
Metric|Multiple-exit discrimator (MED). Un atributo de ruta BGP no transitivo opcional utilizado en BGP para la ruta específica.
LocPrf|Local Preference. Un atributo de ruta BGP discrecional bien conocido que se utiliza en el algoritmo de mejor ruta BGP para la ruta específica.
Weight|Un atributo definido por Cisco y de importancia local que se utiliza en el algoritmo de mejor ruta de BGP para la ruta específica.
Path and Origin|AS_Path: Un atributo de ruta BGP obligatorio y well-known, utilizado para la prevención de bucles y en BGP. Origen: Un atributo de ruta BGP obligatorio y well-known, utilizado en el algoritmo BGP bestpath. Un valor de i representa un IGP, e indica EGP y ? indica una ruta redistribuida en BGP.

El comando `show bgp afi safi network` muestra todas las rutas para un prefijo específico y los atributos de ruta BGP para ese prefijo.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/34db4bdeb2911c4233071470a958ba73823186a0/11.%20BGP/BGP%2017.PNG)

> [!NOTE]
> El comando `show bgp afi safi detail` muestra la tabla BGP completa.

La Adj-RIB-Out es una tabla única que se mantiene para cada punto BGP. El comando `show bgp afi safi neighbor ip-address adverted route` muestra el contenido de la Adj-RIB-Out del router vecino específico.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/34db4bdeb2911c4233071470a958ba73823186a0/11.%20BGP/BGP%2018.PNG)

> [!NOTE]
> El comando `show bgp ipv4 unicast summary` también se puede usar para verificar el intercambio de rutas entre nodos.

## Anuncios de rutas BGP de fuentes indirectas:

BGP debe considerarse como una aplicación de enrutamiento, ya que la sesión BGP y el anuncio de ruta son dos componentes separados. En lo siguiente se menctra las instalaciones de diferentes rutas aprendidos por EIGRP, OSPF y estatico.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/34db4bdeb2911c4233071470a958ba73823186a0/11.%20BGP/BGP%2019.PNG)

Tabla de R1

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/34db4bdeb2911c4233071470a958ba73823186a0/11.%20BGP/BGP%2020.PNG)

Comandos donde se usa el estado `network` para la instalacion de las rutas por eigrp y estatica. OSPF se REDISTRIBUYE.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/34db4bdeb2911c4233071470a958ba73823186a0/11.%20BGP/BGP%2021.PNG)

> [!NOTE]
> Redistribuir las rutas aprendidas de un IGP a BGP es completamente seguro; sin embargo, las rutas aprendidas de BGP deben redistribuirse a un IGP con precaución ya que lo sobrecargara con mas 1 MM de rutas.

El ejemplo 11-13 muestra las tablas de enrutamiento BGP en R1 y R2. Observe que en R1, el siguiente salto coincide con el siguiente salto aprendido de la RIB, la ruta AS_Path está vacía y el código de origen es `IGP` (para rutas aprendidas de la declaración de red) o *incomplete* (redistribuido). La métrica se transfiere de los protocolos de enrutamiento IGP de R3 y R5 y se refleja como el MED. R2 aprende las rutas estrictamente de eBGP y solo ve el MED y los códigos de origen.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/34db4bdeb2911c4233071470a958ba73823186a0/11.%20BGP/BGP%2022.PNG)
![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/34db4bdeb2911c4233071470a958ba73823186a0/11.%20BGP/BFP%2023.PNG)

# IPV4 sumarizacion de rutas.

También conocida como agregación de rutas, conserva los recursos del enrutador y acelera el cálculo de la mejor ruta al reducir el tamaño de la tabla. Si bien la mayoría de los proveedores de servicios no aceptan prefijos mayores que /24 para IPv4 (de /25 a /32), Internet, en la actualidad, aún cuenta con más de 940 000 rutas y continúa creciendo. El resumen de rutas es necesario para reducir el tamaño de la tabla BGP para los enrutadores de Internet.
El resumen de rutas BGP en los **ROUTERS DE BORDE** BGP reduce el cálculo de rutas.

Hay 2 tecnicas de sumarizacion de rutas:

- Estatico: Cree una ruta estática a Null0 para el prefijo de red sumarizada y anuncie el prefijo con un **ESTADO** de red. La desventaja de esta técnica es que la ruta sumarizada siempre se anuncia, incluso si las redes no están disponibles.
- Dinamico: Configure un prefijo de red de agregación. Cuando las rutas de componentes viables que coinciden con el prefijo de red de agregación entran en la tabla BGP, se crea el prefijo de agregación. El enrutador de origen establece el siguiente salto en Null0 como ruta de descarte para el prefijo de agregación para prevenir bucles.

En ambos métodos de agregación de rutas, se anuncia en BGP un nuevo prefijo de red con una longitud de prefijo más corta. Dado que el prefijo agregado es una nueva ruta, el router sumarizador es el originador de la nueva ruta agregada.

## Agregando direcciones:

El resumen de ruta dinámica se logra con el comando de configuración de Address Family BGP `aggregate-address network subnet-mask [summary-only] [as-set]`.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/34db4bdeb2911c4233071470a958ba73823186a0/11.%20BGP/BGP%2024.PNG)

El ejemplo 11-14 muestra las tablas BGP de R1, R2 y R3 antes de realizar el resumen de ruta. Las redes stub de R1 (172.16.1.0/24, 172.16.2.0/24 y 172.16.3.0/24) se anuncian a través de todos los sistemas autónomos, junto con las direcciones de bucle invertido de R1, R2 y R3 (192.168.1.1/32, 192.168.2.2/32 y 192.168.3.3/32) y los enlaces de los puntos (10.12.1.0/24 y 10.23.1.0/24).

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/11fbf754f8fcf165e3d3c3771edd512170da097b/11.%20BGP/BGP%2025.PNG)
![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bb70bde854f14f6998ffb3fb5fa60f2061380d16/11.%20BGP/BGP%2027.PNG)

El R1 agrupa todas las redes stub (172.16.1.0/24, 172.16.2.0/24 y 172.16.3.0/24) en una ruta resumen 172.16.0.0/20. El R2 agrupa todas las direcciones de bucle invertido del router en una ruta resumen 192.168.0.0/16.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bb70bde854f14f6998ffb3fb5fa60f2061380d16/11.%20BGP/BGP%2028.PNG)
![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bb70bde854f14f6998ffb3fb5fa60f2061380d16/11.%20BGP/BGP%2029.PNG)

Despues de sumarizacion.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bb70bde854f14f6998ffb3fb5fa60f2061380d16/11.%20BGP/BGP%2030.PNG)
![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bb70bde854f14f6998ffb3fb5fa60f2061380d16/11.%20BGP/BGP%2031.PNG)

Observe que las rutas de resumen 172.16.0.0/20 y 192.168.0.0/16 son visibles, pero las rutas de componente más pequeñas aún existen en todos los enrutadores. El comando "aggregate-address" anuncia la ruta de resumen además de las rutas de componente originales. El uso de la palabra clave opcional "summary-only" suprime las rutas de componente y solo se anuncia la ruta de resumen. El ejemplo 11-17 muestra la configuración con la palabra clave "summary-only".

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bb70bde854f14f6998ffb3fb5fa60f2061380d16/11.%20BGP/BGP%2032.PNG)

Tablas de R2 despues de la sumarizacion con "summary-only".

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bb70bde854f14f6998ffb3fb5fa60f2061380d16/11.%20BGP/BGP%2033.PNG)
![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bb70bde854f14f6998ffb3fb5fa60f2061380d16/11.%20BGP/BGP%2034.PNG)

El ejemplo 11-20 muestra que se han suprimido las redes stub del R1 y que la ruta de descarte de resumen para la red 172.16.0.0/20 también se ha instalado en el RIB.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bb70bde854f14f6998ffb3fb5fa60f2061380d16/11.%20BGP/BGP%2035.PNG)

## Atomic Aggregate:

Las rutas sumarizadas funcionan como nuevas rutas BGP con una longitud de prefijo más corta. Cuando un enrutador BGP resume una ruta, no publica la información de AS_Path anterior al resumen. Los atributos de ruta BGP como AS_Path, MED y comunidades BGP no se incluyen en el nuevo anuncio BGP.

El atributo de **Atomic Aggregate** indica que se ha producido una pérdida de información de ruta. Para demostrarlo mejor en la siguiente imagen, se ha eliminado el resumen de ruta BGP anterior en R1 y se ha añadido a R2, de modo que R2 ahora resume las redes 172.16.0.0/20 y 192.168.0.0/16 con supresión de ruta específica mediante la palabra clave "**summary-only**".

![Image Alt]()

En lo siguiente se muestra las tablas de R2, R3 y R2 resume y suprime las redes stub de R1 (172.16.1.0/24, 172.16.2.0/24 y 172.16.3.0/24) en la ruta de resumen 172.16.0.0/20. Las rutas componentes mantienen un AS_Path de 65100 en R2 (**Solo en R2**), mientras que la ruta de resumen 172.16.0.0/20 en R3 aparece generada localmente por R2.

![Image Alt]()

En la siguiente imagen se muestra la ruta sumarizada 17.16.0.0/20 en router 3. La información del router indica que el enrutador con el RID 192.168.2.2 sumarizo (agregó) las rutas en AS 65200.

## Agregación de rutas con AS_SET:

Para mantener el historial de información de la ruta BGP (paso de ASN), se puede usar la palabra clave opcional `as-set` con el comando de `aggregate address`. A medida que el enrutador genera la ruta resumida, se copia la información de la ruta BGP de las rutas componentes. Ejemplo en R2.

![Image Alt]()

y asi es como se muestra en el R3 despues del comando reconfigurado con `as-set`

![Image Alt]()

¿Observaste que la ruta resumida 192.168.0.0/16 ya no está presente en la tabla BGP de R3? y de echo lo estara e R1, el motivo es que R2 esta sumarizando la red 192.168.0.0/16 donde estan la **LOOPBACK DE R1 Y R3** al generalizar las rutas de loopbacks traen consigo su ASN que son 65100 y 65300 que serian detectados como LOOP por parte de BGP. Imagen de R2 en `aggregate-address` y su tabla BGP.

![Image Alt]()

Ejemplo de que R1 no importa 192.168.0.0/16 de R2 por detectar el AS65100 de su propia loopback que es anunciada por R2.

![Image Alt]()

##
