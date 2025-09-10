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

Cuando la red funciona correctamente, el tráfico entre los sitios utiliza la red SP preferida (MPLS SP2) en ambas direcciones. Esto simplifica la resolución de problemas cuando el flujo de tráfico es simétrico (la misma ruta en ambas direcciones), a diferencia del reenvío asimétrico (una ruta diferente para cada dirección), ya que es necesario descubrir la ruta completa en ambas direcciones. **La ruta se considera determinista cuando el flujo entre sitios está predeterminado y es predecible.
**
# Conditional Matching: 



# Route Maps: 



# BGP Route Filtering and Manipulation: 



# BGP Communities: 



# Understanding BGP Path Selection: 

