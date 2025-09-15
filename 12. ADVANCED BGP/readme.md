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

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/beb580f8f8ca10746df0c2d17744d2b25cfb39bd/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20MULTI/ADV%20BGP%20MULTI%204.PNG)

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

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/beb580f8f8ca10746df0c2d17744d2b25cfb39bd/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20MULTI/ADV%20BGP%20MULTI%205.PNG)

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

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/beb580f8f8ca10746df0c2d17744d2b25cfb39bd/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20MULTI/ADV%20BGP%20MULTI%206.PNG)

La verdadera potencia y flexibilidad residen en el uso de parámetros de longitud coincidente para identificar múltiples redes con longitudes de prefijo específicas con una sola instrucción las cuales son:

- le (Less than): Menor o igual que, <=
- ge (Greater than): Mayor o igual que, >=

Ejmplo aplicado. Donde muestra la especificación de coincidencia de prefijo con el patrón de **High-Order Bit** 10.168.0.0 y un recuento de **High-Order Bit es 13**; la longitud de coincidencia del prefijo debe ser mayor o igual a 24.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/beb580f8f8ca10746df0c2d17744d2b25cfb39bd/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20MULTI/ADV%20BGP%20MULTI%207.PNG)

EXPLICACION: El prefijo 10.168.0.0/13 no cumple con el parámetro de longitud de coincidencia porque su longitud es menor que el mínimo de 24 bits, mientras que el prefijo 10.168.0.0/24 sí cumple con el parámetro de longitud de coincidencia. El prefijo 10.173.1.0/28 cumple con los requisitos porque los primeros 13 bits coinciden con el patrón de **High-Order Bit** y la longitud del prefijo se encuentra dentro del parámetro de longitud de coincidencia.
El prefijo 10.104.0.0/24 no cumple con los requisitos porque el patrón de **High-Order Bit** no coincide con el recuento de **High-Order Bit**.

La figura 12-8 muestra una especificación de coincidencia de prefijo con el patrón de **High-Order Bit** 10.0.0.0, un conteo de **High-Order Bit** de 8 y una longitud de coincidencia entre 22 y 26.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/beb580f8f8ca10746df0c2d17744d2b25cfb39bd/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20MULTI/ADV%20BGP%20MULTI%208.PNG)

EXPLICACION: El prefijo 10.0.0.0/8 no coincide porque su longitud es demasiado corta. La red 10.0.0.0/24 sí es válida porque el patrón de bits coincide y la longitud del prefijo está entre 22 y 26. El prefijo 10.0.0.0/30 no coincide porque su longitud es demasiado larga. Cualquier prefijo que comience con 10 en el primer octeto y tenga una longitud entre 22 y 26 coincidirá.

> [!NOTE]
> Para que coincida con una longitud de prefijo _específica_ mayor que el número de **High-Order Bit**, es necesario que el valor `ge` y el valor `le` coincidan.

### Prefix List:

Las listas de prefijos pueden contener varias entradas de especificación que coinciden con el prefijo y que contienen una acción de _permit or deny_.
Las listas de prefijos se procesan en orden secuencial descendente, y el primer prefijo coincidente se procesa con la acción de permiso o denegación adecuada.

- Las listas de prefijos se configuran con el comando de configuración global `ip prefix-list prefix-listname [seq sequence-number] {permit | deny} high-order-bit-pattern/high-order-bit-count
[ge ge-value] [le le-value].`

Si no se proporciona una secuencia, el número de secuencia se incrementa automáticamente en 5, basándose en el número de secuencia más alto. La primera entrada es 5. La secuenciación permite eliminar una entrada específica. Dado que las listas de prefijos no se pueden volver a secuenciar, se recomienda dejar suficiente espacio para la inserción de números de secuencia posteriormente.

> [!NOTE]
> IOS XE requiere que el `ge-value` sea mayor que el conteo de **High-Order Bit** y que el `le-value` sea mayor o igual que el `ge-value`:
> - _high-order bit count < ge-value <= le-value_

El Ejemplo 12-1 proporciona un ejemplo de lista de prefijos denominada RFC1918 para todas las redes en el rango de direcciones RFC 1918. Esta lista solo permite la existencia de prefijos /32 en el rango de red 192.168.0.0 y no en ningún otro rango de red de la lista.

La secuencia 5 permite todos los prefijos /32 en el patrón de bits 192.168.0.0/16, la secuencia 10 deniega todos los prefijos /32 en cualquier patrón de bits, y las secuencias 15, 20 y 25 permiten rutas en los rangos de red adecuados. El orden de la secuencia es importante para las dos primeras entradas para garantizar que solo existan prefijos /32 en la red 192.168.0.0 de la lista de prefijos.

### IPV6 Prefix List:

La lógica de coincidencia de prefijos funciona exactamente igual en redes IPv6 que en redes IPv4. Lo más importante es recordar que las redes IPv6 se expresan en hexadecimal y no en decimal al identificar rangos.

- Las listas de prefijos de IPv6 se configuran con el comando de configuración global `ipv6 prefix-list prefix-list-name [seq sequence-number] {permit | deny} high-order-bit-pattern/highorder-bit-count [ge ge-value] [le le-value].` Ejemplo.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/beb580f8f8ca10746df0c2d17744d2b25cfb39bd/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20MULTI/ADV%20BGP%20MULTI%209.PNG)

## Expresiones regulares con (regex):

En ocasiones, el Matching condicional de prefijos de red puede resultar demasiado compleja, por lo que se prefiere identificar todas las rutas de una organización específica. En tal caso, la selección de ruta puede realizarse mediante `BGP AS_Path`.

Las expresiones regulares (regex) se utilizan para analizar la gran cantidad de ASN disponibles (4 294 967 295). Se basan en modificadores de consulta para seleccionar el contenido buscado. La tabla BGP se puede analizar con expresiones regulares mediante el comando `show bgp afi safi regexp regex-pattern`.

Tabla de Modificadores de consultas RegEx
Modificador|Descripcion
:---|:---
_ (guión bajo) | Coincide con un espacio
^ (acento intercalado) | Indica el inicio de una cadena
$ (signo de dólar) | Indica el final de una cadena
[ ] (corchetes) | Coincide con un solo carácter o con una anidación dentro de un rango
\- (guión) | Indica un rango de números entre corchetes
[^] (acento intercalado entre corchetes) | Excluye los caracteres entre corchetes
( ) (paréntesis) | Se utiliza para anidar patrones de búsqueda
\| (barra vertical) | Proporciona la función OR a la consulta
. (punto) | Coincide con un solo carácter, incluido un espacio
\* (asterisco) | Coincide con cero o más caracteres o patrones
\+ (signo más) | Coincide con una o más instancias del carácter o patrón
\? (signo de interrogación) | Coincide con una o ninguna instancia del carácter o patrón

Aprender expresiones regulares puede llevar tiempo, pero las más comunes que se usan en BGP involucran ^, $ y _.

Expresion regular|Significado
:---|:---
`^$` | Rutas de origen local
`permit ^200_` | Solo rutas del AS 200 vecino
`permit _200$` | Solo rutas con origen en el AS 200
`permit _200_` | Solo rutas que pasan por el AS 200
`permit ^[0-9]+ [0-9]+ [0-9]+?` | Rutas con tres o menos entradas AS_Path

> [!NOTE]
> Los servidores públicos, llamados "looking glass", permiten a los usuarios iniciar sesión y ver tablas BGP. La mayoría de estos dispositivos son routers Cisco, pero algunos son de otros proveedores. Estos servidores permiten a los ingenieros de red comprobar si están publicando sus rutas a Internet según lo previsto y ofrecen un excelente método para probar expresiones regulares en la tabla BGP de Internet. Una búsqueda rápida en internet proporcionará listados de sitios web de "looking glass" y servidores de rutas. Recomendamos www.bgp4.as

# Route Maps: 

Los **Route Maps** ofrecen diversas funciones para diversos protocolos de enrutamiento. En su nivel más básico, pueden filtrar redes de forma muy similar a las ACL, pero también proporcionan capacidad adicional mediante la adición o modificación de atributos de red. Para influir en un protocolo de enrutamiento, se debe hacer referencia a un mapa de rutas desde dicho protocolo. Los mapas de rutas son fundamentales para BGP, ya que son el componente principal para modificar una política de enrutamiento única, individualmente para cada vecino.

Un mapa de ruta consta de cuatro componentes:
- Sequence number: Indica el orden de procesamiento del mapa de ruta.
- Conditional matching criteria: Identifica las características del prefijo (el propio prefijo, el atributo de ruta BGP, el siguiente salto, etc.) para una secuencia específica.
- Processing action: Permite o deniega el prefijo.
- Optional action: Permite manipulaciones, según cómo se referencia el mapa de ruta en el enrutador. Las acciones pueden incluir la modificación, adición o eliminación de características de ruta.

Un mapa de rutas utiliza el comando `syntax route-map route-map-name [permit | deny] [sequence-number]`. Las siguientes reglas se aplican a las sentencias de mapa de rutas:
- Si no se proporciona una acción de procesamiento, se utiliza el valor predeterminado `permit`.
- Si no se proporciona un número de secuencia, este se incrementa automáticamente en 10.
- Si no se incluye una sentencia coincidente, se asocia una coincidencia implícita de todos los prefijos.
- El procesamiento dentro de un mapa de rutas se detiene después de que se hayan procesado todas las acciones opcionales (si están configuradas) tras cumplir un criterio de coincidencia condicional.
- Se asocia una denegación o eliminación implícita para los prefijos que no están asociados con una acción `permit`.

Proporciona un ejemplo de Route Map para ilustrar los cuatro componentes de un mapa de rutas mostrados anteriormente. El criterio de Matching condicional se basa en los rangos de red especificados en una ACL.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/beb580f8f8ca10746df0c2d17744d2b25cfb39bd/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RM/ADV%20BGP%20RM%201.PNG)

> [!WARNING]
> Al eliminar una instrucción `route-map` específica, incluya el número de secuencia para evitar eliminar el `route-map` completo.

## Conditional Matching:

Una vez explicados los componentes y el orden de procesamiento de un mapa de rutas, esta parte explica de cómo se puede establecer el **MATCH** de un prefijo. La Tabla 12-6 muestra la sintaxis para comandos comando.

Comando Match|Descripcion
:---|:---
`match as-path acl-number`| Selecciona prefijos según una consulta de expresiones regulares para aislar el ASN en la ruta AS del atributo de ruta BGP (PA). Las ACL de la ruta AS están numeradas del 1 al 500. Este comando permite múltiples variables de match.
`match ip address {acl-number \| acl-name}`| Selecciona prefijos según los criterios de selección de red definidos en la ACL. Este comando permite múltiples variables de match.
`match ip address prefix-list prefix-list-name`| Selecciona prefijos según los criterios de selección. Este comando permite múltiples variables de match.
`match local-preference local-preference`| Selecciona prefijos según el atributo BGP "local-preference". Este comando permite múltiples variables de match.
`match metric {1-4294967295 \| external 1-4294967295} [+- desviación]`| Selecciona prefijos según una métrica que puede ser exacta, un rango o estar dentro de una desviación aceptable.
`match tag tag-value`| Selecciona prefijos según una etiqueta numérica (0 a 4294967295) configurada por otro enrutador. Este comando permite múltiples variables de match.
`match community {1-500 \| community-list-name [exact]}`| Selecciona prefijos según una ACL de lista comunitaria. Las ACL de lista comunitaria están numeradas del 1 al 500. Este comando permite múltiples variables de match.

### Multiple Conditional Match Conditions

Si se configuran varias variables (ACL, prefix-lists, tags, etc.) para una secuencia de **Route map** específica, solo una variable debe coincidir para que el prefijo sea válido. La lógica booleana utiliza un operador OR para esta configuración.
En el Ejemplo 12-4, la secuencia 10 requiere que un prefijo pase ACL-ONE o ACL-TWO. Observe que la secuencia 20 no tiene una declaración de coincidencia, por lo que todos los prefijos que no se pasen en la secuencia 10 serán válidos y se denegarán.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/beb580f8f8ca10746df0c2d17744d2b25cfb39bd/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RM/ADV%20BGP%20RM%202.PNG)

Si se configuran varias opciones de coincidencia para una **Sequence de Route Map específica**, ambas deben cumplirse para que el prefijo sea apto para esa secuencia. La lógica booleana utiliza el operador AND para esta configuración.
En el Ejemplo 12-5, la secuencia 10 requiere que el prefijo coincida con ACL-ONE y que la métrica tenga un valor entre 500 y 600. Si el prefijo no cumple con ambas opciones de coincidencia, no cumple con las condiciones para la secuencia 10 y se deniega porque no existe otra secuencia con una acción de permiso.

Ejemplo 12-5 de mapa de ruta con opciones de Multiple Match
```
route-map EXAMPLE permit 10
  match ip address ACL-ONE
  match metric 550 +- 50
```

### Complex Matching

Algunos ingenieros de red consideran que los mapas de rutas son demasiado complejos si **conditional matching criteria** (es decir, ACL, ACL de ruta AS o lista de prefijos) contienen una sentencia de denegación. El ejemplo 12-6 muestra una configuración donde la ACL utiliza una sentencia de denegación para el rango de red 172.16.1.0/24.
Las configuraciones de lectura como esta deben seguir primero el orden de secuencia y después **conditional matching criteria second**. Solo después de un match se deben utilizar la acción de procesamiento y la acción opcional. El matching de una sentencia denegada en conditional match criteria excluye la ruta de esa secuencia en el mapa de rutas.

El prefijo 172.16.1.0/24 es denegado por ACL-ONE, lo que implica que no hay coincidencia en las secuencias 10 y 20; por lo tanto, no se requiere la acción de procesamiento (permitir o denegar).
La ​​secuencia 30 no contiene una cláusula de coincidencia, por lo que se permiten las rutas restantes.
El prefijo 172.16.1.0/24 pasaría a la secuencia 30 con la métrica establecida en 20. El prefijo 172.16.2.0/24 coincidiría con ACL-ONE y pasaría a la secuencia 10.

Example 12-6 Complex Matching Route Maps
````
ip access-list standard ACL-ONE
  deny 172.16.1.0 0.0.0.255
  permit 172.16.0.0 0.0.255.255
route-map EXAMPLE permit 10
  match ip address ACL-ONE
!
route-map EXAMPLE deny 20
  match ip address ACL-ONE
!
route-map EXAMPLE permit 30
  set metric 20
````
> [!NOTE]
> Los Route Map se procesan siguiendo un orden de evaluación específico: la secuencia, conditional matching criteria, la acción de procesamiento y la _acción opcional_, en ese orden. Las sentencias de denegación del componente de coincidencia se aíslan de la acción de secuencia del mapa de ruta.

## Acciones opcionales:

Accion Establecida|Descripcion
:---|:---
`set as-path prepend {as-number-pattern \|last-as 1-10}`|Antepone la ruta del AS para el prefijo de red con el patrón especificado o a partir de múltiples iteraciones de un AS vecino.
`set ip next-hop {ip-address \| peer-address \|self}`|Establece la dirección IP del siguiente salto para cualquier prefijo coincidente. La manipulación dinámica de BGP utiliza las palabras clave `peer-address` o `self`.
`set local-preference 0-4294967295`| Establece la preferencia local del PA de BGP.
`set metric {+value \| -value \| value} (donde los parámetros de valor son 0–4294967295)`|Modifica la métrica existente o establece la métrica de una ruta.
`set origin {igp \| incomplete}` | Establece el origen del PA de BGP.
`set tag tag-value`| Establece una etiqueta numérica (0\–4294967295) para la identificación de redes por otros enrutadores.
`set weight 0–65535`| Establece el peso de BGP PA.
`set community bgp-community [additive]`| Establece las comunidades BGP PA.

## Palabra clave CONTINUAR:

El comportamiento predeterminado de **ROUTE MAP** procesa las secuencias del mapa de rutas en orden y tras la primera coincidencia, ejecuta la acción de procesamiento, realiza cualquier acción opcional (si es posible) y detiene el procesamiento. Esto evita que se procesen varias secuencias del mapa de rutas. Añadir la palabra clave `"continue"` a un _route map_ permite que este continúe procesando otras secuencias del mapa de rutas. El ejemplo 12-7 proporciona una configuración básica. El prefijo de red 192.168.1.1 coincide en las secuencias 10, 20 y 30. Dado que se añadió la palabra clave "continue" a la secuencia 10, la secuencia 20 se procesa, pero la secuencia 30 no, ya que no había un comando `continue` en la secuencia 20.

Example 12-7 Route Map con comando `continue`
````
ip access-list standard ACL-ONE
  permit 192.168.1.1 0.0.0.0
  permit 172.16.0.0 0.0.255.255
!
ip access-list standard ACL-TWO
  permit 192.168.1.1 0.0.0.0
  permit 172.31.0.0 0.0.255.255
!
route-map EXAMPLE permit 10
  match ip address ACL-ONE
  set metric 20
  continue <------------------------------------
!
route-map EXAMPLE permit 20
  match ip address ACL-TWO
  set ip next-hop 10.12.1.1
  ## NO HAY CONTINUE, ENTONCES LA SECUENCIA 30 NO FUNCIONA ## <------------------------------------
!
route-map EXAMPLE permit 30
  set ip next-hop 10.13.1.3
````
> [!NOTE]
> Comando `Continue` no es muy utilizado por agregar complejidad al querer solucionar problemas.

# BGP Route Filtering and Manipulation: 

El filtrado de rutas es un método para identificar **selectivamente** las rutas anunciadas o recibidas de routers vecinos. El filtrado de rutas puede utilizarse para manipular los flujos de tráfico, reducir el uso de memoria o mejorar la seguridad. Por ejemplo, es común que los ISP implementen filtros de rutas en los puntos BGP con los clientes. Garantizar que solo se permitan las rutas del cliente en el enlace de punto BGP evita que el cliente se convierta accidentalmente en un AS de tránsito en Internet.
La Figura 12-9 muestra la lógica completa del procesamiento de rutas BGP.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/96c20eb0b25dee9e71a89d1c72cf53dc19636f58/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RFM/ADV%20BGP%20RFM%201.PNG)

> [!NOTE]
> Tenga en cuenta que las políticas de enrutamiento se aplican en la recepción de rutas entrantes y en el anuncio de rutas salientes.

IOS XE ofrece cuatro métodos para filtrar rutas entrantes o salientes para un par BGP específico. Estos métodos pueden usarse individualmente o simultáneamente con otros:

- Lista de distribución: Una lista de distribución implica el filtrado de prefijos de red según una ACL estándar o extendida. Una denegación implícita se asocia a cualquier prefijo que no esta permitido.
- Prefix list: Las especificaciones de Prefix-matching en una lista, permiten o deniegan prefijos de red de forma descendente, similar a una ACL. Una denegación implícita se asocia a cualquier prefijo que no esta permitido.
- ACL/filtrado de rutas de AS: Los comandos Regex en una lista permiten permitir o denegar un prefijo de red según los valores actuales de la ruta de AS. Una denegación implícita se asocia a cualquier prefijo que no esta permitido.
- Route maps: Estos proporcionan un método de conditional Matching con diversos atributos de prefijo y permiten realizar diversas acciones. Las acciones pueden ser simplemente permitir o denegar, o incluir la modificación de los atributos de la ruta BGP. Una denegación implícita se asocia a cualquier prefijo que no esta permitido.

> [!NOTE]
> Un vecino BGP no puede usar una lista de distribución y una Prefix-list al mismo tiempo en la misma dirección (inbound or outbound).

Las siguientes secciones explican cada una de estas técnicas de filtrado con más detalle. Imaginemos un escenario simple con R1 (AS 65100) que tiene un único punto eBGP con R2 (AS 65200), que a su vez puede otro punto BGP con otros sistemas autónomos (como AS 65300). La parte relevante de la topología es que R1 se relaciona con R2 y se centra en la tabla BGP de R1, como se muestra en el Ejemplo 12-8, con énfasis en el prefijo de red y la ruta del AS.

Ejemplo 12-8 Tabla BGP de referencia EJEMPLO GRANDE:

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/96c20eb0b25dee9e71a89d1c72cf53dc19636f58/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RFM/ADV%20BGP%20RFM%202.PNG)

## Filtrado con Listas de Distribucion:

Las listas de distribución filtran rutas por base neighbor-by-neighbor, utilizando ACL estándar o extendidas. Para configurar una lista de distribución, se requiere el comando de configuración de familia de direcciones BGP `neighbor ip-address deliver-list {acl-number | acl-name} {in|out}`. Recuerde que las ACL extendidas para BGP utilizan los campos de origen para comparar la porción de red y los campos de destino para comparar la máscara de red.

El ejemplo 12-9 muestra la configuración BGP de R1, que demuestra el filtrado con listas de distribución. La configuración utiliza una ACL extendida denominada ACL-ALLOW. La primera permite cualquier prefijo que comience en el rango 192.168.0.0 a 192.168.255.255 con una longitud de prefijo de solo /32 (hosts). La segunda entrada permite prefijos que contienen el patrón 100.64.x.0 con una longitud de prefijo de /25 para demostrar las capacidades de `Wildcard` de una ACL extendida con BGP. La lista de distribución se asocia entonces con la sesión BGP de R2.

Ejemplo 12-9 Configuración de lista de distribución BGP
````
R1
ip access-list extended ACL-ALLOW <------------------------------------
permit ip 192.168.0.0 0.0.255.255 host 255.255.255.255
permit ip 100.64.0.0 0.0.255.0 host 255.255.255.128
!
router bgp 65100
  neighbor 10.12.1.2 remote-as 65200
  address-family ipv4
    neighbor 10.12.1.2 activate
    neighbor 10.12.1.2 distribute-list ACL-ALLOW in <------------------------------------
````
El ejemplo 12-10 muestra la tabla BGP de R1. R1 inyecta dos rutas locales en la tabla BGP (10.12.1.0/24 y 192.168.1.1/32). Las dos rutas loopback de R2 (AS 65200) y R3 (AS 65300) se permiten porque se encuentran dentro de la primera entrada ACL-ALLOW, y se aceptan dos de las rutas que coinciden con el patrón 100.64.x.0 (100.64.2.0/25 y 100.64.3.0/25). La ruta 100.64.2.192/26 se rechaza porque la longitud del prefijo no coincide con la segunda entrada ACL-ALLOW.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/96c20eb0b25dee9e71a89d1c72cf53dc19636f58/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RFM/ADV%20BGP%20RFM%203.PNG)

## Filtrado por Prefix-List

Las listas de prefijos filtran rutas por vecino. Para configurar una prefix-list, se utiliza el comando de configuración de la familia de direcciones BGP `neighbor ip-address prefix-list prefix-list-name {in | out}`. Para demostrar el uso de una prefix-list, Se utiliza la misma lista de prefijos del Ejemplo 12-1 y se aplicará en el peering de R1 a R2 (AS 65200).
El Ejemplo 12-11 muestra la configuración de la prefix-list y su aplicación a R2.
````
R1# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
R1(config)# ip prefix-list RFC1918 seq 5 permit 192.168.0.0/16 ge 32     ||||
R1(config)# ip prefix-list RFC1918 seq 10 deny 0.0.0.0/0 ge 32           ||||
R1(config)# ip prefix-list RFC1918 seq 15 permit 10.0.0.0/8 le 32        ||||
R1(config)# ip prefix-list RFC1918 seq 20 permit 172.16.0.0/12 le 32     ||||
R1(config)# ip prefix-list RFC1918 seq 25 permit 192.168.0.0/16 le 32    ||||
R1(config)# router bgp 65100
R1(config-router)# address-family ipv4 unicast
R1(config-router-af)# neighbor 10.12.1.2 prefix-list RFC1918 in          ||||
````
Una vez aplicada la Prefix-List, se puede examinar la tabla BGP en el R1, como se muestra en el Ejemplo 12-12. Observe que las rutas 100.64.2.0/25, 100.64.2.192/26 y 100.64.3.0/25 se filtraron porque no cumplían con los criterios de coincidencia de la lista de prefijos. Se puede consultar el Ejemplo 12-8 para identificar las rutas antes de aplicar la lista de prefijos BGP.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/96c20eb0b25dee9e71a89d1c72cf53dc19636f58/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RFM/ADV%20BGP%20RFM%204.PNG)

## Filtrado de ACL de AS_Path:

La selección de rutas de un vecino BGP mediante AS_Path requiere la definición de una `lista de control de acceso AS_Path` (ACL AS_Path). Las expresiones regulares, presentadas anteriormente, son un componente del filtrado AS_Path. El ejemplo 12-13 muestra las rutas que R2 (AS 65200) anuncia a R1 (AS 65100).

Muestra las rutas que R2 (AS 65200) está anunciando a R1 (AS 65100).

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/96c20eb0b25dee9e71a89d1c72cf53dc19636f58/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RFM/ADV%20BGP%20RFM%205.PNG)

R2 anuncia las rutas aprendidas de R3 (AS 65300) a R1. En esencia, **R2 proporciona conectividad de tránsito entre los sistemas autónomos**. Si se tratara de una conexión a Internet y R2 fuera una empresa, no querría anunciar las rutas aprendidas de otros AS. Se recomienda usar una lista de acceso AS_Path para restringir el anuncio únicamente de rutas AS 65200.

El procesamiento se realiza en orden descendente secuencial, y la primera match válida se procesa con la acción de _permit or deny_ correspondiente. Existe una denegación implícita al final de la ACL de AS_Path. IOS admite hasta 500 ACL de AS_Path y utiliza el comando `ip as-path access-list acl-number {deny | permit} regex-query` para crear una ACL de AS_Path. La ACL se aplica luego con el comando `neighbor ip-address filter-list acl-number {in|out}`. El ejemplo 12-14 muestra la configuración en R2 mediante una ACL AS_Path para restringir el tráfico únicamente al tráfico de origen local, utilizando el patrón de expresión regular ^$ (consulte la Tabla Regex). Para garantizar la integridad, la ACL AS_Path se aplica a todas las vecindades eBGP.

Example 12-14 AS Path Access List Configuracion
````
R2
ip as-path access-list 1 permit ^$ <------------------------------------
!
router bgp 65200
  neighbor 10.12.1.1 remote-as 65100
  neighbor 10.23.1.3 remote-as 65300
  address-family ipv4 unicast
    neighbor 10.12.1.1 activate
    neighbor 10.23.1.3 activate
    neighbor 10.12.1.1 filter-list 1 out <------------------------------------
    neighbor 10.23.1.3 filter-list 1 out <------------------------------------
````
Ahora que se ha aplicado la ACL AS_Path, se pueden volver a comprobar las rutas anunciadas.
El ejemplo 12-15 muestra las rutas que se anuncian a R1. Observe que ninguna de las rutas tiene una ACL AS_Path, **lo que confirma que solo las rutas de origen local se anuncian externamente**. Se puede consultar el ejemplo 12-13 para identificar las rutas antes de aplicar la ACL AS_Path de BGP.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/96c20eb0b25dee9e71a89d1c72cf53dc19636f58/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RFM/ADV%20BGP%20RFM%206.PNG)

## Filtrado por Route Maps:

Como se explicó anteriormente, los Route Map ofrecen una funcionalidad adicional al filtrado puro. Tambien proporcionan un método para manipular los atributos de ruta BGP. Los Route Map se aplican por vecino BGP para las rutas anunciadas o recibidas. Se puede usar un Route Map diferente para cada dirección. El Route Map se asocia a un vecino BGP mediante el comando `neighbor ip-address route-map route-map-name {in|out}` bajo la Address Family específica.
El ejemplo 12-16 muestra la tabla BGP de R1, que se utiliza aquí para demostrar la potencia de un mapa de rutas.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/96c20eb0b25dee9e71a89d1c72cf53dc19636f58/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RFM/ADV%20BGP%20RFM%207.PNG)

Los Route Maps también permiten múltiples pasos de procesamiento. Para demostrar este concepto, nuestro Route Map constará de cuatro pasos:
- Denegar cualquier ruta dentro del rango de la red 192.168.0.0/16 mediante una Prefix-List.
- Coincidir con cualquier ruta originada en el AS 65200 dentro del rango de la red 100.64.0.0/10 y establecer la preferencia local BGP en 222.
- Coincidir con cualquier ruta originada en el AS 65200 que no coincida con el paso 2 y establecer el peso BGP en 23456.
- Permitir el procesamiento de todas las demás rutas.
El ejemplo 12-17 muestra la configuración de R1, donde se hace referencia a múltiples Prefix-List junto con una ACL de ruta del AS.

Ejemplo 12-17 Configuración del Route Map de R1 para rutas AS 65200 entrantes:
````
R1
ip prefix-list FIRST-RFC1918 permit 192.168.0.0/16 le 32 <------------------------------------
ip as-path access-list 1 permit _65200$ <------------------------------------
ip prefix-list SECOND-CGNAT permit 100.64.0.0/10 le 32
!
route-map AS65200IN deny 10
  description Deny RFC1918 192.168.0.0/16 routes via Prefix List Matching
  match ip address prefix-list FIRST-RFC1918
!
route-map AS65200IN permit 20
  description Change local preference for AS65200 originated route in 100.64.x.x/10
  match ip address prefix-list SECOND-CGNAT
  match as-path 1
  set local-preference 222
!
route-map AS65200IN permit 30
  description Change the weight for AS65200 originated routes
  match as-path 1
  set weight 23456
!
route-map AS65200IN permit 40
  description Permit all other routes un-modified
!
router bgp 65100
  neighbor 10.12.1.2 remote-as 65200
  address-family ipv4 unicast
    neighbor 10.12.1.2 activate
    neighbor 10.12.1.2 route-map AS65200IN in
````
El ejemplo 12-18 muestra la tabla BGP de R1. Se han producido las siguientes acciones:
- Se descartaron las rutas 192.168.2.2/32 y 192.168.3.3/32. La ruta 192.168.1.1/32 es una ruta generada localmente.
- Se modificó la preferencia local de las rutas 100.64.2.0/25 y 100.64.2.192/26 a 222 porque se originaron en AS 65200 y se encuentran dentro del rango de red 100.64.0.0/10.
- A las rutas 10.12.1.0/24 y 10.23.1.0/24 de R2 se les asignó el peso de atributo BGP localmente significativo de 23456.
- Se aceptaron todas las demás rutas sin modificarlas.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/96c20eb0b25dee9e71a89d1c72cf53dc19636f58/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20RFM/ADV%20BGP%20RFM%208.PNG)

> [!NOTE]
> Se considera una buena práctica utilizar una política de ruta diferente para los prefijos entrantes y salientes para cada vecino BGP.

## Borrar y Limpiar conexiones BGP:

Dependiendo del cambio en la técnica de manipulación de rutas BGP, es posible que sea necesario actualizar una sesión BGP para que surta efecto. BGP admite dos métodos para borrar una sesión BGP.
El primer método es un restablecimiento completo, que cierra la sesión BGP, elimina las rutas BGP del par y es el más disruptivo. El segundo método es un restablecimiento parcial, que invalida la caché BGP y solicita un anuncio completo a su par BGP.

Los enrutadores inician un restablecimiento completo con el comando `clear ip bgp ip-address [soft]` y un restablecimiento parcial mediante la palabra clave opcional `soft`. Todas las sesiones BGP activas se pueden borrar utilizando un asterisco * en lugar de la dirección IP del par.

Cuando cambia una política BGP, la tabla BGP debe procesarse de nuevo para que los vecinos puedan ser notificados en consecuencia. Las rutas recibidas por un par BGP deben procesarse de nuevo. Si la sesión BGP admite la actualización de rutas, el par vuelve a anunciar (actualiza) los prefijos al enrutador solicitante, lo que permite que la política de entrada se procese con los nuevos cambios. La actualización de rutas se negocia para cada familia de direcciones al establecer la sesión.

Al realizar un reinicio soft en sesiones que admiten la actualización de rutas, se inicia una actualización de ruta. Los reinicios soft se pueden realizar para una Address-family específica con el comando `clear bgp afi safi {ip-address|*} soft [in | out]`. Los reinicios soft reducen la cantidad de rutas que deben intercambiarse si se configuran varias Address-family con un solo punto BGP.
Los cambios en las políticas de enrutamiento de salida utilizan la palabra clave opcional `out`, y los cambios en las políticas de enrutamiento de entrada utilizan la palabra clave opcional `in`. Puede usar un * en lugar de especificar la dirección IP de un par para realizar esa acción para todos los pares BGP.

# BGP Communities: 

Las comunidades BGP proporcionan capacidad adicional para etiquetar rutas y modificar la política de enrutamiento BGP en enrutadores de subida y bajada. Las comunidades BGP se pueden añadir, eliminar o modificar selectivamente en cada atributo a medida que una ruta viaja de un enrutador a otro.
Las comunidades BGP son un atributo de ruta BGP transitivo opcional que puede atravesar un sistema autónomo (AS) a otro. Una comunidad BGP es un número de 32 bits que se puede incluir en una ruta. Una comunidad BGP se puede mostrar como un número completo de 32 bits (0-4294967295) o como dos números de 16 bits (0-65535):(0-65535), comúnmente conocido como nuevo formato.

Las comunidades BGP privadas siguen una convención particular: los primeros 16 bits representan el sistema autónomo (AS) de origen de la comunidad y los segundos 16 bits representan un patrón definido por dicho sistema. Un patrón de comunidad BGP privada puede variar entre organizaciones, no necesita registro y puede indicar la ubicación geográfica de un AS, así como un método de anuncio de ruta en otro.

En 2006, la RFC 4360 amplió las capacidades de las comunidades BGP al proporcionar un formato extendido. Las comunidades BGP extendidas proporcionan estructura para diversas clases de información y se utilizan comúnmente para servicios VPN. La RFC 8092 admite comunidades de más de 32 bits (que quedan fuera del alcance de este libro) (INVESTIGACION FUTURA).

## Wel-Known Communities: 

RFC 1997 define un conjunto de comunidades globales (conocidas como comunidades bien conocidas) que se encuentran dentro del rango de comunidades 4.294.901.760 (0xFFFF0000) a 4.294.967.295 (0xFFFFFFFF).
Todos los enrutadores capaces de enviar/recibir comunidades BGP deben implementar comunidades bien conocidas. Se presentan cuatro comunidades bien conocidas comunes:

- Internet: Esta es una comunidad **estandarizada** para identificar rutas que deben anunciarse en Internet. En redes más grandes que implementan BGP en el núcleo, las rutas anunciadas deben anunciarse en Internet y deben tener esta comunidad configurada. Esto permite que los **routers BGP de borde** solo permitan el anuncio de rutas BGP con la comunidad de Internet. El filtrado no es automático, pero puede realizarse con un mapa de rutas de salida.
- No_Advertise: Las rutas con esta comunidad no se anunciarán a ningún par BGP (iBGP o eBGP).
- Local-AS: Las rutas con esta comunidad no se anuncian a un par eBGP, pero sí a los pares de la **confederación BGP**. Los pares de la **confederación BGP** _quedan fuera del alcance del examen_.
- No_Export: Cuando se recibe una ruta con esta comunidad, no se anuncia a ningún par eBGP. Las rutas con esta comunidad sí se pueden anunciar a los pares iBGP.

## Habilitación del soporte de la comunidad BGP:

Los routers IOS XE no anuncian comunidades BGP a sus puntos por defecto. Las comunidades se habilitan individualmente con el comando de configuración de la Address-family BGP `neighbor ip-address send-community [standard | extended | both]` en la configuración de la Address-family del vecino. Si no se especifica una palabra clave, se envían comunidades `Standard` por defecto.
Los routers IOS XE pueden mostrar las comunidades en un nuevo formato, más fácil de leer, con el comando de configuración global `ip bgp-community new-format`. El ejemplo 12-19 muestra primero la comunidad BGP en formato decimal, seguida del nuevo formato.

Ejemplo 12-19 Formatos de la comunidad BGP
```
! Decimal Format <------------------------------------
R3# show bgp 192.168.1.1
! Output omitted for brevity
BGP routing table entry for 192.168.1.1/32, version 6
Community: 6553602 6577023 <------------------------------------
``
! New-Format <------------------------------------
R3# show bgp 192.168.1.1
! Output omitted for brevity
BGP routing table entry for 192.168.1.1/32, version 6
Community: 100:2 100:23423 <------------------------------------
```
## Conditionally matching para Comunidades BGP:

Permite la selección de rutas según las comunidades BGP dentro de los atributos de ruta, lo que permite un procesamiento selectivo en los Route Map. El ejemplo 12-20 muestra la tabla BGP de R1, que ha recibido múltiples rutas de R2 (AS 65200).

Ejemplo 12-20 Rutas BGP desde R2 (AS 65200)
````
R1# show bgp ipv4 unicast | begin Network
Network             Next Hop    Metric   LocPrf   Weight   Path
*> 10.1.1.0/24      0.0.0.0           0            32768   ?
*  10.12.1.0/24     10.12.1.2        22                0   65200 ? <---------------
*>                  0.0.0.0           0            32768   ?
*> 10.23.1.0/24     10.12.1.2       333                0   65200 ? <---------------
*> 192.168.1.1/32   0.0.0.0           0            32768   ?
*> 192.168.2.2/32   10.12.1.2        22                0   65200 ? <---------------
*> 192.168.3.3/32   10.12.1.2      3333                0   65200 65300 ?
````
En este ejemplo, supongamos que desea realizar una conditionally match para una comunidad específica. La tabla BGP completa se puede mostrar con el comando `show bgp afi safi detail` y, a continuación, puede seleccionar manualmente una ruta con una comunidad específica. Sin embargo, si se conoce la comunidad BGP, todas las rutas que coinciden con ella se pueden mostrar con el comando `show bgp afi safi community community`, como se muestra en el Ejemplo 12-21.

Ejemplo 12-21 Visualización de las rutas BGP con una comunidad específica
````
R1# show bgp ipv4 unicast community 333:333 | begin Network <--------------- (community 333:333)
    Network       Next Hop   Metric   LocPrf   Weight Path
*>  10.23.1.0/24  10.12.1.2     333                 0 65200 ?
````
El ejemplo 12-22 muestra la entrada de ruta explícita para la red 10.23.1.0/24 y todos los atributos de ruta BGP. Observe que se agregan dos comunidades BGP (333:333 y 65300:333) a la ruta.

Ejemplo 12-22 Visualización de atributos de ruta BGP para la red 10.23.1.0/24
````
R1# show ip bgp 10.23.1.0/24
BGP routing table entry for 10.23.1.0/24, version 15
Paths: (1 available, best #1, table default)
  Not advertised to any peer
  Refresh Epoch 3
  65200
    10.12.1.2 from 10.12.1.2 (192.168.2.2)
      Origin incomplete, metric 333, localpref 100, valid, external, best
      Community: 333:333 65300:333 <---------------
      rx pathid: 0, tx pathid: 0x0
````
La coincidencia condicional requiere la creación de una lista de comunidades con una estructura similar a la de una ACL, que puede ser estándar o expandida, y que se puede referenciar por número o nombre.
Las listas de comunidades estándar se numeran del 1 al 99 y coinciden con comunidades conocidas o con un número de comunidad privado (as-number:16 bit number). Las listas de comunidades expandidas se numeran del 100 al 500 y utilizan patrones de expresiones regulares.
La sintaxis de configuración para una lista de comunidades es `ip community-list {1-500 | standard listname | expanded list-name} {permit | deny} community-pattern`. Después de definir la lista de comunidades, se referencia en el mapa de rutas con el comando `match community {1-500 | community-list-name [exact]}`.

> [!NOTE]
> Cuando varias comunidades están en la misma declaración de lista de comunidades IP, todas las comunidades para esa declaración deben existir en la ruta. Si solo se requiere una de varias comunidades, se pueden usar varias declaraciones de lista de comunidades IP.

El ejemplo 12-23 demuestra la creación de una lista de comunidades BGP que coincide con la comunidad 333:333. Esta lista se utiliza en la primera secuencia de `route-map COMMUNITY-CHECK`, que deniega cualquier ruta con esa comunidad. La segunda secuencia de `route-map` permite todas las demás rutas BGP y establece el peso BGP (localmente significativo) en 111. El mapa de rutas se aplica entonces en las rutas entrantes anunciadas de R2 a R1.

Ejemplo 12-23 Coincidencia condicional de comunidades BGP
````
R1
ip community-list 100 permit 333:333
!
route-map COMMUNITY-CHECK deny 10
  description Block Routes with Community 333:333 in it
  match community 100
route-map COMMUNITY-CHECK permit 20
  description Allow routes with either community in it
  set weight 111
!
router bgp 65100
  address-family ipv4 unicast
    neighbor 10.12.1.2 route-map COMMUNITY-CHECK in
````
El ejemplo 12-24 muestra la tabla BGP después de aplicar el mapa de rutas al vecino.
La ruta 10.23.1.0/24 se descartó y todas las demás rutas aprendidas del AS 65200 tenían el peso BGP establecido en 111.
````
R1# show bgp ipv4 unicast | begin Network
   Network         Next Hop   Metric  LocPrf  Weight  Path
*> 10.1.1.0/24     0.0.0.0         0           32768  ?
*  10.12.1.0/24    10.12.1.2      22    -------> 111 65200 ?
*>                 0.0.0.0         0           32768 ?
*> 192.168.1.1/32  0.0.0.0         0           32768 ?
*> 192.168.2.2/32  10.12.1.2      22    -------> 111 65200 ?
*> 192.168.3.3/32  10.12.1.2    3333    -------> 111 65200 65300 ?
````
## Configuración de comunidades BGP privadas:

Una comunidad BGP privada se configura en un mapa de rutas con el comando `set community bgpcommunity [additive]`. De forma predeterminada, al configurar una comunidad, las comunidades existentes se sobrescriben, pero se pueden conservar mediante la palabra clave opcional `additive`.
El ejemplo 12-25 muestra las entradas de la tabla BGP para la ruta 10.23.1.0/24, que tiene las comunidades BGP 333:333 y 65300:333. La ruta 10.3.3.0/24 tiene la comunidad 65300:300.

Ejemplo 12-25 Visualización de las comunidades BGP para dos prefijos de red:
````
R1# show bgp ipv4 unicast 10.23.1.0/24
! Output omitted for brevity
BGP routing table entry for 10.23.1.0/24, version 15
  65200
    10.12.1.2 from 10.12.1.2 (192.168.2.2)
      Origin incomplete, metric 333, localpref 100, valid, external, best
      Community: 333:333 65300:333 <--------------------
----------------------------------------------------------------------------
R1# show bgp ipv4 unicast 10.3.3.0/24
! Output omitted for brevity
BGP routing table entry for 10.3.3.0/24, version 12
  65200 65300 3003
    10.12.1.2 from 10.12.1.2 (192.168.2.2)
      Origin incomplete, metric 33, localpref 100, valid, external, best
      Community: 65300:300 <--------------------
````
El ejemplo 12-26 muestra la configuración donde la comunidad BGP se establece en 10:23 para la ruta 10.23.1.0/24. No se utiliza la palabra clave `additive`, por lo que los valores de comunidad anteriores 333:333 y 65300:333 se sobrescriben con la comunidad 10:23. La ruta 10.3.3.0/24 tiene las comunidades 3:0, 3:3 y 10:10 añadidas a las comunidades existentes. El mapa de rutas se asocia entonces a R2 (AS 65200).

Ejemplo 12-26 Configuración de la comunidad BGP privada
````
ip prefix-list PREFIX10.23.1.0 seq 5 permit 10.23.1.0/24
ip prefix-list PREFIX10.3.3.0 seq 5 permit 10.3.3.0/24
!
route-map SET-COMMUNITY permit 10
  match ip address prefix-list PREFIX10.23.1.0
  set community 10:23
route-map SET-COMMUNITY permit 20
  match ip address prefix-list PREFIX10.3.3.0
  set community 3:0 3:3 10:10 additive <--------------------
route-map SET-COMMUNITY permit 30
!
router bgp 65100
  address-family ipv4
    neighbor 10.12.1.2 route-map SET-COMMUNITY in
````
Una vez aplicado el mapa de rutas y actualizadas las rutas, se pueden examinar los atributos de ruta, como se muestra en el Ejemplo 12-27. Como se anticipó, las comunidades BGP anteriores se eliminaron para la red 10.23.1.0/24, pero se mantuvieron para la red 10.3.3.0/24.

Ejemplo 12-27 Verificación de cambios en la comunidad BGP
````
R1# show bgp ipv4 unicast 10.23.1.0/24
! Output omitted for brevity
BGP routing table entry for 10.23.1.0/24, version 22
  65200
    10.12.1.2 from 10.12.1.2 (192.168.2.2)
      Origin incomplete, metric 333, localpref 100, valid, external, best
      Community: 10:23 <--------------------
----------------------------------------------------------------------------
R1# show bgp ipv4 unicast 10.3.3.0/24
BGP routing table entry for 10.3.3.0/24, version 20
  65200 65300 3003
    10.12.1.2 from 10.12.1.2 (192.168.2.2)
      Origin incomplete, metric 33, localpref 100, valid, external, best
      Community: 3:0 3:3 10:10 65300:300 <-------------------- (3:0 3:3 10:10)
````

# Understanding BGP Path Selection: 

El algoritmo de selección de la mejor ruta de BGP influye en la forma en que el tráfico entra o sale de un sistema autónomo (AS). Algunas configuraciones de router modifican los atributos de BGP para influir en el tráfico entrante, saliente o entrante y saliente, según los requisitos de diseño de la red. Muchos ingenieros de red no comprenden la selección de la mejor ruta de BGP, lo que a menudo puede resultar en un enrutamiento deficiente. Esta sección explica la lógica que utiliza un router que utiliza BGP al reenviar paquetes.

## Selección de ruta de enrutamiento utilizando Longest Match:

Los enrutadores siempre seleccionan la ruta que debe tomar un paquete examinando la longitud del prefijo de una entrada de red. La ruta seleccionada para un paquete se elige en función de la longitud del prefijo, donde siempre se prefiere el de mayor longitud. Por ejemplo, /28 se prefiere a /26, y /26 a /24.

Esta lógica puede utilizarse para influir en la selección de rutas en BGP. Supongamos que una organización posee el rango de red 100.64.0.0/16, pero solo necesita anunciar dos subredes (100.64.1.0/24 y 100.64.2.0/24). Podría anunciar ambos prefijos (100.64.1.0/24 y 100.64.2.0/24) desde todos sus enrutadores, pero ¿cómo puede distribuir la carga para cada subred si todo el tráfico entra en un solo enrutador (como R1)? La organización podría modificar varios atributos de ruta BGP (PA) que se anuncian externamente, pero un SP podría tener una política de enrutamiento BGP que ignore dichos atributos de ruta, lo que resultaría en una recepción aleatoria de tráfico de red.

Una forma más elegante de garantizar que las rutas se seleccionen de forma determinista fuera de la organización es anunciar un prefijo de resumen (100.64.0.0/16) a ambos enrutadores. Posteriormente, la organización puede anunciar un Longest Match prefix al router que debería recibir tráfico de red para ese prefijo. La Figura 12-10 muestra el concepto: R1 anuncia el prefijo 100.64.1.0/24, R2 anuncia el prefijo 100.64.2.0/24 y ambos enrutadores anuncian el prefijo de red de sumarizado 100.64.0.0/16.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/81872987257b9e3d928da2d916296f66185926b5/12.%20ADVANCED%20BGP/IMG/ADV%20BGP%20UPS/ADV%20BGP%20UPS%201.png)

Independientemente de la política de enrutamiento de un SP, los prefijos más específicos se anuncian solo en un router. Se proporciona redundancia mediante la publicación de la dirección sumarizada. Si R1 falla, los dispositivos utilizan la publicación de ruta de R2 de 100.64.0.0/16 para acceder a la red 100.64.1.0/24.

> [!NOTE]
> Asegúrese de que rutas de red sumarizadas que su organización publica se encuentren únicamente dentro del alcance de su red. Además, los proveedores de servicios no suelen aceptar rutas IPv4 con una longitud  superior a /24 (por ejemplo, /25 o /26) ni rutas IPv6 con una longitud superior a /48. Las rutas están restringidas para controlar el tamaño de la tabla de enrutamiento de Internet.

En BGP, los anuncios de ruta consisten en paquetes de actualización BGP que contienen `Network Layer Reachability Information (NLRI)`. La NLRI consta de la longitud del prefijo y el prefijo. Una ruta BGP puede contener múltiples rutas a la misma red de destino. Los atributos de cada ruta influyen en su conveniencia cuando un enrutador selecciona la mejor ruta. Por defecto, un enrutador BGP anuncia solo la mejor ruta a los enrutadores vecinos. Dentro de la **Loc-RIB**, se mantienen todas las rutas y sus atributos, y se calcula la mejor ruta. Esta mejor ruta se instala en la RIB del enrutador. Si la mejor ruta ya no está disponible, el enrutador puede usar las rutas existentes para identificar rápidamente una nueva. BGP recalcula la mejor ruta para un prefijo ante cuatro posibles eventos:

- Cambio en la accesibilidad del siguiente salto de BGP
- Fallo de una interfaz conectada a un par eBGP
- Cambio en la redistribución
- Recepción de rutas nuevas o eliminadas para una ruta

BGP instala automáticamente la primera ruta recibida como la mejor. Cuando se reciben rutas adicionales con la misma longitud de prefijo de red, las rutas más recientes se comparan con la mejor ruta actual. Si hay un empate, el procesamiento continúa hasta que se identifica una ruta ganadora.

El algoritmo de mejor ruta de BGP utiliza los siguientes atributos, en el orden mostrado, para la selección de la mejor ruta:

1. Peso
2. Preferencia local
3. Localmente originado (declaración de red, redistribución o agregación)
4. AIGP
5. Ruta AS_Path más corta
6. Tipo de origen
7. MED más baja
8. eBGP sobre iBGP
9. Métrica IGP más baja para el siguiente salto
10. Se prefiere la ruta de la sesión eBGP más antigua
11. Se prefiere la ruta que proviene del punto BGP con el RID más bajo
12. Se prefiere la ruta con la longitud mínima de la lista de clústeres
13. Se prefiere la ruta que proviene de la dirección vecina más baja

La política de enrutamiento BGP puede variar entre organizaciones, según la manipulación de los PA BGP. Dado que algunos PA transmiten datos de un AS a otro, estos cambios también podrían afectar el enrutamiento descendente de otros SP. Otros PA solo influyen en la política de enrutamiento dentro de la organización. Los prefijos de red se ajustan condicionalmente según diversos factores, como la longitud de AS_Path, el ASN específico, las comunidades BGP u otros atributos.

### Weight (peso):

El peso de BGP es un atributo definido por Cisco y el primer paso para seleccionar la mejor ruta de BGP.
El peso es un valor de 16 bits (de 0 a 65 535) asignado localmente en el enrutador; no se anuncia a otros enrutadores. Se prefiere la ruta con el mayor peso. El peso se puede configurar para rutas específicas con un mapa de rutas de entrada o para todas las rutas aprendidas de un vecino específico. El peso solo influye en el tráfico saliente de un enrutador o un sistema autónomo (AS). Dado que es el primer paso en el algoritmo de mejor ruta, debe utilizarse cuando otros atributos no deban influir en la mejor ruta para una red específica.

Al examinar un prefijo de red con el comando `show bgp afi safi network`, se mostrarán todas las rutas y los atributos de ruta asociados, así como la mejor ruta seleccionada. Identificar la mejor ruta puede llevar tiempo para quienes no utilizan BGP, y las versiones más recientes de iOS XE proporcionan comandos para identificar la mejor ruta rápidamente con el comando `show bgp afi safi network bestpath`. El comando `show bgp afi safi network best-path-reason` mostrará todas las rutas y describirá por qué una es preferida o no.

El ejemplo 12-28 muestra la tabla BGP para el prefijo de red 172.16.0.0/24 en R5. En la cuarta línea de la salida, el enrutador indica que existen dos rutas y que la segunda es la mejor. Al examinar la salida de cada ruta, la ruta obtenida a través del AS 200 tiene un peso de 123. La ruta a través del AS 300 no tiene dicho peso, lo que equivale a un valor de 0; por lo tanto, la ruta a través del AS 200 es la mejor. Observe que el segundo conjunto de resultados utiliza la palabra clave opcional `bestpath` y solo muestra la mejor ruta. El tercer conjunto de resultados incluye la palabra clave opcional `best-path-reason`. Este resultado indica que la primera ruta tiene un peso menor que la mejor ruta seleccionada.

Ejemplo 12-28 Un ejemplo de elección de mejor ruta de BGP basada en el peso
````
R5# show bgp ipv4 unicast 172.16.0.0/24
! Output omitted for brevity
BGP routing table entry for 172.16.0.0/24, version 5
Paths: (2 available, best #2, table default) <--------------------(best #2)
  Not advertised to any peer
  300 100
    192.168.6.6 (metric 130816) from 192.168.6.6 (192.168.6.6)
      Origin IGP, metric 0, localpref 100, valid, internal
  200 100
    192.168.4.4 (metric 130816) from 192.168.4.4 (192.168.4.4)
      Origin IGP, metric 0, localpref 100, weight 123, valid, internal, best <--------------------(weight 123,best)
----------------------------------------------------------------------------
R5# show bgp ipv4 unicast 172.16.0.0/24 bestpath
! Output omitted for brevity
BGP routing table entry for 172.16.0.0/24, version 5
Paths: (2 available, best #2, table default)
  200 100
    192.168.4.4 (metric 130816) from 192.168.4.4 (192.168.4.4)
      Origin IGP, metric 0, localpref 100, weight 123, valid, internal, best
----------------------------------------------------------------------------
R5# show bgp ipv4 unicast 172.16.0.0/24 best-path-reason
! Output omitted for brevity
BGP routing table entry for 172.16.0.0/24, version 5
Paths: (2 available, best #2, table default)
  300 100
    192.168.6.6 (metric 130816) from 192.168.6.6 (192.168.6.6)
      Origin IGP, metric 0, localpref 100, valid, internal
      Best Path Evaluation: Lower weight <-------------------- MUY BAJO
  200 100
    192.168.4.4 (metric 130816) from 192.168.4.4 (192.168.4.4)
      Origin IGP, metric 0, localpref 100, weight 123, valid, internal, best
      Best Path Evaluation: Overall best path <-------------------- EXCELENTE MAYOR PESO
````

### Preferencia local (Local preference):

La preferencia local (LOCAL_PREF) es un `well-known path attribute` que se incluye en los anuncios de ruta en un AS. Este atributo es un valor de 32 bits (de 0 a 4 294 967 295) que indica la preferencia para salir del AS hacia la red de destino.
La preferencia local no se anuncia entre pares eBGP y se utiliza normalmente para influir en la dirección del siguiente salto del tráfico saliente (es decir, al salir de un sistema autónomo). La preferencia local se puede configurar para rutas específicas mediante un `route map` o para todas las rutas recibidas de un vecino específico.
**Se prefiere un valor alto sobre uno bajo**. Si un router BGP de borde no define la preferencia local al recibir un prefijo, se utiliza el valor predeterminado de 100 durante el cálculo de la mejor ruta y se incluye en los anuncios a otros pares iBGP. Modificar la preferencia local puede influir en la selección de ruta en otros pares iBGP sin afectar a los pares eBGP, ya que la preferencia local no se publica fuera del sistema autónomo.
El ejemplo 12-29 muestra la tabla BGP para el prefijo de red 172.16.1.0/24 en R4. En la tercera línea de la salida, el enrutador indica que existen dos rutas y que la primera es la mejor. El peso BGP no existe, por lo que se utiliza la preferencia local para seleccionar la mejor ruta.
La ruta obtenida a través del AS 300 es la mejor porque tiene una preferencia local de 333, mientras que la ruta a través del AS 200 tiene una preferencia local de 111.

Ejemplo 12-29 Un ejemplo de elección de mejor ruta de BGP basada en preferencia local
````
R4# show bgp ipv4 unicast 172.16.1.0/24 best-path-reason
! Output omitted for brevity
  BGP routing table entry for 172.16.1.0/24, version 6
Paths: (2 available, best #1, table default) <--------------------(best #1)
  300 100
    192.168.6.6 (metric 131072) from 192.168.6.6 (192.168.6.6)
      Origin IGP, metric 0, localpref 333, valid, internal, best <--------------------(localpref 333)
      Best Path Evaluation: Overall best path <--------------------(best)
  200 100
    100.64.24.2 from 100.64.24.2 (192.168.2.2)
      Origin IGP, localpref 111, valid, external 
      Best Path Evaluation: Lower local preference <--------------------(muybajo)
````

### Locally Originated via Network or Aggregate Advertisement (Originado localmente a través de publicidad en red o agregada): 

El tercer punto de decisión en el algoritmo de mejor ruta es determinar si la ruta se originó localmente. La preferencia se da en el siguiente orden:
- Rutas anunciadas localmente
- Redes agregadas localmente
- Rutas recibidas por puntos BGP.

### Accumulated Interior Gateway Protocol (AIGP):

Es un `nontransitive path attribute` que se incluye en los anuncios de un AS. Los IGP suelen utilizar la métrica de ruta más baja para identificar la ruta más corta a un destino, pero no pueden ofrecer la escalabilidad de BGP. BGP utiliza un AS para identificar un único dominio de control para una política de enrutamiento. BGP no utiliza la métrica de ruta debido a problemas de escalabilidad, combinados con la idea de que cada AS puede utilizar una política de enrutamiento diferente para calcular las métricas.
AIGP permite a BGP mantener y calcular una métrica de ruta conceptual en entornos que utilizan múltiples AS con dominios de enrutamiento IGP únicos en cada AS. La capacidad de BGP para tomar decisiones de enrutamiento basadas en una métrica de ruta es una opción viable porque todos los AS están bajo el control de un único dominio, con políticas de enrutamiento consistentes para BGP e IGP.
En la Figura 12-11, los AS 100, AS 200 y AS 300 están bajo el control del mismo proveedor de servicios. AIGP se ha habilitado en las sesiones BGP entre todos los enrutadores, y los IGP se redistribuyen en BGP. La métrica AIGP se publica entre AS 100, AS 200 y AS 300, lo que permite que BGP la utilice para calcular la mejor ruta entre los sistemas autónomos.

