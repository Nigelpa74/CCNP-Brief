- Trajo consigo la mejora del VLSM.
- Trabaja sobre el protocolo 89
- Comunicación con los router que no son DR: IPv4 address 224.0.0.5 or MAC address 01:00:5E:00:00:05
- Comunicación con los DRs: IPv4 address 224.0.0.6 or MAC address 01:00:5E:00:00:06
## Conceptos:
### Router ID:
The OSPF router ID (RID) is a 32-bit number.
### BITS:
V6-Bit: 
E-Bit: Capaz de procesar `rutas Externas`
R-Bit:
DC-Bit: Descarte y suprimir `Hellos futuros` de ser enviados sobre la interface
N-Bit: Soporta Tipo 7 LSAs
MC-Bit: Soporta Multicast
### External OSPF Rutas:
Los routers que importen ya se convierten en ASBR. Existen 2 tipos de rutas externas, donde el tipo 1 es preferido sobre el tipo 2:
- Tipo 1: Métrica es igual la métrica de redistribución MÁS la métrica total de ruta hacia el ASBR. Si el LSA se propaga más lejos, la métrica incrementa.
- Tipo 2: Métrica es iguala solo la métrica de redistribución. La métrica es la misma por mas que el siguiente router este a más de 30 saltos
### Tipos de red OSPF:
- Broadcast: Es el modo default /
Tiempos: Hola 10, Wait 40, Dead 40
- Non broadcast: Default cuando se habilita OSPF Frame Relay / Tiempos: Hola 30, Wait 120, Dead 120
- P2P: Default cuando se habilita OSPF Frame Relay en sub interfaz p2p / Tiempos: Hola 10, Wait 40, Dead 40
- P2Multi-Point: No default en ningún tipo de interfaz. Interfaz anunciada en una ruta de host (/32), y pone el next-hop address en la interfaz de salida. Usado principalmente para topologías `HUN AND SPOKE` / Tiempos: Hola 30, Wait 120, Dead 120
## Estructura de configuración:
### Con sentencias Network:
````
ip route 0.0.0.0 0.0.0.0 100.64.1.2 ------------
!
router ospf 1
 router-id 192.168.3.3
 default-information originate
 network 0.0.0.0 255.255.255.255 area 1234
 passive-interface GigabitEthernet0/1 -----------
-------------------------------------------
interface GigabitEthernet0/0
 ip address 10.12.1.1 255.255.255.0
 ip ospf authentication --------
 ip ospf authentication-key CISCO --------
!
interface GigabitEthernet0/1
 ip address 10.23.1.2 255.255.255.0
 ip ospf message-digest-key 1 md5 CISCO ----------
!
router ospf 1
 router-id 192.168.5.5
 area 0 authentication message-digest --------
 area 12 authentication --------
 network 10.12.1.0 0.0.0.255 area 12
 network 10.45.1.0 0.0.0.255 area 0
 network 0.0.0.0 255.255.255.255 area 56 ----------
--------------------------------------------
interface GigabitEthernet0/1
 ip address 10.23.1.3 255.255.255.0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 CISCO
!
router ospf 1
 network 10.23.1.0 0.0.0.255 area 0
````
### Con interfaces:
````
router ospf 1
 router-id 192.168.4.4
 default-information originate always ----------
!

interface GigabitEthernet0/0
 ip ospf 1 area 0
 ip ospf network broadcast ---------- default
interface Serial1/0
 ip ospf 1 area 1234
````
### No broadcast:
````
interface Serial 0/0
 ip address 10.12.1.1 255.255.255.252
 encapsulation frame-relay ------
 no frame-relay inverse-arp ------
 frame-relay map ip address 10.12.1.2 102 ------
!
router ospf 1
 router-id 192.168.1.1
 neighbor 10.12.1.2 ------
 network 0.0.0.0 255.255.255.255 area 0
````
### P2P:
````
router ospf 1
 router-id 192.168.4.4
 default-information originate always ----------
!
interface GigabitEthernet0/0
 ip ospf 1 area 0
 ip ospf network point-to-point ----------
interface Serial1/0
 ip ospf 1 area 1234
 ip ospf network point-to-point ---- Default en SERIALS
````
### P2Multi-Point:
Comunmente visto es HUB AND SPOKE mientras se usa la misma subred encontrado en Frame Relay y tecnología L2VPN - MPLS.
````
interface Serial 0/0
  encapsulation frame-relay
  no frame-relay inverse-arp
!
interface Serial 0/0.123 multipoint
  ip address 10.123.1.1 255.255.255.248
  frame-relay map ip 10.123.1.2 102 broadcast
  frame-relay map ip 10.123.1.3 103 broadcast
  ip ospf network point-to-multipoint
!
router ospf 1
 router-id 192.168.1.1
 network 0.0.0.0 255.255.255.255 area 0
````
## Diagnóstico:
### Interfaces:
Con el comando `show ip ospf interface`
### Vecinos:
Con el comando `show ip ospf neighbor`
### Timers:
Con el comando `show ip ospf interface`