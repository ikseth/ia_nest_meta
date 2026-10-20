# Avisos al core: el planificador, visto desde el hito v0.5.0 de extended (2026-10-20)

Destinatario: hilo de `ia_nest_core`, rol disenador.
Emite: `ia_nest_extended`, tras verificacion en laboratorio.
Decide: el core, reconciliado por el usuario. Este documento informa; no decide.

No son Change Requests: ninguno pide una capacidad ni un cambio de contrato.
Son DEFECTOS medidos sobre el core `v0.4.0` mientras extended cerraba su deuda
D8 (eco de `task.run`). Continua los avisos del 2026-08-18 y del 2026-08-22.

Nota de convencion: no se nombran maquinas, rutas ni contenedores (convencion
transversal 5).

## Medida de la que salen

Seis hilos reales del operador (fisica, historia, linux, cocina, matematicas,
ingenieria), repetidos por `task.run` de extended en tres pasadas entre el
2026-10-13 y el 2026-10-14, roster real de 19 dominios, unas 48 llamadas al
planificador por pasada. En los dos casos se leyo el payload que el core
devolvio, no solo la respuesta final.

## Hallazgo 1: el planificador emite planes con dependencias circulares

Unas dos de cada 48 llamadas, `task.plan` devuelve un plan `pipeline` cuyas
dependencias forman un ciclo. El propio core lo rechaza despues en la
validacion del plan (`PlanDependencyError: pipeline plan contains a cycle`) y
`task.run` responde HTTP 400 al consumidor.

- Es intermitente: la misma peticion en una sesion nueva sale 200.
- El plan defectuoso lo genera el planificador del core, y quien lo valida
  tambien es el core. Hoy ninguno de los dos lo sanea ni vuelve a planificar:
  el error llega entero al consumidor.
- Extended lo cubre desde `v0.5.2` reintentando el plan una vez cuando recibe
  ese error concreto. En la pasada final el reintento salto una vez y salvo el
  turno. Es un parche en el consumidor, no un arreglo: cualquier otro
  consumidor del core se encuentra el 400.

## Hallazgo 2: al descomponer, el planificador descarta parte de la peticion

Peticion del operador, segundo turno de un hilo de cocina: una nueva lista de
cenas *incluyendo la receta*. El planificador la reduce a una sola subtarea,
"Crear una lista de cenas bajo los criterios especificados": la parte de la
receta desaparece del plan. El modelo de la subtarea responde con el menu del
turno anterior y sin recetas.

- No es el historial (eso era D8 de extended, cerrada): el plan ya llega sin
  la mitad del encargo.
- `requirements_covered` no lo detecta, porque los requisitos se derivan del
  plan ya recortado. Es la misma familia que la ficha v0.4/0008 (requisitos
  perdidos y no declarados), vista ahora en un hilo real y no en un banco.
- Se vio tambien en un hilo de matematicas antes de `v0.5.2`; tras el cambio
  de reescritura de extended, ese caso ya conserva el matiz. El de cocina
  persiste.

## Lo que extended no sabe

- Si los dos hallazgos comparten causa: los dos salen del mismo prompt de
  planificacion.
- Si la linea de la senal de vacio (ADR 0051) los toca: no producen respuestas
  vacias, asi que en principio no.

La evidencia completa (hilos, payloads y tablas por pasada) vive en las notas de
laboratorio de extended, fuera de git.
