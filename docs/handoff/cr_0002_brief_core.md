# Aviso al core: extended CR-0002 (descubrimiento de capacidades)

Destinatario: hilo de `ia_nest_core`, rol disenador.
Emite: `ia_nest_extended`.
Decide: el core, reconciliado por el usuario. Este documento notifica; no decide.

Notificacion duradera exigida por el proceso de CR
(`docs/change_requests/README.md`). El CR completo, con caso de uso, forma
sugerida, impacto y alternativas, esta en
`docs/change_requests/from-ia_nest_extended/solicitado/CR-0002-descubrimiento-de-capacidades.md`.

## En una frase

Se pide una capacidad `capability.list` en REST: que el core sepa decir, en
tiempo de ejecucion, que capacidades expone.

## Por que ahora

`ia_nest_extended` esta disenando su interfaz de consumo (su Fase 7) sobre el
contrato uniforme del ente (meta ADR 0007): una capa reexpone el contrato
completo de la que envuelve, reenviando de forma generica lo que no transforma,
de modo que una capacidad nueva del core sea alcanzable a traves de las capas
superiores sin editarlas.

Al verificarlo contra el codigo del core (2026-08-14) resulta que eso se cumple
en REST (once rutas, proxeables por ruta) y en MCP (cada capacidad registrada con
nombre, enumerable por el protocolo), pero NO en CLI: un CLI necesita enumerar
sus subcomandos para construir su ayuda, y sin catalogo consultable solo puede
llevar una lista estatica escrita a mano.

## Lo que el core deberia sopesar

1. Si el dato ya existe repartido entre la tabla de rutas de `rest.py` y los
   `@server.tool` de `mcp_server.py`, ¿merece la pena unificarlo en un catalogo
   del que se deriven las tres interfaces, en vez de anadir un cuarto sitio donde
   repetirlo?
2. Su propia regla de compatibilidad dice que "MCP y REST no deben tener logica
   distinta a la CLI". Hoy MCP permite enumerar capacidades y REST/CLI no. Este
   CR se puede leer como cerrar esa asimetria, no como una capacidad ajena.
3. Impacto previsto: adicion compatible, MINOR. Sin cambios en capacidades
   existentes.

## Que NO se pide

- No se pide esquema completo de argumentos ni validacion.
- No se pide que el core conozca a las capas de arriba, ni que llame hacia
  arriba. Este CR no reabre `core ADR 0031`, `0035` ni `0040`.

## Urgencia

Ninguna. La Fase 7a de extended arranca con una lista estatica como interino y
migra a descubrimiento cuando exista. No bloquea trabajo.
