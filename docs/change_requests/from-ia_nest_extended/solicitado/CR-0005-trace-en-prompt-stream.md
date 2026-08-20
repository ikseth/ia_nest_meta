# CR-0005: `prompt.stream` no expone su trace, y su hermana si

- id: CR-0005
- fecha: 2026-08-20
- capa origen: `ia_nest_extended`
- capa destino: `ia_nest_core`
- estado: propuesto

## Caso de uso motor

La politica de persistencia de `ia_nest_extended` exige que cada engrama conserve
el `source_trace_id`: el identificador del request DEL CORE que lo origino. Es lo
que permite, meses despues, reconstruir de donde salio un recuerdo.

Al enriquecer `prompt.stream` -para que el streaming deje de perder memoria y
conocimiento- aparecio que ese dato no esta disponible. Verificado por el cable
contra `v0.4.0`, no por lectura de codigo:

- `reasoning.stream` emite en su evento `done` un objeto `trace`, y ese trace
  incluye `request_id`.
- `prompt.stream` emite en su `done` `finish_reason`, `model`, `reasoning`, `text`
  y los contadores de tokens. **No hay trace, ni `request_id`, ni cabecera que lo
  aporte.**

Son dos capacidades hermanas, del mismo tipo, con distinta capacidad de ser
auditadas.

## Que se pide

Que `prompt.stream` publique el identificador de su request, con la forma que el
core prefiera. La opcion natural es emitir el mismo objeto `trace` que ya emite
`reasoning.stream`, por simetria y porque la capa de encima ya sabe leerlo.

No se pide cambiar los eventos de token, ni el formato del flujo, ni nada de
`task.stream`.

## Impacto

- Core: adicion compatible sobre un evento existente -> PATCH o MINOR segun como
  lo cuente el core. No altera ninguna capacidad ni el formato del flujo.
- Extended: no bloquea. Mientras no exista, los engramas nacidos de
  `prompt.stream` guardan `source_trace_id` a NULO, declarado como limitacion
  conocida en su contrato y su CHANGELOG.

## Alternativas consideradas

- **Usar el identificador de la capa de encima.** Descartada, y es la unica que
  merece explicacion: ese campo significa "el request del core que origino esto".
  Rellenarlo con un identificador propio produce una traza que PARECE completa y
  no lo es, y engana a quien audite dentro de seis meses. Un hueco visible es
  mejor que un valor plausible y falso.
- **Relajar la politica de persistencia.** Seria cambiar una regla de la capa por
  una carencia de la de abajo, y ademas dejaria sin auditar tambien lo que hoy si
  se puede auditar.
- **Correlacionar por tiempo o por identidad.** Adivinar cual de varios requests
  concurrentes origino un engrama es exactamente el tipo de dato que parece
  correcto y no lo es.

## Tension doctrinal

Ninguna de fondo. No invierte el grafo de dependencias ni pide al core saber nada
de las capas de arriba: pide que una capacidad publique sobre si misma lo que su
hermana ya publica.

Si acaso, apunta a la misma asimetria interna que ya senalaron `CR-0002` y el
hallazgo de la CLI que ignora la variable de configuracion: capacidades del mismo
tipo con superficies distintas. La regla de compatibilidad del propio core dice
que sus interfaces no deben divergir; aqui divergen dos capacidades hermanas.

Extended propone; el core dispone. Un "no" explicito tambien sirve: entonces el
nulo deja de ser provisional y pasa a declararse permanente.
