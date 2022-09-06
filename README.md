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
add action=mark-connection chain=prerouting comment="Marcar conexiones Entrantes de ISP" connection-mark=no-mark   in-interface=ether1-wan new-connection-mark=ISP1_conn passthrough=yes
add action=mark-connection chain=prerouting connection-mark=no-mark   in-interface=ether2-wan new-connection-mark=ISP2_conn passthrough=yes
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
 add action=mark-routing chain=prerouting comment="Definir Rutas" connection-mark=ISP1_conn new-routing-mark=to_ISP1  in-interface=bridge passthrough=no
 add action=mark-routing chain=prerouting connection-mark=ISP2_conn new-routing-mark=to_ISP2 in-interface=bridge passthrough=no
 ```
 Luego crearlas debes volver abrir configurar manualmente el campo New Routing Mark para cada una ejemplo New Routing Mark: to_ISP1 o to_ISP2 dato proviene de /routing/table 

- Salidas
```
/ip firewall mangle
 add chain=output connection-state=new connection-mark=no-mark action=mark-connection new-connection-mark=ISP1_conn out-interface=ether1-wan comment="Marcar las salida de las conexiones"
 add action=mark-routing chain=output  connection-mark=ISP1_conn new-routing-mark=to_ISP1  passthrough=no
 add chain=output connection-state=new connection-mark=no-mark action=mark-connection new-connection-mark=ISP1_conn out-interface=ether2-wan
 add action=mark-routing chain=output connection-mark=ISP2_conn new-routing-mark=to_ISP2  passthrough=no
```
Luego crearlas debes volver abrir configurar manualmente el campo New Routing Mark para cada una ejemplo New Routing Mark: to_ISP1 o to_ISP2 dato proviene de /routing/table 

- Definimos las rutas Ip->Route

1.Monitor dst-address=(Dns operador claro:190.157.8.33, movistar:200.21.200.10, etb:200.75.51.132) 
```
/ip/route/
add distance=1 gateway=192.168.137.1 comment="Default IPS1 out"
add distance=2 gateway=192.168.1.1   comment="Default IPS2 out"

/ip/route/
add distance=1 gateway=192.168.137.1 routing-table=to_ISP1  comment="Routing IPS1"
add distance=1 gateway=192.168.1.1 routing-table=to_ISP2 comment="Routing IPS2"

# load-balanced w/ auto failover default gateways
/ip/route/
add dst-address=8.8.8.8/32  gateway=192.168.137.1 scope=11 comment="Balance IPS1"
add dst-address=200.21.200.10/32  gateway=192.168.1.1 scope=11 comment="Balance IPS2"


/ip/route/
add check-gateway=ping distance=10 gateway=8.8.8.8 target-scope=11 routing-table=to_ISP1  comment="Failover IPS1"
add check-gateway=ping distance=20 gateway=200.21.200.10 target-scope=11 routing-table=to_ISP2 comment="Failover IPS2"





```



