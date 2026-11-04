## Estructura de configuración
### Clásico
````
ip access-list standard R1
 permit 10.1.100.0
ip access-list standard R3
 permit 10.3.100.0
!
ip access-list standard FILTER-R1-10.1.100.X
 deny 10.1.100.0
 permit any
!
ip prefix-list FILTER-R3-10.3.100.X deny 10.3.100.0/24
ip prefix-list FILTER-R3-10.3.100.X permit 0.0.0.0/0 le 32
!
interface GigabitEthernet0/1
 ip address 10.12.1.1 255.255.255.0
 ip hello-interval eigrp 100 3 -------------
 ip hold-time eigrp 100 15
interface gi0/4
 ip summary-address eigrp 100 172.16.0.0/16 -------------
 ip bandwidth-percent eigrp 100 25 -------------
!
router eigrp 100
 offset-list R1 in 200000 GigabitEthernet0/1 -------------
 offset-list R3 in 200000 GigabitEthernet0/3

 timers active-time 2 -------------
 no auto-summary
 eigrp stub

 distribute-list FILTER-R1-10.1.100.X in -------------
 distribute-list prefix FILTER-R3-10.3.100.X out
!
interface tunnel 100
  ip address 10.123.1.1 255.255.255.0
  no ip split-horizon eigrp 100
````
### Nombrado
````
ip access-list standard R1
 permit 10.1.100.0
ip access-list standard R3
 permit 10.3.100.0
!
ip access-list standard FILTER-R1-10.1.100.X
 deny 10.1.100.0
 permit any
!
ip prefix-list FILTER-R3-10.3.100.X deny 10.3.100.0/24
ip prefix-list FILTER-R3-10.3.100.X permit 0.0.0.0/0 le 32
!
router eigrp EIGRP-NAMED
 address-family ipv4 unicast autonomous-system 100
  af-interface tunnel 100
    no split-horizon

  eigrp stub
  !
  af-interface GigabitEthernet0/4
   summary-address 172.16.0.0 255.255.0.0 -------------
  
  af-interface Serial1/0
   stub-site wan-interface -------------EXCLUSIVO
   bandwidth-percent 25-------------  
  af-interface Serial1/1
   stub-site wan-interface

  eigrp stub-site 100:1 ------------- EXCLUSIVO 

  af-interface default
   hello-interval 3 -------------
   hold-time 15 -------------
  exit-af-interface
   !
   topology base
    offset-list R1 in 200000 GigabitEthernet0/1 -------------
    offset-list R3 in 200000 GigabitEthernet0/3
    distribute-list FILTER-R1-10.1.100.X in -------------
    distribute-list prefix FILTER-R3-10.3.100.X out
    timers active-time 2 -------------
    no auto-summary
   exit-af-topology
   
   network 0.0.0.0
 exit-address-family
````
> [!NOTE]
> EIGRP neighbors can still form an adjacency if the timers do not match, hello menor a hold time

## Comando estrella:
### SIA (Stuck in active):
En caso de que un router vecino quede en SIA (Stuck in active) se debe modificar el `active timer` que por default es 3 min (donde la respuesta se espera en 90s). Para ver si un vecino esta SIA es con el comando:
````
show ip eigrp topology active
````
Para ver el timer configurado:
````
show ip protocols
````
### SUMMARY:
EIGRP cuando lo sumarizas se instala por default una ruta NULL para evitar loops.
### STUB:
Los routers configurados como STUB solo anuncian `CONNECTED` y `SUMMARY routes` pero se puedec configurar para que reparta más:
````
eigrp stub {connected | received-only | redistributed | static | summary}
````
Un problema común con STUB es que puede ser mal colocado y evita que se compartan otras routers. 

Ahí es donde entra EIGRP stub site que es una caracteristica que trabaja por identificar las interfaces WAN y configurar un `IDENTIFICADOR DE SITIO EIGRP STUB`. SOLO DISPONIBLE EN NAMED CONFIGURATION MODE.

Ver donde está configuro el stub site es en el vecino con el comando:
```
show ip eigrp topology 10.1.1.0/24
    Extended Community: StubSite:100:1
```
### OFFSET LIST:
Quieres cambiar el camino de EIGRP, acá es donde entra OFFSET-LIST donde cambia los atributos de camino basado en la `dirección de la actualización`, un `prefijo específico` o una combinación de ambos. Su configuración depende de una ACCESS LIST
````
offset-list [acl-number | acl-name] {in | out} offset-value [interface-id]
````