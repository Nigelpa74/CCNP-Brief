## Razones de falla para adyacencia:
- Interface is down: The interface `MUST` be up/up.
- Interface not running the OSPF process: If the interface is not enabled for OSPF, it does not send hello packets or form adjacencies.
- Mismatched timers: Hello and dead timers `MUST` match between neighbors.
- Mismatched area numbers: The two ends of a link `MUST` be in the same OSPF area.
- Mismatched area type: In addition to a normal OSPF area type, an area type could be either a stub area or a not-so-stubby area (NSSA). The routers `MUST` agree on the type of area they are in.
- Different subnets: Neighbors `MUST` be in the same subnet.
- Passive interface: The passive interface feature suppresses the sending and receiving of hello packets while still allowing the interface’s network to be advertised.
- Mismatched authentication information: If one OSPF interface is configured for authentication, the OSPF interface at the other end of the link `MUST` be configured with matching authentication information.
- ACLs: An ACL may be denying packets to the OSPF multicast address 224.0.0.5.
- MTU mismatch: Neighboring interfaces `MUST` have matching maximum transmission unit (MTU) values.
- Duplicate router IDs: Router IDs `MUST` be unique.
- Mismatched network types: Based on the OSPF network type characteristics and default values, two neighbors configured with a different OSPF network type might not form an adjacency.

## Posibles fijas en examen.
- Error en sentencia Network 
- Error en timers: A diferencia de EIGRP, los timers de HOLA y DEAD tienen que encajar. Se verifica con `show ip ospf interface gigabitEthernet 1/0` o incluso con `debug ip ospf hello`
- Error en número de área: Verificable con `show ip ospf interface *interface_type and number*`.
- Error en tipo de área: Se puede con el comando `show ip ospf`
- Diferente subred: Se tiene que ver la ip `show running-config interface interface_type interface_number`.
- Interfaz pasiva: Se puede ver con `show ip protocols`
- Autenticación errónea: Se puede ver con `show ip ospf interface gigabitEthernet 1/0`, `show ip ospf` y `debug ip ospf adj`
- ACL: Se puede verificar en `show ip protocols`
- Diferente tipo de red: Broadcast y demás. Se puede verificar con `show ip ospf interface *interface_type interface_number*`
- Verificar STUBBY AREAS: Con el comando `show ip ospf`
- Túneles GRE: Se verifica con el comando `show ip ospf interface ser 1/0`
- Errores en sumarización: Se puede verificar con `show ip ospf`
- Discontinuidad: Todas las áreas si o si deben estar conectadas al área 0. En caso de no estarlo se tiene que usar `virtual-link`