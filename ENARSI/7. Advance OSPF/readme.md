## Conceptos clave:

### LSA:
OSPF Usa estos 6 tipos de LSAs:
- Type 1, router: LSAs that advertise prefixes within an area (Prefijos y Routers en el dominio OSPF)
- Type 2, network: LSAs that indicate the routers attached to broadcast segment within an area (Información routers vecinos) Si DR no fue elegido en esa área el LSA2 no existe, mayormente en STUBS. Se puede más con el comando `show ip ospf database`
Para ver información detallada del LSA2 se usa el comando `show ip ospf network`. Cuando un nuevo DR se elige cambia el LSA2
- Type 3, summary: LSAs that advertise prefixes that originate from a different area (Redes importadas de otras áreas). El comando para ver mejor el LSA es `show ip ospf database summary`
- Type 4, ASBR summary: LSA used to locate the ASBR from a different area (Ubica al ASBR por el tipo 5 de LSA). Comando para poder verlo es con `show ip ospf database asbr-summary`
- Type 5, AS external: LSA that advertises prefixes that were redistributed in to OSPF. Router que redistribuya se convierte en ASBR. Comando para visualizarlo mejor es con `show ip ospf database external`
- Type 7, NSSA external: LSA for external prefixes that were redistributed in a local NSSA area
Los LSAs del 1 al 3 sirven para creación del INTRA-AREA e INTER-AREA. Los LSAs del 4 al 7 son totalmente relacionados a rutas OSPF EXTERNAS. (Que fueron redistribuidas en el dominio OSPF). Solo ocurren cuando se redistribuyen rutas dentro de un área NSSA. Comando para ver es `show ip ospf database nssa-external`

### Tipos de enlace de red:
- Transit: Un tipo de tránsito de red que indica que se tiene una adyacencia formada donde un DR fue elegido.
- Point-to-point: La adyacencia se formó con un tipo de red que no tiene un DR
- Stub: Indica que en ese link no se estableció adyacencia de vecinos. Tipos de link P2P y TRANSIT que no se vuelven una adyacencia con otro router OSPF también son considerados como red STUB. `Secondary connected Networs` siempre será considerado stub por adyacencias jamas funcionan con ello.

### ÁREAS STUBBY:
Enfoque en sumarizar rutas que dejan una área. OSPF STUBBY areas tienen un método para filtrar rutas externas y la opción de bloquear rutas `inter-area`, son identificados por el AREA FLAG en un paquete hello de OSPF. Cada router en dominio STUBBY tiene que ser configurado como STUB. Las siguientes áreas son:

- Stub Area: Prohiben los LSA tipo 5 y 4 de entrar al área. Y en caso quiera entrar se genera una ruta `default 0.0.0.0` del tipo LSA 3 
````
Configuración de TODOS los routers en esa área
R3(config)# router ospf 1
R3(config-router)# area 34 stub --------
````

- Totally Stub Area: Esta prohibe los LSAs de tipo 3 (Inter-area), tipo 4 LSA y tipo 5 LSA de entrar al área en el ABR. Cuando reciben el LSA 3 o 5 el ABR genera un `default route` para el área. Cuando una interfaz del ABR está en el área 0, automáticamente genera el `default route`.
````
Config SOLO DEL ABR:
R3(config)# router ospf 1
R3(config-router)# area 34 stub no-summary --------

Config de los Miembros
R4(config)# router ospf 1
R4(config-router)# area 34 stub
````

- Not-so-stubby Area: Mientras un área Stub prohibe los LSAs 5 y 4 e incluso que se redistribuyan rutas externas en este tipo de áreas. Un área Not-so-stubby area prohibe la entrada de LSA tipo 5 pero PERMITE LA REDISITRIBUCIÓN de rutas externas. Mientras el ASBR redistribuye la ruta lo hace con el LSA 7 y no con el 5. Pero cuando el LSA ya alcanza el ABR lo cambia por un tipo LSA 5. El ABR no anuncia una `ruta default` por defecto cuando un LSA del tipo 5 o 7 es bloqueado; sin embargo, durante configuración existe la forma de anunciar la ruta default para brindar conectividad.
````
Config SOLO DEL ABR:
R3(config)# router ospf 1
R3(config-router)# area 34 nssa default-information-originate -------- con default OPCIONAL

Config de los Miembros
R4(config)# router ospf 1
R4(config-router)# area 34 nssa
````

- Totally NSSAs: Mientras totally stubby prohibe LSA 3, 4 y 5 de entrar al área por el ABR, además de prohibir rutas de ser redisitribuidas en el área. En tal caso que áreas que necesiten bloquear LSAs 3 y 5 pero aún asi proveer la capacidad de redistribuir rutas externas en OSPF `DEBEN SER OSPF TOTALLY NSSA`. Cuando el ASBR redistribuye la ruta en OSPF, el ASBR lo hace como un tipo de LSA 7 y cuando ese LSA alcance el ABR lo tranforma en LSA 5. Cuando un ABR de totally NSSA recibe un LSA tipo 3 del backbone, se genera una ruta `default route` para el área totally NSSA. Cuando una interfaz del ABR está con el área 0 actúa como el disparador 

> [!NOTE]
>  Totally stubby areas y totally NSSAs no son definidos en el RFC 2328