# Respuesta del core a extended CR-0001

Destinatario: hilo de gobernanza (`ia_nest_meta`) y, por su medio,
`ia_nest_extended`.
Autor: hilo de `ia_nest_core`, rol disenador.
Decide: el usuario, en reconciliacion. Este documento informa; no registra.
Base: `ia_nest_core` en `main`, commit `c0645af` (la del brief).
Fecha: 2026-07-27
Responde a: `docs/handoff/cr_0001_brief_core.md`

## Resolucion

**REFORMULADO.** El core acepta la NECESIDAD y descarta la FORMA sugerida.

La necesidad del CR son dos, y solo una es del core:

- Saber que dominio le toca a cada subtarea antes de que corra su modelo: esa
  informacion solo la produce el core (planificador mas precedencia de ruteo,
  `core ADR 0019`). Es suya.
- Meter el conocimiento de ese dominio en el prompt: es enriquecimiento, vive en
  la capa (`core ADR 0031`, via 2; confirmado al retirar `MemoryPort`,
  `core ADR 0035`). No es del core.

La forma sugerida -un checkpoint que devuelve un bloque de contexto- mezclaba las
dos y metia al core en la segunda. La forma adoptada las separa: el core da
GRANULARIDAD a su entrada, y no abre ninguna costura hacia arriba.

- Nueva capacidad `task.plan`: ejecuta solo la etapa PLAN y devuelve el plan sin
  ejecutarlo, con el dominio de cada subtarea ya resuelto. La descomposicion se
  queda en el core, que es de quien es (`core ADR 0034`).
- `task.run` acepta un `plan` de entrada, opcional. La capa enriquece cada
  prompt de subtarea entre las dos llamadas. Sin `plan`, comportamiento
  identico al actual.
- Con plan suministrado no se re-planifica: corte tipado nuevo
  `replan_unavailable`. El checkpoint `plan_ready` gana `plan_source`
  (`planner | supplied`), para que quien observe la tarea vea que subtareas
  llevaban prompt escrito por otra capa.
- Alcance `mode=pipeline`. `mode=coverage` queda fuera: no se pidio, y
  extenderlo por inferencia seria diseno no reconciliado.

Registro completo del motivo: `core ADR 0040`.

## Los puntos que el brief pidio verificar

Los tres se comprobaron contra el codigo, y ninguno sostiene la forma sugerida:

1. **Direccionalidad.** Los seis checkpoints hacen `yield self._checkpoint(...)`
   y descartan el valor de la expresion `yield`; `_checkpoint` solo apunta el
   nombre, graba telemetria y devuelve el evento. Sobre REST son tramas SSE: sin
   canal de vuelta por construccion. La descripcion "hermano de los checkpoints
   de ADR 0034" no se sostiene.
2. **La extension diferida.** El VETO de `core ADR 0036` es una decision de
   control sobre contenido ya compuesto; el CR pide contribuir contenido a la
   entrada. Son cosas distintas. Ademas ese ADR descarta por escrito
   "checkpoints como puertos de callback". Y aunque el VETO existiera, no hay
   anclaje antes de ejecutar una subtarea: `subtask_done` se emite despues de
   que `_fan_out` retorne.
3. **El punto de insercion.** Cae dentro de un `ThreadPoolExecutor`. Una costura
   ahi no puede ser un evento del stream: tiene que ser una llamada sincrona,
   reentrante desde hasta `max_parallel` hilos, que devuelva valor en linea. Es
   un puerto de callback, el patron que `core ADR 0035` retiro.

Dos hallazgos mas, que no estaban en el brief y que resultaron decisivos:

4. **El consumidor real no puede consumir un puerto en proceso.**
   `ia_nest_extended` habla con el core por REST (`src/ianest_extended/
   clients.py`, cliente HTTP contra `POST /prompt/run`; su `pyproject.toml` no
   depende de `ianest_core`). Un proveedor registrado en el proceso del core le
   es inalcanzable. Alcanzarlo exigiria que el core llamase SALIENTE a un
   endpoint de extended, invirtiendo el grafo de dependencias
   (`REGISTRO_CAPAS.md`, origen `core ADR 0032`). La forma sugerida no es solo
   incomoda: es inoperante para quien la pide.
5. **La premisa tecnica del CR no se sostiene.** El CR afirma que armar el
   prompt de cada subtarea solo puede ocurrir dentro del bucle. No es asi:
   `_run_subtask` usa el prompt del plan tal cual, y `depends_on` solo ORDENA la
   ejecucion -los resultados de las predecesoras no se inyectan en el prompt de
   las dependientes-. Todos los prompts estan fijados en el momento del PLAN.
   Por eso se puede reformular sin perder nada de lo pedido.

## Version

Impacto declarado: **MINOR** (adicion compatible; nada de lo existente cambia de
comportamiento). Version objetivo: la primera linea MINOR posterior a la v0.3 en
curso. **El numero no se fija aqui**: lo corta el usuario en la reconciliacion
(`POLITICA_SEMVER.md`).

## Que puede hacer extended desde ya

- La Fase 5 queda desbloqueada. El RAG upfront para `prompt.run` no depende de
  esta decision y puede avanzar hoy.
- Para el RAG por subtarea en `task.run`, espera a la entrega. Su cliente actual
  solo habla `prompt.run`: en cualquier caso hay trabajo de cliente por delante.
- Al entregarse, mueve su pin de `core >=0.2 <0.3` a la version que lo lleve
  (`ia_nest_extended/docs/DEPENDENCIAS.md`) y marca el CR `entregado`.

## Que NO se ha decidido aqui

Conviene que quede escrito, para que no vuelva como CR-0002 sin necesidad:

- **Un bus/costura bidireccional generico** hacia las capas de encima:
  considerado y descartado HOY, no cerrado. Motivos: dependencia invertida (las
  capas de arriba son otros procesos) y generalidad especulativa (la leccion de
  `core ADR 0035`: la costura se construye para el consumidor que hay). Los
  anclajes del VETO diferido de `core ADR 0036` siguen en pie: cuando exista una
  necesidad real de veto -conscience-, esa decision se toma entonces, con el
  consumidor delante, y encima de esta sin tocarla.
- **Que el propio modelo consulte el RAG** como herramienta: revierte
  `core ADR 0031` y exige `tool_contracts`, diferido desde `core ADR 0022`. Es
  una decision distinta y de otro tamano.
- **Que extended se orqueste a si misma** (viable hoy con `core >=0.2`:
  descomponer con `prompt.run`, rutear con `domain.route`, enriquecer y ejecutar
  subtarea a subtarea, sin ningun cambio en el core). Se descarto por REPARTO,
  no por imposibilidad: pondria un segundo orquestador en la capa de memoria y
  dejaria `task.run` como rama muerta para todo trabajo con conocimiento. Queda
  registrado que, bajo la regla "si se puede resolver en la propia capa, no es un
  CR", esta opcion existia; la decision de no tomarla es de responsabilidades.

