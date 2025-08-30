# Topicos Presentes
> OSPF y temas avanzados

## Areas: 
Manera que las `areas` generadas en ospf se comunican.

* Link-State Advertisements: LSA `#0969DA` y sus tipos.
  
* Discontiguous Networks: Un mal area 0 distribuido genera problemas a menos que lo realices por GRE TUNNEL

* OSPF Path Selection: OSPF prefiere el intra area (que sea en una misma area) que inter area (viaje por otras areas)

* Summarization of Routes: OSPF se puede sumarizar y publicar

* Route Filtering: Hay manera de filtrar lo que se publica en los LSDB para otras areas

> [!NOTE]
> Soporta hasta 4 ECMP paths default y se puede aumentar dentro del proceso OSPF `maximun paths #s`.

- [ ] Carpeta completa :tada:\

 ![Image Alt](https://github.com/Nigelpa74/CCNP-brief/blob/35479f397f5912056e406a0c9b2250337302fe77/Area%20Ospf.png)

## Tipos de LSAs:

- Type 1, router LSA: Advertises the LSAs that originate within an area
- Type 2, network LSA: Advertises a multi-access network segment attached to a DR
- Type 3, summary LSA: Promociona las otras redes de otras areas.
- Type 4, ASBR summary LSA: Advertises a summary LSA for a specific ASBR
- Type 5, AS external LSA: Advertises LSAs for routes that have been redistributed
- Type 7, NSSA external LSA: Advertises redistributed routes in NSSAs

En verificacion con Wireshark se tiene lo siguiente:



## Sumarizacion de Areas: 



> [!NOTE]
> Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.

> [!IMPORTANT]
> Key information users need to know to achieve their goal.

> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.
