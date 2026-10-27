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
| H01P2 | Asignar cada entrada a un hito o descartarla con motivo | con el usuario | H01P1 | en curso |
| H01P3 | Deck: tarjetas a hitos y pasos; las que sobran a Terminado con puntero | Deck, tablero Casa | H01P2 | pendiente |
| H01P4 | `PLAN.md` de capa sin orden entre capas, con puntero al hito | un commit por repo | H01P2 | pendiente |
| H01P5 | Arranque de hilos: `AGENTS.md` de cada capa apunta a la hoja de ruta; memoria | repos de capa, memoria | H01P4 | pendiente |
| H01P6 | Orden de la casa: registro (borde v0.1.1; core registrado v0.4.1 y rocinante en v0.4.0), enlace del handoff de meta en `ia_nest_lab/enlaces.txt` | meta, lab | H00 | pendiente |

## Reglas del hito

- Sin big-bang: un commit por repo; cada reasignacion dudosa se consulta.
- El inventario es de solo lectura; las decisiones estan en `H01P2`.
- Semilla del inventario: el diagnostico del hilo de replanificacion.
