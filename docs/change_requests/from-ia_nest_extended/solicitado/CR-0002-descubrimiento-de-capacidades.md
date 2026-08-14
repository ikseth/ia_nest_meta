# CR-0002: descubrimiento de capacidades en REST

- id: CR-0002
- fecha: 2026-08-14
- capa origen: `ia_nest_extended`
- capa destino: `ia_nest_core`
- estado: propuesto

## Caso de uso motor

Por meta ADR 0007, una capa expone el contrato COMPLETO de la que envuelve:
reenvia sin alterar lo que no transforma, con un mecanismo generico y sin codigo
por capacidad. El objetivo es que una capacidad nueva del core sea alcanzable a
traves de las capas superiores SIN editarlas.

Verificado contra el codigo del core (2026-08-14), eso se cumple en parte:

- **REST**: `create_app` declara once rutas para las nueve capacidades. Un
  reenvio generico por ruta funciona: lo que la capa no sobreescribe, lo proxea.
  Una capacidad nueva del core seria alcanzable sin tocar la capa.
- **MCP**: cada capacidad se registra con nombre
  (`@server.tool(name="prompt.run")`), asi que un cliente MCP puede enumerarlas
  por el propio protocolo.
- **CLI**: no puede. Un CLI necesita ENUMERAR sus subcomandos para construir su
  ayuda y validar argumentos. Sin catalogo consultable, el CLI de una capa
  superior solo puede llevar una lista estatica, escrita a mano, que se
  desactualiza en cuanto el core anade una capacidad.

El resultado es que el invariante del ente lo cumplen dos interfaces de tres, y
la que falla es la que el operador usa a diario.

## Que se pide

Una forma de consultar, en tiempo de ejecucion y por REST, QUE capacidades expone
el core: sus nombres y lo minimo para invocarlas. No se pide un esquema completo
ni validacion de argumentos.

## Forma sugerida (no vinculante)

Una capacidad `capability.list`, hermana de `domain.list` y `model.list`, con
ruta `GET /capability/list`, que devuelva por capacidad: nombre, ruta y metodo
REST, si transporta identidad de request y si su respuesta es streaming.

El core ya tiene el dato: es el mismo que sostienen la tabla de rutas de
`rest.py` y los `@server.tool` de `mcp_server.py`. La forma preferible seria que
ese catalogo fuese la fuente unica de la que se derivan las tres interfaces, en
lugar de un cuarto sitio donde repetirlo.

## Impacto

- Core: adicion compatible -> MINOR. No altera ninguna capacidad existente.
- Extended: no bloquea. La Fase 7a arranca con lista estatica y pasa a
  descubrimiento cuando exista; el cambio queda contenido en su CLI.

## Alternativas consideradas

- **Descubrir por MCP desde extended.** Funciona hoy sin tocar el core, pero
  obliga a meter un cliente MCP en el CLI de la capa solo para pintar su ayuda, y
  ata una interfaz a otra sin motivo.
- **Lista estatica permanente en cada capa.** Es un dato del core replicado a
  mano en cada capa superior; se desactualiza en silencio y contradice la
  convencion transversal 6 (meta ADR 0008). Sirve como interino, no como destino.
- **Que el core no ofrezca descubrimiento y el CLI de cada capa se degrade.**
  Aceptable si el core prefiere no crecer, pero deja al operador con la unica
  interfaz que no hereda las capacidades nuevas.

## Tension doctrinal (honesta)

Ninguna de fondo, y conviene decirlo porque este CR llega despues de descartar
otro cambio mayor en el core. No invierte el grafo de dependencias, no reabre
`core ADR 0031`, `0035` ni `0040`, y no pide al core saber nada de las capas de
arriba: solo que sepa decir lo que el mismo hace.

Al contrario, apunta a una asimetria interna del propio core: su regla de
compatibilidad dice que "MCP y REST no deben tener logica distinta a la CLI", y
hoy MCP permite enumerar capacidades mientras REST y CLI no. Extended propone; el
core dispone.
