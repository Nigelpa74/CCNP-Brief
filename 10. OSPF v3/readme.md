# OSPFv3:

## OSPFv3 Fundamentals: El routing y sus protocolos

- Soporte para familias de IPV4 Y 6
- Nuevos LSAs para IPV6
- OSPFv3 usa el termino *link* en cambio de *network* porque el SPF calculo on po links en cambio de subred.
- Nuevo Link-state filed para determinar el flooding scope del SLA.
- OSPFv3 corre directamente sobre IPV6
- Router ID de preferencia agregado manualmente.
- Autenticacion de vecinos eliminada, ahora lo hace por IPsec cabezal de extension en el paquete IPV6
- OSPFv3 inter router comunicaciones is mantenido por IPV6 *link lical address*. Vecinos detectados por `Non-broadcast multiple access (NBMA)`. Un vecino debe ser manualmente especificado usando el `link locak address`.
- Instancias multiples con el paquete OSPFv3 incluye un ID de instancia, siendo usado para manipular que routers en un segmento de red son permitidos para las adyacencias.

> [!NOTE]
> RFC 5340 se encarga de explicar diferencias entre OSPFv2 y v3

Direccion ip de informacion es publicitado independientemente por los tipos de LSAs:

- Intra-area prefix LSA
- Link LSA

> [!IMPORTANT]
>La comunicacion de OSPFv3 es por numero de protocolo 89 en el campo IPV6 y tambien usa os 5 paquetes y logica como OSPFv2 dependiendo del destino o si es un UNICAST LINK-LOCAL o un MULTICAST.

- `FF02::05:` OSPFv3 AllSPFRouters
- `FF02::06:` OSPFv3 AllDRouters

## OSPFv3 Configuration: Ambiente necesario para que OSPF v3 funcione

- PASO 1: Necesario el `ipv6 unicast-routing` y luego `router ospfv3 ID`.
- PASO 2: Definir el router ID `router-id ID` que puede ser 0.1.2.3.
- PASO 3: Opcional de definir el address family `address-family (ipv6/ipv4) unicast`
- PASO 4: Habilitarlo en una interface `ospfv3 PROCCESS-ID ipv6 area AREA-ID`

> [!NOTE]
> OSPFv3 no usa el estado de network para inicializar interfaces.

## IPv4 Support in OSPFv3: Intercambio de rutas IPV4


