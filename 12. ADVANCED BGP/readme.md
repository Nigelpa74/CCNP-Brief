El Protocolo de Border Gateway Protocol (BGP) admite cientos de miles de rutas, lo que lo convierte en la opción ideal para Internet. 
Las organizaciones también utilizan BGP por su flexibilidad y sus propiedades de ingeniería de tráfico. 
Este capítulo amplía el Capítulo 11, "BGP", explicando las funciones y conceptos avanzados de BGP relacionados con el protocolo de enrutamiento BGP, como el multihoming BGP, el filtrado de rutas, las comunidades BGP y la lógica para identificar la mejor ruta para una red de destino específica.

# BGP Multihoming: 

El método más sencillo para proporcionar redundancia es proporcionar un segundo circuito. Añadir un segundo circuito y establecer una segunda sesión BGP a través de ese enlace con otro punto se conoce como multihoming BGP, ya que existen múltiples sesiones para aprender rutas y establecer la conectividad.
El comportamiento predeterminado de BGP es anunciar solo la mejor ruta a la RIB, lo que significa que solo se utiliza una ruta para un prefijo de red al reenviar el tráfico de red a un destino.

## Resiliencia en los proveedores de servicios (ISP):

Pueden ocurrir fallos de enrutamiento dentro de la red de un proveedor de servicios, y algunas organizaciones optan por usar un proveedor de servicios diferente para cada circuito. Se podría seleccionar un segundo proveedor de servicios por diversas razones, pero la elección suele basarse en el coste, la disponibilidad de circuitos para ubicaciones remotas o la separación del plano de control.

Escenarios Multihoming:

- Escenario 1: R1 se conecta a R3 con el mismo SP. Este diseño contempla los fallos de enlace; sin embargo, un fallo en cualquiera de los routers o en la red del SP1 provoca un fallo de red.
- Escenario 2: R1 se conecta a R3 y R4 con el mismo SP. Este diseño contempla los fallos de enlace; sin embargo, un fallo en R1 o en la red del SP1 provoca un fallo de red.
- Escenario 3: R1 se conecta a R3 y R4 con diferentes SP. Este diseño contempla los fallos de enlace y de la red de cualquiera de los SP, y puede optimizar el enrutamiento del tráfico.
Sin embargo, un fallo en R1 provoca un fallo de red.
- Escenario 4: R1 y R2 establecen una sesión iBGP entre sí. R3 se conecta a SP1 y R4 a SP2. Este diseño contempla los fallos de enlace y de la red de cualquiera de los SP, y puede optimizar el enrutamiento del tráfico.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c9ce545af33d9609feb2d2b07e5fbfa95929dce3/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20MULTI/ADV%20BGP%20MULTI%201.PNG)

## Enrutamiento de tránsito de Internet:

Si una empresa utiliza BGP para conectarse con más de un proveedor de servicios, corre el riesgo de que su sistema autónomo (AS) se convierta en un **AS de tránsito**.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c9ce545af33d9609feb2d2b07e5fbfa95929dce3/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20MULTI/ADV%20BGP%20MULTI%202.PNG)

Si R1 y R2 usan la política de enrutamiento BGP predeterminada, SP3 recibe el prefijo 100.64.1.0/24 de AS 100 y AS 500. SP3 selecciona la ruta a través de AS 500 porque AS_Path **es mucho más corto** **que el que pasa por las redes de SP1 y SP2**. Un usuario que se conecta a SP3 (AS 300) se enruta a través de la red empresarial (AS 500) para llegar a un servidor conectado a SP4 (AS 400).

La red AS 500 proporciona enrutamiento de tránsito **a todos** los usuarios de Internet, lo que puede saturar los enlaces del punto de AS 500. Además de causar problemas a los usuarios de AS 500.
El enrutamiento de tránsito se puede evitar aplicando políticas de ruta BGP saliente que solo permitan anunciar rutas BGP locales a otros sistemas autónomos.

## Ruta de tránsito de sucursales: 

Un diseño de red adecuado debe considerar los patrones de tráfico para evitar un enrutamiento deficiente o bucles de enrutamiento. Todos los router están configurados para preferir el transporte MPLS SP2 al transporte MPLS SP1 (activo/pasivo). Todos los enrutadores de rama se interconectan y anuncian todas las rutas a través de eBGP a los enrutadores SP. Los enrutadores de rama no filtran ningún prefijo y todos configuran la preferencia local para MPLS SP2 a un valor más alto para enrutar el tráfico a través de él.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/c9ce545af33d9609feb2d2b07e5fbfa95929dce3/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20MULTI/ADV%20BGP%20MULTI%203.PNG)

Cuando la red funciona correctamente, el tráfico entre los sitios utiliza la red SP preferida (MPLS SP2) en ambas direcciones. Esto simplifica la resolución de problemas cuando el flujo de tráfico es simétrico (la misma ruta en ambas direcciones), a diferencia del reenvío asimétrico (una ruta diferente para cada dirección), ya que es necesario descubrir la ruta completa en ambas direcciones. **La ruta se considera determinista cuando el flujo entre sitios está predeterminado y es predecible.**

Durante una falla de enlace en la red SP, existe la posibilidad de que un router de rama se conecte al router de rama de destino a través de un router de rama intermediario.

![Image Alt]()

La conectividad de tránsito no planificada presenta los siguientes problemas:
- Los circuitos del router de tránsito pueden saturarse porque fueron dimensionados solo para el tráfico de ese sitio y no para el tráfico que los atraviesa.
- Los patrones de enrutamiento pueden volverse impredecibles y **no deterministas**. En este escenario, el tráfico de R31 fluye a través de R41, pero el tráfico de retorno puede tomar una ruta de retorno diferente. La ruta podría ser muy diferente si el tráfico proveniera de un enrutador diferente. Esto impide el enrutamiento determinista, complica la resolución de problemas y puede hacer que el personal de su NOC se sienta como si estuviera jugando al topo al resolver problemas de red.

Los entornos multihomed deben configurarse de forma que los routers de sucursal **no puedan actuar como routers de tránsito**. En la mayoría de los diseños, el enrutamiento en tránsito del tráfico desde otra sucursal no es deseable, ya que el ancho de banda de la WAN podría no estar dimensionado adecuadamente. En esencia, las sucursales no publican lo que aprenden de la WAN, sino solo las redes que se conectan a la LAN. Si se requiere un comportamiento de tránsito, este se restringe a los centros de datos o ubicaciones específicas, como se indica a continuación.

- Un diseño de enrutamiento adecuado permite adaptarse a las interrupciones.
- El ancho de banda se puede dimensionar según corresponda.
- El patrón de enrutamiento es bidireccional y predecible.

> [!NOTE]
> El enrutamiento de tránsito en el centro de datos u otras ubicaciones planificadas es normal en los diseños empresariales, ya que se ha considerado el ancho de banda. Normalmente, esto se realiza cuando algunas sucursales solo están disponibles con un proveedor de servicios (SP) y las demás se conectan con un SP diferente.

# Conditional Matching: 

En esta sección se revisan algunas de las técnicas comunes utilizadas para hacer coincidir condicionalmente una ruta, mediante listas de control de acceso (ACL), listas de prefijos, expresiones regulares (regex) y ACL de ruta AS.

## Access control lists (ACLs):

Originalmente diseñado para proporcionar filtrado de paquetes que fluyen hacia o desde una interfaz de red, similar a la funcionalidad de un firewall básico. Ahora las ACL proporcionan clasificación de paquetes para una variedad de características, como la calidad de servicio (QoS) o para identificar redes dentro de protocolos de enrutamiento.

ACLs are composed of _access control entries (ACEs)_ donde puede ser negar o permitir. La clasificación de paquetes comienza desde arriba (secuencia más baja) y continúa hacia abajo (secuencia más alta) hasta que se identifica un **patrón coincidente**. Cuando se encuentra una coincidencia, se toma la acción apropiada (permitir o denegar) y el procesamiento se detiene. Al final de cada ACL hay una **ACE de denegación implícita**, que deniega todos los paquetes que no coincidieron previamente en la ACL.

> [!NOTE]
> Recuerda tener un buen orden o denegaras todo lo que quieras permitir.

- ACL estándar: Definen paquetes basándose únicamente en la red de origen.
- ACL extendidas: Definen paquetes según el origen, el destino, el protocolo, el puerto o una combinación de otros atributos de paquete. _Este libro trata sobre el enrutamiento y limita el alcance de las ACL al origen, el destino y el protocolo._(FUTURA INVESTIGACION)

Las ACL estándar utilizan una entrada numerada del 1 al 99, del 1300 al 1999 o una ACL con nombre. Las ACL extendidas utilizan una entrada numerada del 100 al 199, del 2000 al 2699 o una ACL con nombre. Las ACL con nombre aportan relevancia a la funcionalidad de la ACL siendo aveces preferida.

### Standar ACL:

- Paso 1. Defina la ACL con el comando `ip access-list standard {acl-number | acl-name}` y entrara al modo configuración de ACL donde puede ingresar los ACE.
- Paso 2. Configure la entrada ACE específica con el comando `[sequence] {permit | deny} source source-wildcard`. En lugar de usar `source source-wildcard`, la palabra clave `any` reemplaza 0.0.0.0 255.255.255.255, y el uso de la palabra clave `host` hace referencia a una dirección IP /32, por lo que se puede omitir el `source-wildcard`.

Entrada ACE|Redes
:---|:---
permit any| Permite todas las redes
permit 172.16.0.0 0.0.255.255|Permite todas las redes en el rango de red 172.16.0.0/16
permit host 192.168.1.1|Permite solo la red 192.168.1.1/32

### Extended ACL:

- Paso 1. Defina la ACL mediante el comando `ip access-list extended {acl-number | acl-name}` y centrara al modo configuración de ACL.
- Paso 2. Configure la entrada ACE específica con el comando `[sequence] {permit | deny} protocol source source-wildcard destination destination-wildcard`. El comportamiento para seleccionar un prefijo de red con una ACL extendida varía según el protocolo IGP (EIGRP, OSPF o IS-IS) o BGP.

### BGP seleccion de red:

Las ACL extendidas reaccionan de forma diferente al coincidir con rutas BGP que al coincidir con rutas IGP.
Los campos de origen coinciden con la porción de red de la ruta, y los campos de destino coinciden con la máscara de red, como se muestra en la Figura 12-5. 
Hasta que llego la introducción de las **prefix-list**, donde antes las ACL extendidas eran el único criterio de coincidencia utilizado con BGP.

![Image Alt]()

Demuestra el concepto de `wildcard` para la máscara de red y subred.

Extended ACL|Coincidencia con la redes
:---|:---
permit ip 10.0.0.0 0.0.0.0 255.255.0.0 0.0.0.0|Solo permite la red 10.0.0.0/16
permit ip 10.0.0.0 0.0.255.0 255.255.255.0 0.0.0.0|Permite cualquier red 10.0.x.0 con un prefijo de /24
permit ip 172.16.0.0 0.0.255.255 255.255.255.0 0.0.0.255|Permite cualquier red 172.16.x.x con un prefijo de /24 a /32
permit ip 172.16.0.0 0.0.255.255 255.255.255.128 0.0.0.127|Permite cualquier red 172.16.x.x con un prefijo de /25 a /32

## Prefix Matching:

Las listas de prefijos proporcionan otro método para identificar redes en un protocolo de enrutamiento. Una lista de prefijos identifica una dirección IP, un prefijo de red o un rango de redes específico y permite la selección de múltiples redes con distintas longitudes de prefijo mediante una especificación de coincidencia de prefijo. 

Se recomienda este método en lugar del de selección de red ACL.

Una especificación de coincidencia de prefijo consta de dos partes: un patrón de bits de orden superior y un recuento de bits de orden superior, que determina los bits de orden superior del patrón de bits que deben coincidir. En cierta documentación, se hace referencia al patrón de bits de orden superior como la dirección o red, y al recuento de bits de orden superior como la longitud del prefijo o la longitud de la máscara.
En la Figura 12-6, la especificación de coincidencia de prefijo muestra el patrón de bits de orden superior 192.168.0.0 y el recuento de bits de orden superior 16 convertidos a binario para mostrar dónde se encuentra el recuento de bits de orden superior. Dado que no se incluyen parámetros de longitud de coincidencia adicionales, el recuento de bits de orden superior es una coincidencia exacta.

![Image Alt]()

La verdadera potencia y flexibilidad residen en el uso de parámetros de longitud coincidente para identificar múltiples redes con longitudes de prefijo específicas con una sola instrucción las cuales son:

- le (Less than): Menor o igual que, <=
- ge (Greater than): Mayor o igual que, >=

Ejmplo aplicado. Donde muestra la especificación de coincidencia de prefijo con el patrón de **High-Order Bit** 10.168.0.0 y un recuento de **High-Order Bit es 13**; la longitud de coincidencia del prefijo debe ser mayor o igual a 24.

![Image Alt]()

EXPLICACION: El prefijo 10.168.0.0/13 no cumple con el parámetro de longitud de coincidencia porque su longitud es menor que el mínimo de 24 bits, mientras que el prefijo 10.168.0.0/24 sí cumple con el parámetro de longitud de coincidencia. El prefijo 10.173.1.0/28 cumple con los requisitos porque los primeros 13 bits coinciden con el patrón de **High-Order Bit** y la longitud del prefijo se encuentra dentro del parámetro de longitud de coincidencia.
El prefijo 10.104.0.0/24 no cumple con los requisitos porque el patrón de **High-Order Bit** no coincide con el recuento de **High-Order Bit**.

La figura 12-8 muestra una especificación de coincidencia de prefijo con el patrón de **High-Order Bit** 10.0.0.0, un conteo de **High-Order Bit** de 8 y una longitud de coincidencia entre 22 y 26.

![Image Alt]()

EXPLICACION: El prefijo 10.0.0.0/8 no coincide porque su longitud es demasiado corta. La red 10.0.0.0/24 sí es válida porque el patrón de bits coincide y la longitud del prefijo está entre 22 y 26. El prefijo 10.0.0.0/30 no coincide porque su longitud es demasiado larga. Cualquier prefijo que comience con 10 en el primer octeto y tenga una longitud entre 22 y 26 coincidirá.

> [!NOTE]
> Para que coincida con una longitud de prefijo _específica_ mayor que el número de **High-Order Bit**, es necesario que el valor `ge` y el valor `le` coincidan.

### Prefix List:

Las listas de prefijos pueden contener varias entradas de especificación que coinciden con el prefijo y que contienen una acción de _permit or deny_.
Las listas de prefijos se procesan en orden secuencial descendente, y el primer prefijo coincidente se procesa con la acción de permiso o denegación adecuada.

- Las listas de prefijos se configuran con el comando de configuración global `ip prefix-list prefix-listname [seq sequence-number] {permit | deny} high-order-bit-pattern/high-order-bit-count
[ge ge-value] [le le-value].`

Si no se proporciona una secuencia, el número de secuencia se incrementa automáticamente en 5, basándose en el número de secuencia más alto. La primera entrada es 5. La secuenciación permite eliminar una entrada específica. Dado que las listas de prefijos no se pueden volver a secuenciar, se recomienda dejar suficiente espacio para la inserción de números de secuencia posteriormente.

# Route Maps: 



# BGP Route Filtering and Manipulation: 



# BGP Communities: 



# Understanding BGP Path Selection: 


