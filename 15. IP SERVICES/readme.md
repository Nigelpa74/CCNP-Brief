Además de enrutar y conmutar paquetes de red, un enrutador puede realizar funciones adicionales para optimizar la red. Este capítulo abarca la sincronización horaria, las tecnologías de virtual gateway y la Network Address Translation (NAT).

# Sincronización horaria:

La hora del sistema de un dispositivo se utiliza para medir los períodos de inactividad o computación. Garantizar la consistencia de la hora en un sistema es importante, ya que las aplicaciones suelen utilizarla para optimizar los procesos internos. Desde la perspectiva de la gestión de una red, es importante que la hora esté sincronizada entre los dispositivos de red por varias razones:

- Administrar contraseñas que cambian a intervalos específicos
- Gestionar el intercambio de claves de cifrado
- Verificar la validez de los certificados según su fecha y hora de caducidad
- Correlacionar eventos de seguridad en múltiples dispositivos (routers, switches, firewalls, sistemas de control de acceso a la red, etc.)
- Solucionar problemas en los dispositivos de red y correlacionar eventos para identificar la causa raíz de un evento

Sin un método de sincronización, la hora puede variar entre dispositivos. Incluso si la hora se configura con precisión en todos los dispositivos, los intervalos de tiempo podrían ser más rápidos en un dispositivo que en otro. Con el tiempo, las horas podrían empezar a diferir. Algunos dispositivos solo utilizan un reloj de software, que se reinicia al reiniciar el equipo. Otros utilizan un reloj de hardware, que mantiene la hora al reiniciar el equipo.

## Network Time Protocol (NTP):

La RFC 958 introdujo el Protocolo de Tiempo de Red (NTP), que se utiliza para sincronizar un conjunto de relojes de red en una arquitectura cliente-servidor distribuida. NTP es un protocolo basado en UDP que se conecta con los servidores en el puerto 123. El puerto de origen del cliente es dinámico.

NTP se basa en un concepto jerárquico de comunicación. En la cima de la jerarquía se encuentran los dispositivos autorizados que funcionan como un **SERVER NTP** con un reloj atómico. El **CLIENTE NTP** consulta la hora del servidor NTP y la actualiza según la respuesta. Dado que NTP se considera una aplicación, la consulta puede realizarse en múltiples saltos, lo que requiere que los clientes NTP identifiquen la precisión horaria basándose en mensajes con otros enrutadores.

El proceso de sincronización NTP no es rápido. En general, un cliente NTP puede sincronizar una gran discrepancia horaria con una precisión de un par de segundos con unos pocos ciclos de sondeo de un servidor NTP. Sin embargo, obtener una precisión de decenas de milisegundos requiere horas o días de comparaciones. En cierto modo, la hora de los clientes NTP se acerca a la hora del servidor NTP.

NTP utiliza el concepto de **STRATUMS** para identificar la precisión de la fuente del reloj. Los servidores NTP conectados directamente a una fuente de tiempo autorizada son servidores de stratum 1. Un cliente NTP que consulta a un servidor de stratum 1 se considera un cliente de stratum 2. Cuanto más alto sea el stratum, mayor será la probabilidad de desviación horaria con respecto a la fuente de tiempo autorizada debido al número de desfases temporales entre los stratums NTP.

La Figura 15-1 muestra el concepto de stratums, con R1 conectado a un reloj atómico y considerado un servidor de stratum 1. R2 está configurado para consultar a R1, por lo que se considera un cliente de stratum 2. R3 está configurado para consultar a R2, por lo que se considera un cliente de stratum 3. Esto podría continuar hasta el stratum 15. Observe que R4 está configurado para consultar a R1 en múltiples saltos y, por lo tanto, se considera un cliente de stratum 2.

![Image Alt](https://github.com/Nigelpa74/CCNP-Brief/blob/bf5cd4014b462048caf360d8f0b92fae382dfa70/15.%20IP%20SERVICES/IMG/IPS%20TIME/IPS%20TIME%201.JPG)

## NTP Configuracion:

La configuración de un cliente NTP es bastante sencilla. La configuración del cliente utiliza el comando de configuración global `ntp server <ip-address> [prefer] [source <interface-id>]`. La `interfaz source`, opcional, se utiliza para especificar la dirección IP de origen para las consultas a ese servidor. Se pueden configurar varios servidores NTP para redundancia, y la adición de la palabra clave opcional `prefer` indica de qué servidor NTP debe provenir la sincronización horaria.

Los dispositivos Cisco pueden actuar como servidores después de haber podido consultar un servidor NTP. Por ejemplo, en la Figura 15-1, una vez que R2 ha sincronizado la hora con R1 (una fuente horaria de stratum 1), R2 puede actuar como servidor para R3. La configuración de relojes externos queda fuera del alcance de este libro.
Sin embargo, debe saber que puede usar el comando `ntp master <stratum-number>` para configurar estáticamente el stratum de un dispositivo cuando actúa como servidor NTP.

El ejemplo 15-1 demuestra la configuración de R1, R2, R3 y R4 de la Figura 15-1

Ejemplo 15-1 Configuración NTP simple de múltiples stratums
````
R1# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
R1(config)# ntp master 1 <--------------------
-----------------------------------------------------------
R2# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
R2(config)# ntp server 192.168.1.1 <--------------------
-----------------------------------------------------------
R3# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
R3(config)# ntp server 192.168.2.2 source loopback 0 <--------------------
-----------------------------------------------------------
R4# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
R4(config)# ntp server 192.168.1.1 <--------------------
````

Para ver el estado del servicio NTP, utilice el comando `show ntp status`, que genera el siguiente resultado:

1. Si el reloj de hardware está sincronizado con el reloj de software (es decir, si el reloj se reinicia al reiniciar), la referencia de stratum del dispositivo local y el identificador del reloj de referencia (local o dirección IP).
2. La frecuencia y precisión del reloj.
3. El tiempo de actividad y la granularidad de NTP.
4. El tiempo de referencia.
5. El desfase y el retraso del reloj entre el cliente y el servidor de stratum de nivel inferior.
6. Dispersión de raíz (es decir, el error calculado del reloj real asociado al reloj atómico) y dispersión de pares (es decir, la dispersión de raíz más el tiempo estimado para llegar al servidor NTP raíz).
7. Filtro de bucle NTP (que queda fuera del alcance de este libro).
8. Intervalo de sondeo y tiempo desde la última actualización.

El ejemplo 15-2 muestra el resultado del estado de NTP de R1, R2 y R3. Observe que el stratum se ha incrementado, junto con el reloj de referencia.

Ejemplo 15-2 Visualización del estado de NTP
````
R1# show ntp status
Clock is synchronized, stratum 1, reference is .LOCL. <--------------------
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**10
ntp uptime is 2893800 (1/100 of seconds), resolution is 4000
reference time is E0E2D211.E353FA40 (07:48:17.888 EST Wed Jul 24 2019)
clock offset is 0.0000 msec, root delay is 0.00 msec
root dispersion is 2.24 msec, peer dispersion is 1.20 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is 0.000000000 s/s
system poll interval is 16, last update was 4 sec ago.
-----------------------------------------------------------------------------
R2# show ntp status
Clock is synchronized, stratum 2, reference is 192.168.1.1 <--------------------
nominal freq is 250.0000 Hz, actual freq is 249.8750 Hz, precision is 2**10
ntp uptime is 2890200 (1/100 of seconds), resolution is 4016
reference time is E0E2CD87.28B45C3E (07:28:55.159 EST Wed Jul 24 2019)
clock offset is 1192351.4980 msec, root delay is 1.00 msec
root dispersion is 1200293.33 msec, peer dispersion is 7938.47 msec
loopfilter state is 'SPIK' (Spike), drift is 0.000499999 s/s
system poll interval is 64, last update was 1 sec ago.
-----------------------------------------------------------------------------
R3# show ntp status
Clock is synchronized, stratum 3, reference is 192.168.2.2 <--------------------
nominal freq is 250.0000 Hz, actual freq is 250.0030 Hz, precision is 2**10
ntp uptime is 28974300 (1/100 of seconds), resolution is 4000
reference time is E0E2CED8.E147B080 (07:34:32.880 EST Wed Jul 24 2019)
clock offset is 0.5000 msec, root delay is 2.90 msec
root dispersion is 4384.26 msec, peer dispersion is 3939.33 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is -0.000012120 s/s
system poll interval is 64, last update was 36 sec ago.
````

El comando `show ntp associations` proporciona una versión simplificada del estado y el retardo del servidor NTP. La dirección 127.127.1.1 se refleja en el dispositivo local cuando se configura con el comando `ntp master <stratum-number>`. El ejemplo 15-3 muestra las asociaciones NTP para R1, R2 y R3.

Ejemplo 15-3: Visualización de las asociaciones NTP
````
R1# show ntp associations

address ref clock st when poll reach delay offset disp
*~127.127.1.1 .LOCL. 0 0 16 377 0.000 0.000 1.204
* sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
----------------------------------------------------------------------------
R2# show ntp associations

address ref clock st when poll reach delay offset disp
*~192.168.1.1 127.127.1.1 1 115 1024 1 1.914 0.613 191.13
* sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
----------------------------------------------------------------------------
R3# show ntp associations

address ref clock st when poll reach delay offset disp
*~192.168.2.2 192.168.1.1 2 24 64 1 1.000 0.500 440.16
* sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
````
