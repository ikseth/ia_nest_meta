# CR-0003: identificador de build en la respuesta del core

- id: CR-0003
- fecha: 2026-08-18
- capa origen: `ia_nest_extended`
- capa destino: `ia_nest_core`
- estado: propuesto

## Caso de uso motor

`capability.list` y `runtime.health` devuelven `core_version`, que es la cifra con
la que las capas de encima fijan su vinculo por SemVer (core ADR 0046). Pero
`core_version` sale del manifiesto del paquete, no del artefacto que esta
sirviendo, de modo que **no distingue dos builds de la misma version**.

Medido el 2026-08-18, sin teoria de por medio. En un entorno donde el codigo se
actualizo pero el servicio no se reinicio, la respuesta de `capability.list` era
identica antes y despues del cambio: `core_version` decia lo mismo en dos
artefactos separados por veintiun commits, uno de los cuales anadia un campo
publico a la respuesta de `task.plan`.

Consecuencia real: tres horas de medidas invalidas contra la REST, con un
diagnostico inicial equivocado -"el core no cumple su contrato, le falta
`degradations`"- cuando el contrato SI se cumplia en el codigo y lo que fallaba
era la premisa de que el proceso servia ese codigo. La verificacion por CLI y la
suite no lo detectan, porque leen del disco en cada invocacion; solo lo sufre
quien consume por RED, que es precisamente como consumen las capas de encima.

No es un caso de mala operacion aislado: es la clase de error que la respuesta del
core podria hacer imposible, y hoy no puede.

## Que se pide

Que la respuesta del core permita saber QUE ARTEFACTO esta sirviendo, no solo que
version declara su manifiesto. Con eso, un consumidor puede afirmar contra que
esta midiendo, y una discrepancia entre codigo y proceso se detecta en una
llamada.

No se pide telemetria nueva ni una capacidad nueva.

## Forma sugerida (no vinculante)

Un campo hermano de `core_version` en `runtime.health` y en `capability.list` con
el identificador del commit desplegado, resuelto en tiempo de ejecucion y con un
valor explicito -no vacio, no ausente- cuando no se pueda determinar; por
ejemplo `unknown`, coherente con como `runtime.health` ya trata lo que no puede
sondear.

Distincion que conviene preservar: `core_version` es la cifra del CONTRATO, y es
la que gobierna el vinculo por SemVer. El identificador de build es
OBSERVABILIDAD, y no debe usarse para fijar dependencias. Son dos ejes y no
sustituye uno al otro.

## Impacto

- Core: adicion compatible -> PATCH o MINOR, segun donde lo situe. No altera
  ninguna capacidad existente ni la semantica de `core_version`.
- Extended: no bloquea nada. Lo consumira en su verificacion de laboratorio para
  poder declarar contra que artefacto mide.

## Alternativas consideradas

- **Que cada consumidor compare el arranque del servicio con el estado del repo.**
  Es lo que hubo que hacer, y funciona, pero es un procedimiento que depende de
  que alguien se acuerde y que solo puede ejecutar quien tenga acceso a la maquina
  del core. Un consumidor remoto no puede.
- **Reiniciar siempre tras actualizar, y confiar en la disciplina.** Es correcto y
  hay que hacerlo igual, pero no es verificable desde fuera: la ausencia de una
  comprobacion no se detecta hasta que da un resultado falso.
- **Deducirlo de la forma de la respuesta.** Es lo que acabo pasando -faltaba un
  campo, y de ahi se dedujo el build-, y es exactamente lo que no debe hacer
  falta: obliga a conocer el historial del core para interpretar su respuesta.

## Tension doctrinal (honesta)

Poca, y conviene decir donde esta. El core es deliberadamente parco en lo que
publica, y su contrato dice que `runtime.health` informa de lo que puede sondear
sin filtrar detalles de instalacion. Un identificador de commit no es un detalle
de red ni una credencial, asi que no roza la convencion transversal 5; pero si es
informacion sobre el despliegue, y esa frontera la dibuja el core.

La otra cara: sin esto, el propio core ya se equivoco una vez el mismo dia -su
informe de laboratorio afirma un smoke "sobre main exacto" que por REST no lo
fue-. El campo protege al core de si mismo antes que a sus consumidores.

Extended propone; el core dispone.
