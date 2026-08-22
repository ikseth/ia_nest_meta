# Avisos al core: lo que aparecio calibrando el laboratorio (2026-08-22)

Destinatario: hilo de `ia_nest_core`, rol disenador.
Emite: `ia_nest_extended`, tras verificacion en laboratorio.
Decide: el core, reconciliado por el usuario. Este documento informa; no decide.

No son Change Requests: ninguno pide una capacidad ni un cambio de contrato.
Son HALLAZGOS medidos. Continua el brief del 2026-08-18, y uno de sus hallazgos
reaparece con evidencia nueva.

Nota de convencion: no se nombran maquinas, rutas ni contenedores. Cada hallazgo
se describe por su forma, no por su instalacion (convencion transversal 5).

## Lo primero: dos avisos anteriores se RETIRAN

Honestidad de medida antes que volumen de hallazgos. Al revisar una sesion real
aparecieron seis sospechas; cuatro no sobrevivieron a calibrar el laboratorio, y
conviene decirlo porque tres de ellas se habrian mandado aqui como defectos.

El laboratorio corria el `core.lab.example.yaml` que el core publica: seis
dominios, sin `cultura`, con `max_tokens: 512` para todos. Con ese catalogo:

- El planificador mandaba preguntas de cine al dominio `codigo` (6 de 6
  subtareas). Parecia un defecto de enrutado. **No lo es.**
- `task.run` no convergia nunca: 14 iteraciones, 14 `rerun`, cero `done`.
  Parecia un bucle roto. **Tampoco lo es.**

Con el roster real -19 dominios- las dos cosas desaparecen:

- **Concordancia planificador / `domain.route`: 22 de 25 subtareas (88%)**, y
  las tres discrepancias son de vecindad semantica (humanidades/cultura,
  finanzas/matematicas), no errores. El planificador enruta bien cuando el
  catalogo cubre el espacio de preguntas.
- La misma tarea que no convergia en 122 s con cuatro iteraciones **converge en
  una sola y en 33 s**. Los `rerun` no eran un bucle roto: eran el evaluador
  haciendo su trabajo sobre respuestas que un catalogo pobre habia mandado a
  modelos equivocados.

Queda de aquello un apunte menor, el hallazgo 3 de abajo.

## Hallazgo 1: el gate da verde a respuestas falsas, y la cuarta linea no basta

El brief del 2026-08-18 sugirio anadir al gate una cuarta linea,
`stop_reason == task_done`, porque las tres existentes no miran COMO termino la
tarea. Sigue siendo buena idea. Lo que aporta hoy es que **no basta**, y hay
contraejemplos medidos.

Cuatro pasadas de la misma pregunta factual -sinopsis de una pelicula conocida y
si tiene precuelas o secuelas-, con el roster real y `effort: high`:

| pasada | subtareas | `stop_reason` | `degradations` | contenido |
|---|---|---|---|---|
| 1 | 4 | `task_done` | `[]` | sinopsis falsa; dice que la obra es "adaptacion" de si misma |
| 2 | 4 | `task_done` | `[]` | sinopsis correcta; niega la precuela, que existe |
| 3 | 16 | `task_done` | `[]` | sin contenido: "explora la lucha entre lo real y lo imaginario" |
| 4 | 4 | `task_done` | `[]` | inventa un personaje principal y su actor, ninguno de los dos existe |

**4 de 4 responden mal a la pregunta explicita, y 4 de 4 salen en verde.** Las
cuatro habrian pasado tambien la cuarta linea propuesta, porque el evaluador
acepto: `stop_reason` es `task_done` en todas.

Lo que esto dice no es que el evaluador este roto -se sondeo y con un caso
trivial devuelve `done` de forma estable-, sino que **ninguna de las senales del
gate mira si la respuesta es cierta**, y para preguntas factuales verificables
el verde no informa. Es una limitacion honesta del diseno actual, no un bug; se
senala porque el gate se usa para cerrar fases.

Contraste util: con las mismas preguntas respondidas de una pieza por
`prompt.run`, sin descomponer, las respuestas fueron correctas. La degradacion
aparece al trocear y recombinar preguntas multiparte, no en el modelo.

## Hallazgo 2: el planificador no es estable ante el mismo prompt

En esas cuatro pasadas, mismo prompt y mismo `effort`, el plan salio con **4, 16,
4 y 4** subtareas. La de 16 produjo la respuesta mas vacia de las cuatro.

No se propone forma; se senala porque afecta al coste (la pasada de 16 subtareas
gasta cuatro veces mas para dar menos) y porque hace poco reproducible cualquier
medida sobre `task.run`.

## Hallazgo 3: el ejemplo publicado tiene el fallo que su propio comentario describe

`config/core.lab.example.yaml` documenta, en un comentario del bloque
`orchestration`, que un modelo de razonamiento no sirve como planificador porque
emite su cadena de pensamiento antes de la respuesta y con `max_tokens: 512` el
pensamiento agota el presupuesto. La causa esta bien diagnosticada y el
planificador se arreglo.

Pero el mismo fichero deja el dominio `razonamiento` servido por ese modelo con
el perfil `default`, de 512 tokens, y el comentario lo justifica diciendo que
ese dominio "es su sitio". Medido: las subtareas de ese dominio salen con
`finish_reason: length` y **cadena vacia**, o con texto corrompido. El
combinador integra el vacio como si fuera un resultado.

El despliegue real del laboratorio ya no tiene el problema, porque su roster
define un perfil aparte con presupuesto mayor. El aviso es sobre el ARTEFACTO
PUBLICADO: quien arranque por el ejemplo se encuentra el fallo servido de fabrica.

Sugerencia, no vinculante: que el ejemplo lleve el perfil de razonamiento que el
propio laboratorio ya usa.

## Lo que NO se pide

Se considero emitir un CR para que `task.run` declarase que su evaluador no
acepto la respuesta -extended lo necesita para no memorizar como candidato lo
que el core corto-. **Se descarto tras comprobarlo por el cable: el dato ya
viaja.** `stop_reason` esta en la respuesta, y extended no lo leia. Corregido en
la capa de arriba, donde tocaba. Queda anotado aqui porque la regla de oro del
proceso de CR -si se puede resolver en la propia capa, no es un CR- se aplico
sobre una comprobacion, no sobre una impresion.
