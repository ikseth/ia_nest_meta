# Brief de handoff: decision del core sobre extended CR-0001

Destinatario: hilo de `ia_nest_core`, rol disenador.
Autor: hilo de gobernanza (`ia_nest_meta`).
Decide: el usuario, en reconciliacion. Este brief no decide nada.
Base: `ia_nest_core` en `main`, commit `c0645af`.
Fecha: 2026-07-26

## Lee antes

1. El CR:
   `ia_nest_meta/docs/change_requests/from-ia_nest_extended/solicitado/CR-0001-checkpoint-enriquecimiento-por-subtarea.md`
2. El proceso: `ia_nest_meta/docs/change_requests/README.md` (regla de oro:
   la capa origen PROPONE, la destino DISPONE; la forma sugerida NO es
   vinculante).
3. `core ADR 0036` (contrato de `task.run`), `core ADR 0034` (checkpoints de
   supervision), `core ADR 0035` (retirada de `MemoryPort`), `core ADR 0031`
   (enriquecimiento vs herramientas).

## Que se decide

`ia_nest_extended` pide un punto de enriquecimiento POR SUBTAREA en `task.run`:
una costura donde la capa pueda inyectar contexto del dominio de cada subtarea
ANTES de ejecutar su modelo. Motivo: enriquecer upfront obliga a meter el
conocimiento de todos los dominios candidatos en el contexto de todas las
subtareas, y no cabe en el presupuesto de tokens observado.

El core decide UNA de estas, con su forma y su version objetivo:

- aceptar (con la forma que el core elija),
- reformular,
- rechazar con motivo.

## Puntos a verificar antes de decidir

El CR describe su forma sugerida como "un checkpoint hermano de los
observables/vetables del core ADR 0034". Antes de evaluarla, conviene
comprobar en el codigo si esa descripcion se sostiene. Tres puntos:

1. **Direccionalidad de los checkpoints actuales.**
   `src/ianest_core/runtime/task_runtime.py`, checkpoints emitidos en las lineas
   206, 210, 219, 222, 224 y 264 (`task_received`, `plan_ready`, `subtask_done`,
   `combine_ready`, `iteration_end`, `task_done`), con el helper en la linea 999.
   Verifica si emiten en una sola direccion o si alguno admite valor de vuelta.
   El CR necesita que la costura DEVUELVA un bloque de contexto.

2. **Que dice ADR 0036 sobre el camino de vuelta.**
   `docs/decision_records/0036-contrato-task-run.md`, linea 27: la semantica de
   VETO se declara "extension futura"; ahi solo se fijan los puntos de anclaje.
   Valora si lo que pide el CR pertenece a esa extension diferida o es otra
   cosa.

3. **Donde cae el punto de insercion, en tiempo de ejecucion.**
   El CR lo situa entre ROUTE y la ejecucion de cada subtarea. En el codigo eso
   es `_run_subtask` (linea 931), invocado desde `_fan_out` (linea 914) dentro
   de un `ThreadPoolExecutor` (linea 921). Observa que `subtask_done` se emite
   desde el bucle principal (linea 219) DESPUES de que `_fan_out` retorne.
   Determina que implica eso para una costura que debe actuar ANTES de la
   llamada al modelo de cada subtarea, y si condiciona la forma viable.

## Lo que este brief NO decide

No propone forma. Si de la verificacion sale que la costura viable no es un
checkpoint sino otro mecanismo, esa conclusion y su tension doctrinal (en
particular respecto a `core ADR 0035`) son del core, no del taller. Meta
gobierna el canal, no el diseno de la capa destino.

Tampoco decide el numero de version: se propone impacto, no se corta tag
(`ia_nest_meta/docs/POLITICA_SEMVER.md`).

## Criterios de salida

Si ACEPTA o REFORMULA:

- ADR propio en `docs/decision_records/` con la forma elegida y su motivo.
- Item en `docs/PLAN.md` con version objetivo.
- Impacto de version declarado (el CR preve MINOR) y linea en `CHANGELOG.md`.

Si RECHAZA:

- Motivo escrito, suficiente para que extended replantee o lo asuma.

En ambos casos, avisar al hilo de gobernanza para reflejar el estado en el CR y,
si procede, moverlo a `resuelto/`.

## Regla que manda sobre todas

Ante ambiguedad, PARA y pregunta. No rellenes un hueco por inferencia: en modo
ciego eso introduce diseno no reconciliado
(`ia_nest_meta/docs/DOCTRINA_MULTI_IA.md`).
