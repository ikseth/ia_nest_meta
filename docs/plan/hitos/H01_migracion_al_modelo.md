# H01: migrar plan, tarjetas y repos al modelo

Estado: en curso
Abierto: 2026-10-27

## Objetivo

Que todo lo vivo de IA_NEST tenga un unico sitio en el plan de meta, y que no
queden dos sistemas conviviendo (meta ADR 0014). Incluye poner orden en el
estado registrado.

## Criterio de salida

Los cuatro puntos del criterio de meta ADR 0014, mas:

- Cada entrada del inventario (`H01P1`) tiene un unico hito o un "descartado
  porque ...".

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H01P1 | Inventario de lo vivo: tarjetas, `PLAN.md` de capas, handoffs, issues y CR abiertos | solo lectura, todos los repos y Deck | H00 | hecho: [inventario](../pasos/H01P1_inventario.md) |
| H01P2 | Asignar cada entrada a un hito o descartarla con motivo | con el usuario | H01P1 | hecho: cinco bloques validados el 2026-10-27 |
| H01P3 | Deck: tarjetas a hitos y pasos; las que sobran a Terminado con puntero | Deck, tablero Casa | H01P2 | hecho: #144 (H01) creada; #66, #124 (H07), #125 (H08), #135 (H04) reconvertidas; #123, #136, #96, #102 a Terminado |
| H01P4 | `PLAN.md` de capa sin orden entre capas, con puntero al hito | un commit por repo | H01P2 | hecho en ramas locales `plan/H01-migracion` (extended, edge, agents); falta PR |
| H01P5 | Arranque de hilos: `AGENTS.md` de cada capa apunta a la hoja de ruta; memoria | repos de capa, memoria | H01P4 | hecho en ramas locales (extended, edge, agents, conscience, web) y memoria; falta PR y el core (aprobacion) |
| H01P6 | Orden de la casa: registro (borde v0.1.1; core registrado v0.4.1 y rocinante en v0.4.0), enlace del handoff de meta en `ia_nest_lab/enlaces.txt` | meta, lab | H00 | hecho, salvo la accion del operador de abajo |
| H01P7 | Saneamiento de git: ramas remotas antiguas (unas 50 en extended, 5 en edge, 10 en meta; revisar tambien core). Lista por repo con lo que cada una tiene fuera de `main`; se borran solo las contenidas en `main` y con aprobacion del usuario; las que tengan trabajo suelto se consultan una a una | todos los repos del ente | H01P6 | pendiente |

## Hecho en H01P6

- Registro: fila del borde a `v0.1.1`, verificada en vivo. El core sigue en
  `v0.4.1`: el registro lleva lo publicado, no lo desplegado.
- CR-0003, CR-0004 y CR-0005 a `resuelto/`, entregados en core `v0.4.1`.
- core#36 cerrado con puntero (aprobado por el usuario).
- Ramas remotas ya mergeadas borradas (edge, agents, extended, meta) y
  `web_chat` archivado (aprobado por el usuario).
- Enlaces de los handoffs de meta en `ia_nest_lab/enlaces.txt` (ya estaban).

## Accion del operador

- rocinante: actualizar el core a `v0.4.1` por el instalador desde el tag y
  refrescar la copia de `extended.setup.conf` (inventario A07, E03).

## Reglas del hito

- Sin big-bang: un commit por repo; cada reasignacion dudosa se consulta.
- El inventario es de solo lectura; las decisiones estan en `H01P2`.
- Semilla del inventario: el diagnostico del hilo de replanificacion.
