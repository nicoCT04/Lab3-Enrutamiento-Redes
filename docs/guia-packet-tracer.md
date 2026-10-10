# Guía para armar la topología en Packet Tracer

Esta guía cubre **solo la parte física**: colocar equipos, tarjetas y cables. Las IPs, VLANs y protocolos
se configuran después por CLI con los archivos de `configs/`.

## Inventario

| Tipo | Modelo | Cantidad | Nombres |
|---|---|---|---|
| Router | 1941 | 5 | R-OSPF-1, R-OSPF-2, R-CENTRAL, R-EIGRP-1, R-EIGRP-2 |
| Tarjeta serial | HWIC-2T | 6 | 1 por router de acceso, **2** en R-CENTRAL |
| Switch | 2960 | 4 | SW-OSPF-1, SW-OSPF-2, SW-EIGRP-1, SW-EIGRP-2 |
| PC | PC-PT | 7 | ver tabla de PCs |

## Paso 1 — Archivo

1. Abrir Packet Tracer → **File › New**.
2. Guardar de una vez como `Lab3-Enrutamiento.pkt` **dentro de la carpeta del repo** (`Lab3-Enrutamiento-Redes/`).

## Paso 2 — Routers y tarjetas seriales

1. Abajo a la izquierda: **Network Devices › Routers › 1941**. Arrastrar 5 routers al lienzo, ordenados como en el diagrama.
2. Para cada router: clic → pestaña **Config** → campo **Display Name** → poner su nombre (ej. `R-OSPF-1`).
3. Para cada router: pestaña **Physical**:
   - Apagar el router con el **interruptor** (botón en la imagen del equipo; el LED verde se apaga).
   - En la lista de módulos de la izquierda buscar **HWIC-2T** y arrastrarlo al **slot 0** (el slot vacío de la derecha en la parte trasera).
   - **Solo R-CENTRAL:** arrastrar un segundo **HWIC-2T** al **slot 1** (el otro slot vacío).
   - Volver a encender con el interruptor.
4. Verificación: en la pestaña **CLI** de cada router, al terminar de iniciar, escribir `no` si pregunta por el
   diálogo inicial y luego `show ip interface brief`. Deben aparecer `Serial0/0/0` y `Serial0/0/1`
   (y en R-CENTRAL además `Serial0/1/0` y `Serial0/1/1`).

## Paso 3 — Switches y PCs

1. **Network Devices › Switches › 2960**: 4 switches, cada uno junto a su router de acceso. Renombrarlos (Display Name).
2. **End Devices › PC**: 7 PCs. Renombrarlos según la tabla:

| PC | Va conectada a |
|---|---|
| PC-OSPF1-V10 | SW-OSPF-1 · Fa0/1 |
| PC-OSPF1-V30 | SW-OSPF-1 · Fa0/2 |
| PC-OSPF2 | SW-OSPF-2 · Fa0/1 |
| PC-EIGRP1-V10 | SW-EIGRP-1 · Fa0/1 |
| PC-EIGRP1-V20 | SW-EIGRP-1 · Fa0/2 |
| PC-EIGRP1-V99 | SW-EIGRP-1 · Fa0/3 |
| PC-EIGRP2 | SW-EIGRP-2 · Fa0/1 |

## Paso 4 — Cables seriales (6 enlaces)

En **Connections** (icono del rayo) elegir **Serial DCE** (el rayo con un relojito).
**Hacer clic primero en el router de la columna "Extremo A"**: ese lado queda como DCE.
No usar la conexión automática: elegir el puerto exacto en el menú que aparece.

| # | Extremo A (DCE) | Puerto A | Extremo B | Puerto B |
|---|---|---|---|---|
| 1 | R-OSPF-1 | Serial0/0/0 | R-OSPF-2 | Serial0/0/0 |
| 2 | R-OSPF-1 | Serial0/0/1 | R-CENTRAL | Serial0/0/0 |
| 3 | R-OSPF-2 | Serial0/0/1 | R-CENTRAL | Serial0/0/1 |
| 4 | R-EIGRP-1 | Serial0/0/0 | R-EIGRP-2 | Serial0/0/0 |
| 5 | R-EIGRP-1 | Serial0/0/1 | R-CENTRAL | Serial0/1/0 |
| 6 | R-EIGRP-2 | Serial0/0/1 | R-CENTRAL | Serial0/1/1 |

Es normal que los enlaces seriales queden en **rojo**: se ponen en verde cuando se configuren las IPs (`no shutdown`).

## Paso 5 — Cables Ethernet

En **Connections** elegir **Copper Straight-Through** (la línea negra continua).

| Desde | Puerto | Hacia | Puerto |
|---|---|---|---|
| R-OSPF-1 | GigabitEthernet0/0 | SW-OSPF-1 | GigabitEthernet0/1 |
| R-OSPF-2 | GigabitEthernet0/0 | SW-OSPF-2 | GigabitEthernet0/1 |
| R-EIGRP-1 | GigabitEthernet0/0 | SW-EIGRP-1 | GigabitEthernet0/1 |
| R-EIGRP-2 | GigabitEthernet0/0 | SW-EIGRP-2 | GigabitEthernet0/1 |
| Cada PC | FastEthernet0 | su switch | el Fa0/x de la tabla del paso 3 |

R-CENTRAL **no** lleva switch.

## Paso 6 — Presentación (opcional, suma en "buenas prácticas")

- Con la herramienta de **rectángulo** (barra superior) dibujar una zona para *OSPF · área 0* y otra para *EIGRP · AS 100*.
- Con **Place Note** (icono de nota) poner etiquetas con la red de cada enlace (ej. `10.0.0.0/30`) y de cada LAN.

## Paso 7 — Guardar y avisar

**File › Save** (Ctrl/Cmd + S). No hace falta configurar nada por CLI: eso se hace con los archivos de `configs/`.

## Problemas comunes

| Síntoma | Causa / solución |
|---|---|
| No aparece Serial0/0/x al conectar | El HWIC-2T no quedó insertado o el router se encendió sin la tarjeta. Revisar pestaña Physical. |
| No deja arrastrar la tarjeta | El router está encendido: apagarlo primero. |
| Quedó al revés el DCE | Borrar el cable (icono X) y volver a hacerlo empezando por el Extremo A. |
| Me equivoqué de puerto | No pasa nada: se ajusta en los configs. Avisar qué puerto quedó. |
