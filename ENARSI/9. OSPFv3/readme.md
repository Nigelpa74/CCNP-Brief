- Soporte para address-f ipv6
- Nuevos LSAs
- Corre directamente sobre IPV6
- Authentication eliminada, ahora se lleva por extensiones de ip-sec en headers de paquete IPV6.
- Id protocolo 89 y comunicación de SFProuters (FF02::05) y hacia DRrouter (FF02::06).

## Conceptos Esenciales:

### LSA anuncios:
Empezando, modifica la estructura del LSA router 1, ahora usa el término Prefijo inter-area LSA, además que los routers ASBR ahora se les llama Router LSA inter-area. Diferencia principal es que el router LSA es responsable de anunciar los parámetros de las interfaces de tipos como (p2p,broadcast,no-broadcast,p2multi-point y links virtuales).

La información de dirección ip anunciada es independiente por 2 tipos de LSA:
- Intra-area prefix LSA
- Link-local LSA

### LSA OSPFv3 tipos:

- 0x2001 (Device):
Every router generates router LSAs that describe the state and cost of the router’s interfaces to the area.

- 0x2002 (Network):
A designated router generates network LSAs to announce all the routers attached to the link, including itself.

- 0x2003 (Inter-area prefix):
Area border routers generate inter-area prefix LSAs to describe routes to IPv6 address prefixes that belong to other areas.

- 0x2004 (Inter-area router):
Area border routers generate inter-area router LSAs to announce the addresses of autonomous system boundary routers in other areas.

- 0x2005 (AS-external):
Autonomous system boundary routers advertise AS-external LSAs to announce default routes or routes learned through redistribution from other protocols.

- 0x2007 (NSSA):
Autonomous system boundary routers that are located in a not-so-stubby area (NSSA) advertise NSSA LSAs for routes redistributed into the area.

- 0x0008 (Link):
The link LSA maps all the global unicast address prefixes associated with an interface to the link-local interface IP address of the router. The link LSA is shared only between neighbors on the same link.

- 0x2009 (Intra-area prefix):
The intra-area-prefix LSA is used to advertise one or more IPv6 prefixes that are associated with a router, stub, or transit network segment.

### Bits que describen capacidades des router:

## Estructura de configuración:
````
ipv6 unicast routing ----------
!
interface Loopback0
 ipv6 address 2001:DB8::1/128
 ospfv3 1 ipv6 area 0
!
interface GigabitEthernet0/1
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:0:1::1/64
 ospfv3 1 ipv6 area 0 ----------
 ospfv3 network point-to-point
!
router ospfv3 1 ----------
 router-id 192.168.1.1
 passive-interface default ----------
 passive-interface GigabitEthernet0/1 --------
 address-family ipv6 unicast
  area 0 range 2001:db8:0:0::/65
````

## Configuración de seguridad:
OSPFv3 autenticación es por medio de IPsec, en cambio de IPSEC-VPN tunnel los vecinos de OSPFv3 no se intercanbian IKE para negociar la seguridad. En cambio se usa en OSPFv3 IPsec Security Parameter Index (SPI).

Con el comando `show crypto ipsec sa | include spi` determinará las sesiones ipsec activas y los valores SPI actualmente usados.

### Ejemplo:
OSPFv3 paquetes usando ESP. Sesión IP-SEC creado con los siguiente valores. Tanto en ÁREA e INTERFAZ:
- Security policy index: = 500
- Encryption algorithm: = 3des
- Encryption key: = 012345678901234567890123456789012345678901234567
- Authentication algorithm: = sha1
- Authentication key: = 0123456789012345678901234567890123456789

Interfaz:
````
interface GigabitEthernet0/1
 ospfv3 encryption ipsec spi 500 esp 3des 012345678901234567890123456789012345678901234567 sha1 0123456789012345678901234567890123456789
! one single CLI command.
````
Área:
````
router ospfv3 100
 area 0 encryption ipsec spi 500 esp 3des 012345678901234567890123456789012345678901234567 sha1 0123456789012345678901234567890123456789
````

## Diagnóstico:

### Reemplazo:
El comando de show en ospfv3 se reemplaza el `ip ospf` con `ospfv3`
- `show ospfv3 ipv6 neighbor`
- `show ospfv3 interface G0/0`
- Ipssec `show ospf3 interface`