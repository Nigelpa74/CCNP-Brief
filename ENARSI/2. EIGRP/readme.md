## Terminilogía Clave:
### Feasible distance (FD): 
The metric value for the lowest path metric to reach a destination. 

The feasible distance is calculated locally using the formula shown in the “Path Metric Calculation” section, later in this chapter.
The FD calculated by R1 for the 10.4.4.0/24 destination network is 3328 (that is, 256 + 256 + 2816).

### Reported distance (RD):
Distance reported by a router to reach a destination.

The reported distance value is the feasible distance for the advertising router.
R3 advertises the 10.4.4.0/24 destination network to R1 and R2 with an RD of 3072. R4 advertises the 10.4.4.0/24 destination network to R1, R2, and R3 with an RD of 2816.

### Feasibility condition:
For a route to be considered a backup route, the RD received for that route must be less than the FD calculated locally. This logic guarantees a loop-free path.
````
Reported distance (RD) < Feasible distance (FD)
````
## Estructura de configuración
### Clásico
````
R1 (Classic Configuration)
interface Loopback0
 ip address 192.168.1.1 255.255.255.255
!
interface GigabitEthernet0/1
    ip address 10.12.1.1 255.255.255.0
    ip authentication mode eigrp 100 md5
    ip authentication key-chain eigrp 100 EIGRPKEY
    delay 100
!
interface GigabitEthernet0/2
    ip address 10.11.11.1 255.255.255.0
!
key chain EIGRPKEY
 key 2
  key-string CISCO
!
router eigrp 100
 maximum-paths 6
 variance 2
 eigrp router-id 192.168.1.1
 passive-interface default
 passive-interface gi0/2
 network 10.11.11.1 0.0.0.0
 network 10.12.1.1 0.0.0.0
 network 192.168.1.1 0.0.0.0
````
### Nombrado
````
R2 (Named Mode Configuration)
interface Loopback0
 ip address 192.168.2.2 255.255.255.255
!
interface GigabitEthernet0/1
    ip address 10.12.1.2 255.255.255.0
    delay 100
!
interface GigabitEthernet0/2
    ip address 10.22.22.2 255.255.255.0
!
key chain EIGRPKEY
 key 2
  key-string CISCO
!
router eigrp EIGRP-NAMED
 address-family ipv4 unicast autonomous-system 100
  eigrp router-id 192.168.2.2
  !
  af-interface default
   passive-interface
   authentication mode md5
   authentication key-chain EIGRPKEY
  exit-af-interface
  !
  af-interface GigabitEthernet0/1
   no passive-interface
  exit-af-interface
  !
  topology base
   variance 2
   maximum-paths 6 (default es max 4)
  exit-af-topology
  network 0.0.0.0
 exit-address-family
````
Comando estrella:
 ````
 variance
 ````
 The FD for the successor route is 3328, and the FD for the feasible successor is 5376. The formula provides a value of about 1.6 and is always rounded up to the nearest whole number to provide an EIGRP variance multiplier of 2.
 5376 / 3328 =< 1.6

 La varianza es 2

 ````
 show ip protocols
 ````
 To accelerate troubleshooting of passive interfaces, as well as other settings, use the command show ip protocols, which provides a lot of valuable information about all the routing protocols. With EIGRP, it displays the EIGRP process identifier, ASN, K values used for path calculation, RID, neighbors, AD settings, passives interfaces