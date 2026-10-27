# H00: modelo de orquestacion desde meta

Estado: en curso
Abierto: 2026-10-27

## Objetivo

Que el plan del ente viva en meta, en tres niveles, y que los hilos salgan de
meta (meta ADR 0014).

## Criterio de salida

El ADR y el esqueleto del plan estan mergeados en `main` de meta, con el charter
de `README.md` y el mandato de `AGENTS.md` ajustados al ADR. El criterio de
implantacion del modelo (meta ADR 0014) se comprueba al cerrar `H01`.

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H00P1 | meta ADR 0014 | meta | - | escrito, por revisar |
| H00P2 | Esqueleto: hoja de ruta, H00 y H01 | meta `docs/plan/` | H00P1 | escrito, por revisar |
| H00P3 | `README.md` y `AGENTS.md` alineados con el ADR | meta | H00P1 | escrito, por revisar |
| H00P4 | PR y merge | meta | H00P1-3 | pendiente (aprobacion del usuario) |

## Origen

Hilo de replanificacion del 2026-10-27
(`ia_nest_lab/comun/traspasos/meta/handoff_nuevo_hilo_2026-10-27_replanificacion.md`).
Diagnostico con diagramas: artifact privado del operador.
