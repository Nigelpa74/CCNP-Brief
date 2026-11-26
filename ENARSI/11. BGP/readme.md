## Conceptos generales:
TCP 179

ASNs privados 64,512 - 65,634 y 4,200,000,000 - 4,294,967,294 

BGP PAs (path atributes):
- Well Known: Reconocidos por todas las implementaciones BGP; se tiene Mandatory (Si o si reconocidos y pasados entre ASs) y Discretionary (Si o si reconocidos pero Pueden o No estar incluidos con cada anuncio de prefijo)

- Optional: No tiene que ser reconocido por todas las implementaciones BGP; se tiene Transitive (Pudiendo ser configurado y estar con cada anuncio de rutas de AS a AS) y No-Transitive (No puede ser compartido de AS a AS)

- Primero MP-BGP que tiene a AFI (ipv4-6) y SAFI (unicast - multicast)

Tipo de mensaje BGP:
- Open: Inicio de la asociación BGP.
- Update: Anuncia, actualiza rutas.
- Notification: Informa de errores.
- Keepalive: Asegurar que bgp vecinos están activos.

Estados de sesión BGPO:
- Idle: Iniciando la sesión TCP
- Connect: Sesión TCP completada y envia OPEN. Si falla a IDLE
- Active: BGP inicia nuevo TCP handshake
- OpenSent: Se envía (Versión BGP, source-ip, RID, ttl, Security). Si falla a IDLE
- OpenConfirm: Se espera un mensaje ya sea Keepalive o Notification. Si recibe KEEP pasa a ESTABLISHED. Si falla a IDLE
- Established: Todo bien. Si falla a IDLE 

## Estructura de comandos:
IPV4 address family Habilitado default
````
router bgp 65100
 neighbor 10.12.1.2 remote-as 65200
 neighbor 10.12.1.2 password CISCOBGP
 neighbor 10.12.1.2 timers 10 40
 neighbor 10.23.1.3 next-hop-self
 neighbor 10.34.1.4 route-reflector-client
````
IPV4 address family deshabilitado
````
router bgp 65200
 no bgp default ipv4-unicast
 neighbor 10.12.1.1 remote-as 65100
 neighbor 10.12.1.1 password CISCOBGP
 neighbor 10.12.1.1 timers 15 50
 !
 address-family ipv4
  neighbor 10.12.1.1 activate
  neighbor 10.23.1.3 next-hop-self
  neighbor 10.34.1.4 route-reflector-client
 address-family ipv6
  neighbor 2001:DB8:0:12::2 activate
  redistribute connected
````
IPV6 sobre IPV4
````
R1
router bgp 65100
 bgp router-id 192.168.1.1
 no bgp default ipv4-unicast
 neighbor 10.12.1.2 remote-as 65200
 !
address-family ipv6 unicast
  redistribute connected
  neighbor 10.12.1.2 activate

R2
router bgp 65200
 bgp router-id 192.168.2.2
 no bgp default ipv4-unicast
 neighbor 10.12.1.1 remote-as 65100
 neighbor 10.23.1.3 remote-as 65300
 !
address-family ipv6 unicast
 network 2001:db8::2/128
 network 2001:db8:0:12::/64
 aggregate-address 2001:db8::/62 summary-only
 neighbor 10.12.1.1 activate
 neighbor 10.23.1.3 activate

R3
router bgp 65300
 bgp router-id 192.168.3.3
 no bgp default ipv4-unicast
 neighbor 10.23.1.2 remote-as 65200
 !
 address-family ipv6 unicast
  network 2001:DB8::3/128
  network 2001:DB8:0:3::/64
  network 2001:DB8:0:23::/64
  neighbor 10.23.1.2 activate
````

### Tablas BGP:

- Adj-RIB-in: Contiene rutas en su forma original (Osea antes de ser aplicados filtros). Esa tabla se purga después que todas las políticas de ruta son procesados para guardar memoria.
- Loc-RIB: Contiene todas las rutas originadas localmente o que son recibidos. Después de que las rutas pasen por validación de Next-hop y alcanzabilidad son elegidas por el algoritmo de Mejor Ruta. Esta table es usado para presentar rutas hacia la tabla de enrutamiento.
- Adj-RIB-out: Contiene rutas procesadas que pasaron las políticas Out.

### RR reglas:

- Si un RR recibe un NLRI de un no-RR, el RR lo anuncia hacia un RR-cliente. No anuncia el NLRI hacia un no-RR-cliente
- Si un RR recibe un NLRI de un RR-cliente, este lo anuncia el NLRI hacia RR-cliente y no-RR-cliente
- Si un RR recibe una ruta de un peer eBGP, este lo anuncia la ruta a RR-clientes y a no-RR-cliente

### Confederation