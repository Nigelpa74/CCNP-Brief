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
