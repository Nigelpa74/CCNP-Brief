## Estructura de configuración:
````
R2# show run | section router ospfv3
router ospfv3 10
 area 23 stub ------
 !
 address-family ipv4 unicast
 passive-interface default ------
 no passive-interface GigabitEthernet0/0
 no passive-interface GigabitEthernet1/0 ------
 default-information originate
 router-id 2.2.2.2
 exit-address-family
 !
 address-family ipv6 unicast
 passive-interface default
 no passive-interface GigabitEthernet0/0
 no passive-interface GigabitEthernet1/0
 default-information originate
 router-id 22.22.22.22
 exit-address-family

R2# show run int gig 1/0
interface GigabitEthernet1/0
 ip address 10.1.23.2 255.255.255.0
 ipv6 address 2001:DB8:0:23::2/64
 ospfv3 10 ipv6 area 23 ------
 ospfv3 10 ipv4 area 23
end

R2# show run int gig 0/0
interface GigabitEthernet0/0
 ip address 10.1.12.2 255.255.255.0
 ipv6 address 2001:DB8:0:12::2/64
 ospfv3 10 hello-interval 11
 ospfv3 10 ipv6 area 0
````

## Comandos de diagnóstico:
`show ipv6 protocols` se puede ver:
- process ID
- router ID
- tipo de router (ABR o ASBR)
- Áreas al cual el router pertenece
- Si puerto está en dominio STUB, NSSA, etc

`show ipv6 ospf`
- PID
- RID
- ABR o ASBR
- Timers
- Áreas al cual el router pertenece
- Si puerto está en dominio STUB, NSSA, etc
- Reference Bandwitdh.
- Authentication

`SHOW IPV6 OSPF INTERFACE BRIEF`
- Vez que interfaces participan del proceso
- Costo
- Status DR/BDR
- Área en el que participa

`SHOW IPV6 OSPF INTERFACE`
- Network type
- Coste
- Autenticación
- DR/BDR
- Priority
- Timers

`SHOW IPV6 OSPF NEIGHBOR`
- Neighbor RID
- Priority
- Dead timer
- Interfaz a la que te conectas

`SHOW IPV6 OSPF DATABASE`

`SHOW IPV6 INTERFACE G1/0`
- Grupo Multicast
- MTU
- ACL
- TIMERS

## Fijas examen:

- Verificar el tipo de STUB en los router.
- Timers diferentes `debuf ipv6 osfpf hello`
- Falta de redistribución de default.