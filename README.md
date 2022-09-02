# Load-Balancing-with-Failover-RouterOS-7
Failover (WAN Backup)


- Crear en Routing->table
```

/routing/table
add fib name=to_ISP1
add fib name=to_ISP2
```

- Crear las reglas mangle

```
/ip firewall mangle
add action=mark-connection chain=prerouting comment="Marcar conexiones Entrantes de ISP" connection-mark=no-mark in-interface=ether1-wan new-connection-mark=ISP1_conn passthrough=yes
add action=mark-connection chain=prerouting connection-mark=no-mark in-interface=ether2-wan new-connection-mark=ISP2_conn passthrough=yes
```

- Balanceo de los Servicios
```
/ip firewall mangle
add action=mark-connection chain=prerouting comment="Balanceo de los ISP" connection-mark=no-mark dst-address-type=!local in-interface=bridge new-connection-mark=ISP1_conn passthrough=yes per-connection-classifier=both-addresses:2/0
add action=mark-connection chain=prerouting connection-mark=no-mark dst-address-type=!local in-interface=bridge new-connection-mark=ISP2_conn passthrough=yes per-connection-classifier=both-addresses:2/1
```
- Definir Rutas
```
/ip firewall mangle
 add action=mark-routing chain=prerouting comment="Definir Rutas" connection-mark=ISP1_conn in-interface=bridge passthrough=no
 add action=mark-routing chain=prerouting connection-mark=ISP2_conn in-interface=bridge passthrough=no
 ```
 Luego crearlas debes volver abrir configurar manualmente el campo New Routing Mark para cada una ejemplo New Routing Mark: to_ISP1 o to_ISP2 dato proviene de /routing/table 
- Salidas
```
/ip firewall mangle
 add action=mark-routing chain=output comment="Marcar las salida de las conexiones" connection-mark=ISP1_conn  passthrough=no
 add action=mark-routing chain=output connection-mark=ISP2_conn  passthrough=no
```
Luego crearlas debes volver abrir configurar manualmente el campo New Routing Mark para cada una ejemplo New Routing Mark: to_ISP1 o to_ISP2 dato proviene de /routing/table 

- Definimos las rutas Ip->Route

```
/ip/route/
add dst-address=8.8.8.8 scope=10 gateway=%ether1-wan
add dst-address=8.8.4.4 scope=10 gateway=%ether2-wan

/ip/route/


add distance=1 gateway=8.8.8.8 routing-table=to_ISP1 check-gateway=ping
add distance=2 gateway=8.8.4.4 routing-table=to_ISP2 check-gateway=ping


```



