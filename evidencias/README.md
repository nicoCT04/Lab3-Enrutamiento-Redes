# Evidencias para el PDF2

Tomarlas después de aplicar todo `configs/` y esperar ~1 min a que converjan OSPF y EIGRP.
En Packet Tracer no existe `terminal length 0`: cuando salga `--More--` presionar espacio y borrar esas líneas al pegar.
Copiar la salida de la CLI como texto (seleccionar → Ctrl+C) y guardarla con el nombre indicado.

| Archivo | Router | Comando | Qué debe verse |
|---|---|---|---|
| `route-R-OSPF-1.txt` | R-OSPF-1 | `show ip route` | `192.168.0.0/24` con **dos** next-hops (10.0.0.2 y 10.0.0.6), `O E1 172.16.0.0/24` |
| `route-R-OSPF-2.txt` | R-OSPF-2 | `show ip route` | `192.168.1.0/26` y `.64/28` con dos next-hops, `O E1 172.16.0.0/24` |
| `route-R-CENTRAL.txt` | R-CENTRAL | `show ip route` | Rutas `O` y `D`, `D 192.168.0.0/23 ... Null0` y `S 172.16.0.0/24 ... Null0` |
| `route-R-EIGRP-1.txt` | R-EIGRP-1 | `show ip route` | `D EX 192.168.0.0/23`, y `172.16.0.0/25` con dos rutas |
| `route-R-EIGRP-2.txt` | R-EIGRP-2 | `show ip route` | `D EX 192.168.0.0/23` |
| `ospf-interface-R-OSPF-1.txt` | R-OSPF-1 | `show ip ospf interface` | Se0/0/0 con **Cost: 128**, Se0/0/1 con Cost: 64 |
| `ospf-ecmp-R-OSPF-1.txt` | R-OSPF-1 | `show ip route 192.168.0.0` | Dos entradas con la misma métrica (129) |
| `eigrp-topology-R-EIGRP-1.txt` | R-EIGRP-1 | `show ip eigrp topology` | `172.16.0.0/25` con 2 successors: FD 2195712 y 6586880 (RD 2170112) |
| `eigrp-share-R-EIGRP-1.txt` | R-EIGRP-1 | `show ip route 172.16.0.10` | Las dos rutas con métricas 2195712 y 6586880 (PT muestra *traffic share count* 1 en ambas) |
| `ping-tracert.png` | PC-OSPF1-V10 | `ping 172.16.0.10` y `tracert 172.16.0.10` | 0 % de pérdida; saltos 192.168.1.1 → 10.0.0.6 → 10.0.0.21 → 172.16.0.10 |

También sirven para revisar que todo esté bien: `show ip ospf neighbor` en R-CENTRAL (2 vecinos) y
`show ip eigrp neighbors` en R-CENTRAL (2 vecinos).

Si la proporción de *traffic share* no queda ~3:1 en Packet Tracer, ajustar `delay` en R-EIGRP-1 Se0/0/1 y
actualizar el cálculo en `docs/conceptos.md`.
