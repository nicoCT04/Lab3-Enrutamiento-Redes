# Conceptos investigados (base para el PDF1)

## 1. OSPF y EIGRP en este diseño

| | OSPF | EIGRP |
|---|---|---|
| Tipo | Estado de enlace (Dijkstra / SPF) | Vector distancia avanzado (DUAL) |
| Métrica | Costo = 10^8 / ancho de banda (bps) | Compuesta: ancho de banda + retardo (K1 = K3 = 1) |
| Distancia administrativa | 110 | 90 interna / 170 externa |
| Identificador | Proceso 1, área 0 | AS 100 (debe coincidir en todos) |

- **Router ID**: identifica a cada router en el protocolo. Se fijó a mano (1.1.1.1, 2.2.2.2, 3.3.3.3,
  4.4.4.4, 5.5.5.5) para que no dependa de las IPs de las interfaces.
- **Wildcard**: inverso de la máscara (`/30` → `0.0.0.3`). Indica qué interfaces entran al protocolo.
- **passive-interface**: la red se sigue anunciando, pero no se envían hellos por esa interfaz. Se usa hacia
  las LANs porque ahí no hay routers vecinos; evita tráfico inútil y que un equipo ajeno forme vecindad.
- **no auto-summary**: EIGRP resumiría `172.16.x.x` a su clase (`/16`) en los bordes de red mayor. Se
  desactiva para anunciar cada subred VLSM con su máscara real.

## 2. Redistribución y métrica semilla

**Redistribución**: inyectar en un protocolo las rutas aprendidas por otro. Solo ocurre en R-CENTRAL, el
único router que corre ambos.

**Métrica semilla**: métrica inicial que recibe una ruta externa. Las métricas de OSPF (costo) y de EIGRP
(ancho de banda y retardo) no se pueden convertir entre sí, así que EIGRP necesita que se la indiquen; sin
ella las rutas redistribuidas no se instalan (métrica infinita).

`redistribute ospf 1 metric 1544 2000 255 1 1500`

| Valor | Significado | Por qué |
|---|---|---|
| 1544 | Ancho de banda en kbps | Igual al de los seriales T1 de la red |
| 2000 | Retardo en decenas de µs (20 000 µs) | Retardo por defecto de un serial |
| 255 | Confiabilidad (255 = 100 %) | Enlace confiable |
| 1 | Carga (1/255) | Enlace sin carga |
| 1500 | MTU en bytes | MTU estándar de Ethernet / serial |

Con esos valores las rutas OSPF entran a EIGRP como si estuvieran detrás de un serial más, lo que es coherente
con la topología.

## 3. Rutas externas OSPF: E1 vs E2

| | E1 | E2 (por defecto) |
|---|---|---|
| Métrica | Semilla + costo interno hasta el ASBR | Solo la semilla, igual en todo el dominio |
| Uso típico | Varios ASBR: cada router elige el más cercano | Un solo punto de salida |

**Se usó E1** (`metric-type 1`): la métrica de la ruta del lado EIGRP en la tabla de R-OSPF-1 y R-OSPF-2 refleja el
costo real del camino hasta R-CENTRAL, de forma coherente con los costos que se ajustaron para el balanceo. Si se
agregara otro ASBR, cada router elegiría el más cercano sin cambiar nada.

**`subnets`**: sin esta palabra OSPF solo redistribuye redes con su máscara de clase; con ella entran también
las subredes VLSM.

**EIGRP → OSPF en Packet Tracer**: con `redistribute eigrp 100` entraban a OSPF todas las subredes EIGRP una por
una (PT ignora la falta de `subnets`) y también el resumen `192.168.0.0/23` de vuelta. PT no tiene
`summary-address`, `route-map` ni `distribute-list` en OSPF para filtrarlas, así que el lado EIGRP se inyecta
como **una sola ruta externa E1**: `172.16.0.0/24`, una estática a Null0 redistribuida con
`redistribute static metric-type 1 subnets` (ver sección 6). Las PCs del lado OSPF alcanzan todo el lado EIGRP
por ese resumen.

## 4. Balanceo de carga en OSPF (ECMP)

OSPF instala hasta 4 rutas cuando tienen **exactamente el mismo costo** (ECMP).

- Serial: 10^8 / 1 544 000 = 64. GigabitEthernet: 10^8 / 10^9 → 1 (mínimo).
- R-OSPF-1 → 192.168.0.0/24 sin ajuste: directo 64 + 1 = **65**; vía R-CENTRAL 64 + 64 + 1 = **129**.
- Con `ip ospf cost 128` en el enlace directo (ambos extremos): directo 128 + 1 = **129** = vía R-CENTRAL.

Resultado: dos next-hops (10.0.0.2 y 10.0.0.6) con costo 129. Al configurar ambos extremos el balanceo
funciona en los dos sentidos.

## 5. Balanceo desigual en EIGRP (variance)

- **Feasible distance (FD)**: métrica total de la mejor ruta hacia el destino.
- **Reported distance (RD)**: métrica que anuncia el vecino desde él hasta el destino.
- **Successor**: vecino de la mejor ruta. **Feasible successor**: ruta de respaldo que cumple
  **RD < FD** (condición de factibilidad, garantiza que no haya bucle).
- **variance n**: se usan también las rutas factibles cuya métrica sea **menor que n × FD**. El tráfico se
  reparte en proporción **inversa** a la métrica: si la secundaria tiene el triple de métrica, recibe 1 de
  cada 4 paquetes → **75/25** con `variance 3`.

Métrica (K por defecto) = 256 × (10^7 / BW mínimo en kbps + Σ retardos en decenas de µs).
Con BW = 1544 → 10^7 / 1544 = 6476. Retardo del Gi0/0 de R-EIGRP-2 = 1.

Desde R-EIGRP-1 hacia 172.16.0.0/25 (LAN de R-EIGRP-2):

| Interfaz | `delay` | Resultado |
|---|---|---|
| R-EIGRP-1 Se0/0/0 (directo) | 2100 | FD = 256 × (6476 + 2100 + 1) = **2 195 712** |
| R-CENTRAL Se0/1/1 | 2000 (default) | RD = 256 × (6476 + 2000 + 1) = **2 170 112** < FD → feasible successor |
| R-EIGRP-1 Se0/0/1 (a R-CENTRAL) | 17253 | Métrica = 256 × (6476 + 17253 + 2000 + 1) = **6 586 880** |

6 586 880 / 2 195 712 ≈ **3** → 3:1 = 75 % por el enlace directo y 25 % vía R-CENTRAL.
Se usó 17253 y no 17254 porque con 17254 la métrica sería **exactamente** 3 × FD, y variance solo acepta
métricas menores a ese valor.

En Packet Tracer `show ip route 172.16.0.10` muestra las dos rutas instaladas, pero con *traffic share count* 1
en ambas: PT no simula el reparto desigual (en IOS real saldría ~3:1, por ejemplo 120 y 40). La evidencia es la
de configuración: las dos rutas en `show ip eigrp topology` con métricas en proporción 3:1.

Sin cambiar los retardos, RD = FD y la ruta vía R-CENTRAL no sería feasible successor: variance no la usaría.

## 6. Sumarización de rutas

**Qué es**: anunciar una sola ruta que cubra varias subredes contiguas.
**Qué resuelve**: tablas de enrutamiento más pequeñas, menos actualizaciones y menos CPU; además un cambio
dentro de una subred (por ejemplo, una VLAN que se cae) no se propaga al otro dominio.
**Cómo se calcula**:

1. Pasar a binario las subredes a resumir.
2. Contar los bits iniciales que tienen en común → ese es el prefijo.
3. Poner en 0 los bits restantes → esa es la dirección de red del resumen.

| Hacia | Subredes | Bits comunes | Resumen | Comando en R-CENTRAL |
|---|---|---|---|---|
| EIGRP | 192.168.0.0/24, 192.168.1.0/26, 192.168.1.64/28 | 23 | **192.168.0.0/23** | `ip summary-address eigrp 100 192.168.0.0 255.255.254.0` en Se0/1/0 y Se0/1/1 |
| OSPF | 172.16.0.0/25, .128/26, .192/27, .224/28 | 24 | **172.16.0.0/24** | `ip route 172.16.0.0 255.255.255.0 Null0` + `redistribute static metric-type 1 subnets` |

El cálculo binario completo está en [`vlsm-direccionamiento.md`](vlsm-direccionamiento.md#5-rutas-de-resumen-para-la-redistribución-en-r-central).

**Por qué esos resúmenes**: son los más ajustados posibles. Cubren todas las LANs de su lado sin incluir redes
del otro dominio ni de los enlaces `10.0.0.0/24`.

- **Hacia EIGRP**: `ip summary-address eigrp` en las interfaces que salen al dominio EIGRP. El router crea solo
  una ruta a **Null0** para el resumen.
- **Hacia OSPF**: en IOS real se usaría `summary-address` dentro de `router ospf` (las rutas EIGRP entran como
  externas, así que `area range` no sirve). **Packet Tracer no tiene ese comando**, así que se usó la técnica
  equivalente: una estática del resumen a **Null0** redistribuida como E1. R-CENTRAL sigue reenviando por las
  rutas EIGRP más específicas (prefijo más largo); la estática solo descarta el tráfico hacia partes del
  resumen que no existen, igual que la ruta Null0 que crearía `summary-address`.
