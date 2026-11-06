## Estructura de configuración
### Clásico
````
ipv6 prefix-list BLOCK-DEFAULT seq 5 deny ::/0
ipv6 prefix-list BLOCK-DEFAULT seq 10 permit ::/0 le 128
!
interface GigabitEthernet0/1
 ipv6 address 2001:DB8:0:12::1/64
 ipv6 address fe80::1 link-local
 ipv6 eigrp 100 ----------
 ipv6 summary-address eigrp 100 2001:DB8:1::/48 ----------
!
interface GigabitEthernet0/2
 ipv6 address 2001:DB8:1:1::1/64
 ipv6 address fe80::1 link-local
 ipv6 eigrp 100
 ipv6 summary-address eigrp 100 ::/0 ----------
!
interface Loopback0
 ipv6 address 2001:DB8:1::1/128
 ipv6 eigrp 100
!
ipv6 unicast-routing
!
ipv6 router eigrp 100
 distribute-list prefix-list BLOCK-DEFAULT in GigabitEthernet0/1

 passive-interface Loopback0 ----------
 eigrp router-id 192.168.1.1 ----------
````
### Nombrado
- Soporte para VRF
- When the IPv6 address family is configured for the EIGRP named process, ALL the IPv6-enabled interfaces immediately start participating in routing
- If an IPv4 address is not defined or if the router ID is not manually configured, the routing process does NOT initiate
````
ipv6 prefix-list BLOCK-DEFAULT seq 5 deny ::/0
ipv6 prefix-list BLOCK-DEFAULT seq 10 permit ::/0 le 128
!
interface GigabitEthernet0/1
 ipv6 address 2001:DB8:0:12::2/64
 ipv6 address fe80::2 link-local
 summary-address 2001:DB8:2::/48 ----------
!
interface GigabitEthernet0/2
 ipv6 address 2001:DB8:2:2::2/64
 ipv6 address fe80::2 link-local ----------
 summary-address ::/0 ----------
!
interface Loopback0
 ipv6 address 2001:DB8:2::2/128
!
ipv6 unicast-routing
!
router eigrp NAMED-MODE
 address-family ipv6 unicast autonomous-system 100
  topology base
   distribute-list prefix-list BLOCK-DEFAULT in GigabitEthernet0/1 ----------
  exit-af-topology
  eigrp router-id 192.168.2.2
 exit-address-family
````
## Solución de problemas
### Interfaz caída:
Se puede ver interfaces en UP y DOWN con `show ipv6 interface brief`
### Diferente ASn:
Se verifica con `show ipv6 protocols`
### Números K diferentes:
Se verifica con `show ipv6 protocols`
### Interfaces pasivas:
Se verifica con `show ipv6 protocols`
### Falla en la autenticación:
Se verifica con `show ipv6 eigrp interfaces detail` y `show key chain`
### Timers o tiempos:
Se verifica con `show ipv6 eigrp interfaces detail`
### ACL:
Si hay una access list que niega el tráfico destinado a la dirección multicas FF02::A
### STUB: 
Se verifica con `show ipv6 protocols`
## Solución de problemas para el EIGRP Nombrado:
Cuando crees un eigrp nombredo y tengas 2 address family estos son totalmente independientes 