# Plan VLSM y tabla de direccionamiento

Metodología usada en los tres bloques:

1. Ordenar los segmentos de **mayor a menor** cantidad de hosts.
2. Para cada segmento buscar el menor `h` (bits de host) tal que `2^h − 2 ≥ hosts requeridos`.
3. Prefijo = `32 − h`. Tamaño del bloque (salto) = `2^h`.
4. Asignar a partir de la primera dirección libre; la siguiente subred empieza justo después del broadcast de la anterior.

Así ninguna subred se solapa y cada bloque queda alineado a su tamaño.

---

## 1. VLANs — lado OSPF · bloque `192.168.0.0/20`

Rango privado clase C (`192.168.0.0/16`), tomando un `/20`: `192.168.0.0 – 192.168.15.255` (4096 direcciones).

| Orden | Segmento | Hosts req. | Cálculo | Prefijo | Bloque |
|---|---|---|---|---|---|
| 1 | R-OSPF-2 · LAN datos | 200 | 2^8 − 2 = 254 ≥ 200 (2^7 − 2 = 126 no alcanza) | /24 | 256 |
| 2 | R-OSPF-1 · VLAN 10 datos | 50 | 2^6 − 2 = 62 ≥ 50 (2^5 − 2 = 30 no alcanza) | /26 | 64 |
| 3 | R-OSPF-1 · VLAN 30 admin/servidores | 10 | 2^4 − 2 = 14 ≥ 10 (2^3 − 2 = 6 no alcanza) | /28 | 16 |

| Segmento | Red | Máscara | Primer host | Último host | Broadcast | Gateway |
|---|---|---|---|---|---|---|
| R-OSPF-2 LAN | 192.168.0.0/24 | 255.255.255.0 | 192.168.0.1 | 192.168.0.254 | 192.168.0.255 | 192.168.0.1 |
| R-OSPF-1 VLAN 10 | 192.168.1.0/26 | 255.255.255.192 | 192.168.1.1 | 192.168.1.62 | 192.168.1.63 | 192.168.1.1 |
| R-OSPF-1 VLAN 30 | 192.168.1.64/28 | 255.255.255.240 | 192.168.1.65 | 192.168.1.78 | 192.168.1.79 | 192.168.1.65 |

Espacio libre para crecimiento: `192.168.1.80 – 192.168.15.255`.

---

## 2. VLANs — lado EIGRP · bloque `172.16.0.0/20`

Rango privado clase B (`172.16.0.0/12`), tomando un `/20`: `172.16.0.0 – 172.16.15.255` (4096 direcciones).

> El enunciado indica que R-EIGRP-1 lleva **3 VLANs**, pero la tabla de hosts solo define dos (10 y 20).
> Se agregó la **VLAN 99 — Administración** con 10 hosts para cumplir ambas condiciones.

| Orden | Segmento | Hosts req. | Cálculo | Prefijo | Bloque |
|---|---|---|---|---|---|
| 1 | R-EIGRP-2 · LAN datos | 100 | 2^7 − 2 = 126 ≥ 100 (2^6 − 2 = 62 no alcanza) | /25 | 128 |
| 2 | R-EIGRP-1 · VLAN 10 datos | 60 | 2^6 − 2 = 62 ≥ 60 | /26 | 64 |
| 3 | R-EIGRP-1 · VLAN 20 voz | 30 | 2^5 − 2 = 30 ≥ 30 | /27 | 32 |
| 4 | R-EIGRP-1 · VLAN 99 admin | 10 | 2^4 − 2 = 14 ≥ 10 | /28 | 16 |

| Segmento | Red | Máscara | Primer host | Último host | Broadcast | Gateway |
|---|---|---|---|---|---|---|
| R-EIGRP-2 LAN | 172.16.0.0/25 | 255.255.255.128 | 172.16.0.1 | 172.16.0.126 | 172.16.0.127 | 172.16.0.1 |
| R-EIGRP-1 VLAN 10 | 172.16.0.128/26 | 255.255.255.192 | 172.16.0.129 | 172.16.0.190 | 172.16.0.191 | 172.16.0.129 |
| R-EIGRP-1 VLAN 20 | 172.16.0.192/27 | 255.255.255.224 | 172.16.0.193 | 172.16.0.222 | 172.16.0.223 | 172.16.0.193 |
| R-EIGRP-1 VLAN 99 | 172.16.0.224/28 | 255.255.255.240 | 172.16.0.225 | 172.16.0.238 | 172.16.0.239 | 172.16.0.225 |

Espacio libre para crecimiento: `172.16.0.240 – 172.16.15.255`.

---

## 3. Enlaces router-router · bloque `10.0.0.0/24`

Cada enlace serial punto a punto tiene 2 equipos: 2^2 − 2 = 2 ≥ 2 → **/30** (255.255.255.252), bloque de 4.

| Enlace | Red | Extremo A (DCE) | Extremo B | Broadcast |
|---|---|---|---|---|
| R-OSPF-1 ↔ R-OSPF-2 | 10.0.0.0/30 | R-OSPF-1 Se0/0/0 · 10.0.0.1 | R-OSPF-2 Se0/0/0 · 10.0.0.2 | 10.0.0.3 |
| R-OSPF-1 ↔ R-CENTRAL | 10.0.0.4/30 | R-OSPF-1 Se0/0/1 · 10.0.0.5 | R-CENTRAL Se0/0/0 · 10.0.0.6 | 10.0.0.7 |
| R-OSPF-2 ↔ R-CENTRAL | 10.0.0.8/30 | R-OSPF-2 Se0/0/1 · 10.0.0.9 | R-CENTRAL Se0/0/1 · 10.0.0.10 | 10.0.0.11 |
| R-EIGRP-1 ↔ R-EIGRP-2 | 10.0.0.12/30 | R-EIGRP-1 Se0/0/0 · 10.0.0.13 | R-EIGRP-2 Se0/0/0 · 10.0.0.14 | 10.0.0.15 |
| R-EIGRP-1 ↔ R-CENTRAL | 10.0.0.16/30 | R-EIGRP-1 Se0/0/1 · 10.0.0.17 | R-CENTRAL Se0/1/0 · 10.0.0.18 | 10.0.0.19 |
| R-EIGRP-2 ↔ R-CENTRAL | 10.0.0.20/30 | R-EIGRP-2 Se0/0/1 · 10.0.0.21 | R-CENTRAL Se0/1/1 · 10.0.0.22 | 10.0.0.23 |

Espacio libre: `10.0.0.24 – 10.0.0.255`.

---

## 4. Tabla de interfaces por dispositivo

| Dispositivo | Interfaz | VLAN | Dirección IP | Máscara | Descripción |
|---|---|---|---|---|---|
| R-OSPF-1 | Gi0/0.10 | 10 | 192.168.1.1 | 255.255.255.192 | Gateway VLAN 10 Datos |
| R-OSPF-1 | Gi0/0.30 | 30 | 192.168.1.65 | 255.255.255.240 | Gateway VLAN 30 Admin |
| R-OSPF-1 | Se0/0/0 | — | 10.0.0.1 | 255.255.255.252 | Enlace a R-OSPF-2 |
| R-OSPF-1 | Se0/0/1 | — | 10.0.0.5 | 255.255.255.252 | Enlace a R-CENTRAL |
| R-OSPF-2 | Gi0/0 | — | 192.168.0.1 | 255.255.255.0 | LAN Datos |
| R-OSPF-2 | Se0/0/0 | — | 10.0.0.2 | 255.255.255.252 | Enlace a R-OSPF-1 |
| R-OSPF-2 | Se0/0/1 | — | 10.0.0.9 | 255.255.255.252 | Enlace a R-CENTRAL |
| R-CENTRAL | Se0/0/0 | — | 10.0.0.6 | 255.255.255.252 | Enlace a R-OSPF-1 |
| R-CENTRAL | Se0/0/1 | — | 10.0.0.10 | 255.255.255.252 | Enlace a R-OSPF-2 |
| R-CENTRAL | Se0/1/0 | — | 10.0.0.18 | 255.255.255.252 | Enlace a R-EIGRP-1 |
| R-CENTRAL | Se0/1/1 | — | 10.0.0.22 | 255.255.255.252 | Enlace a R-EIGRP-2 |
| R-EIGRP-1 | Gi0/0.10 | 10 | 172.16.0.129 | 255.255.255.192 | Gateway VLAN 10 Datos |
| R-EIGRP-1 | Gi0/0.20 | 20 | 172.16.0.193 | 255.255.255.224 | Gateway VLAN 20 Voz |
| R-EIGRP-1 | Gi0/0.99 | 99 | 172.16.0.225 | 255.255.255.240 | Gateway VLAN 99 Admin |
| R-EIGRP-1 | Se0/0/0 | — | 10.0.0.13 | 255.255.255.252 | Enlace a R-EIGRP-2 |
| R-EIGRP-1 | Se0/0/1 | — | 10.0.0.17 | 255.255.255.252 | Enlace a R-CENTRAL |
| R-EIGRP-2 | Gi0/0 | — | 172.16.0.1 | 255.255.255.128 | LAN Datos |
| R-EIGRP-2 | Se0/0/0 | — | 10.0.0.14 | 255.255.255.252 | Enlace a R-EIGRP-1 |
| R-EIGRP-2 | Se0/0/1 | — | 10.0.0.21 | 255.255.255.252 | Enlace a R-CENTRAL |

### Equipos finales

| Equipo | Switch / puerto | VLAN | IP | Máscara | Gateway |
|---|---|---|---|---|---|
| PC-OSPF1-V10 | SW-OSPF-1 Fa0/1 | 10 | 192.168.1.10 | 255.255.255.192 | 192.168.1.1 |
| PC-OSPF1-V30 | SW-OSPF-1 Fa0/2 | 30 | 192.168.1.70 | 255.255.255.240 | 192.168.1.65 |
| PC-OSPF2 | SW-OSPF-2 Fa0/1 | 1 | 192.168.0.10 | 255.255.255.0 | 192.168.0.1 |
| PC-EIGRP1-V10 | SW-EIGRP-1 Fa0/1 | 10 | 172.16.0.130 | 255.255.255.192 | 172.16.0.129 |
| PC-EIGRP1-V20 | SW-EIGRP-1 Fa0/2 | 20 | 172.16.0.200 | 255.255.255.224 | 172.16.0.193 |
| PC-EIGRP1-V99 | SW-EIGRP-1 Fa0/3 | 99 | 172.16.0.230 | 255.255.255.240 | 172.16.0.225 |
| PC-EIGRP2 | SW-EIGRP-2 Fa0/1 | 1 | 172.16.0.10 | 255.255.255.128 | 172.16.0.1 |

Los switches con VLANs se conectan al router por `Gi0/1` en modo **trunk** (router-on-a-stick).

---

## 5. Rutas de resumen (para la redistribución en R-CENTRAL)

Se buscan los bits iniciales comunes a todas las subredes de cada lado.

**Lado OSPF → se anuncia hacia EIGRP**

```
192.168.0.0   = 11000000.10101000.0000000|0.00000000
192.168.1.0   = 11000000.10101000.0000000|1.00000000
192.168.1.64  = 11000000.10101000.0000000|1.01000000
                └──── 16 bits + 7 bits = 23 bits comunes ────┘
```

Resumen: **`192.168.0.0/23`** (255.255.254.0) → cubre `192.168.0.0 – 192.168.1.255`.

**Lado EIGRP → se anuncia hacia OSPF**

```
172.16.0.0    = 10101100.00010000.00000000.|0 0000000
172.16.0.128  = 10101100.00010000.00000000.|1 0000000
172.16.0.192  = 10101100.00010000.00000000.|1 1000000
172.16.0.224  = 10101100.00010000.00000000.|1 1100000
                └──── 24 bits comunes ────┘
```

Resumen: **`172.16.0.0/24`** (255.255.255.0) → cubre `172.16.0.0 – 172.16.0.255`.

En ambos casos el resumen es el más ajustado posible: cubre todas las subredes del lado sin incluir
redes del otro dominio ni de los enlaces `10.0.0.0/24`.
