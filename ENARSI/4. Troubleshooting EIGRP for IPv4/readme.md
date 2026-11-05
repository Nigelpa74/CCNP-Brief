## Razones de falla para adyacencia:
- Interface is down: The interface must be up/up.
- Mismatched autonomous system numbers: Both routers need to be using the same autonomous system number.
- Incorrect network statement: The network statement must identify the IP address of the interface you want to include in the EIGRP process.
- Mismatched K values: Both routers must be using exactly the same K values.
- Passive interface: The passive interface feature suppresses the sending and receiving of hello packets while still allowing the interface’s network to be advertised.
- Different subnets: The exchange of hello packets must be done on the same subnet; if it isn’t, the hello packets are ignored.
- Authentication: If authentication is being used, the key ID and key string must match, and the key must be valid (if valid times have been configured).
- ACLs: An access control list (ACL) may be denying packets to the EIGRP multicast address 224.0.0.10.
- Timers: Timers do not have to match; however, if they are not configured correctly, neighbor adjacencies could flap.

## Posibles fijas en examen.
### Error en sentencia Network: 
Esto se puede verificar con que interfaces participan del proceso `show ip eigrp interfaces` o `show ip protocols`
### Mismatched K Values:
Se puede verificar los valores con `show ip protocols`
### Interfaces Pasivas:
Se puede verificar los valores con `show ip protocols`
### Diferentes Sub-redes:
Verificar las ips de las interfaces
### Autenticación:
Para verificar en todos los casos es primero la interfaz con el `show running interface` y para ver las llaves `show key chain`. En caso de querer examinar con el debug se tiene a los comandos `debug eigrp packets`
### Access control lists:
Tener con los ACLs aplicadas hacia ADENTRO, los de OUT no tiene muchas repercusiones en los de EIGRP, se puede ver `show ip protocols`.

La comunicación entre vecinos es la siguiente en IPV6. Desde el link local FE80::2 hacia el MULTICAST DE FF02::A
### TIMERS: 
Aunque no tienen que ser iguales, tienen que almenos estar en rangos de tiempo.
### STUB:
Verificar si el STUB está mal ubicado, se puede configurar el stub para compartir mas rutas de lo pensado.
### SPLIT HORIZON:
En DMVPN verificar que no este habilitado el SPLIT HORIZON
