# Lab 3 — Enrutamiento OSPF / EIGRP con redistribución

**Curso:** Redes — Sección 10 · **Docente:** Kevin Velásquez · **Modalidad:** en parejas

Diseño e implementación en Packet Tracer de una red de dos sedes: una corre **OSPF (área 0)** y la otra
**EIGRP (AS 100)**, unidas por **R-CENTRAL**, que hace redistribución mutua entre ambos dominios.
Incluye VLSM, VLANs con enrutamiento entre VLANs, balanceo ECMP (OSPF), balanceo desigual 75/25
(EIGRP) y sumarización de rutas.

## Topología

```
        OSPF · área 0                                   EIGRP · AS 100

   [SW-OSPF-1]                                              [SW-EIGRP-1]
        |                                                        |
    R-OSPF-1 ----------.                          .---------- R-EIGRP-1
        |               \                        /               |
  enlace directo         +------ R-CENTRAL ------+          enlace directo
        |               /   (redistribución)     \               |
    R-OSPF-2 ----------'                          '---------- R-EIGRP-2
        |                                                        |
   [SW-OSPF-2]                                              [SW-EIGRP-2]
```

## Estructura del repositorio

| Carpeta / archivo | Contenido |
|---|---|
| `docs/vlsm-direccionamiento.md` | Cálculo VLSM paso a paso y tabla de direccionamiento (base para el PDF1) |
| `docs/conceptos.md` | Redistribución, E1/E2, ECMP, variance y sumarización con sus cálculos (base para el PDF1) |
| `docs/referencias/` | Enunciado del laboratorio y material de apoyo del curso |
| `configs/` | Configuración IOS de cada router y switch, lista para pegar en la CLI |
| `evidencias/` | Salidas de `show` y capturas de ping/tracert (base para el PDF2) |
| `Lab3-Enrutamiento.pkt` | Archivo de Packet Tracer |
| `PDF2-Evidencias.pdf` | Tablas de enrutamiento, balanceo OSPF/EIGRP, ping y tracert |

## Bloques de direccionamiento

| Uso | Bloque base | Resumen anunciado |
|---|---|---|
| VLANs — lado OSPF | `192.168.0.0/20` | `192.168.0.0/23` |
| VLANs — lado EIGRP | `172.16.0.0/20` | `172.16.0.0/24` |
| Enlaces router-router | `10.0.0.0/24` (seis `/30`) | — |

El detalle de cada subred está en [`docs/vlsm-direccionamiento.md`](docs/vlsm-direccionamiento.md).

## Entregables

- `Lab3-Enrutamiento.pkt` — red funcional con los 5 routers, switches, VLANs y protocolos.
- **PDF1** — cálculos VLSM a mano y conceptos investigados.
- **PDF2** — `show ip route` de los 5 routers, evidencia de balanceo OSPF/EIGRP, ping y tracert extremo a extremo.
