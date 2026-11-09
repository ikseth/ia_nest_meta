# CR-0001: el core invoca herramientas de una capa externa por MCP, desde un dominio y con politica por perfil

- id: CR-0001
- fecha: 2026-11-06
- capa origen: `ia_nest_external_home_assistant`
- capa destino: `ia_nest_core`
- estado: entregado

## Caso de uso motor

Hito H09 de meta, tramo 2: el ente ve el estado de la casa y la controla. Hoy
la voz de Home Assistant ya conversa a traves del ente (tramo 1, H09P4), pero
el ente no puede leer el estado de una entidad ni encender una luz: el core no
tiene ninguna frontera hacia herramientas que actuen. `tool_contracts`
(`core ADR 0007`) es decision valida pero diferida "hasta que exista una
herramienta concreta que lo justifique" (`core ADR 0022`). Esta es la primera.

Lo decidido en meta para este caso (H09P7-H09P10), que acota lo que se pide:

- Solo un dominio ve herramientas: `domotica`, por su perfil. El confirmador
  manda alli lo que es de la casa; los demas dominios no cambian (H09P7).
- El contrato entre el core y la capa es MCP: el core cliente, la capa
  servidor. Una integracion nueva debe ser configuracion del core, sin codigo
  (invariante de meta ADR 0007) (H09P8).
- Politica por perfil en la configuracion del core, denegar por defecto,
  provisional hasta conscience. `child` (y la voz, que se atiende como `child`):
  consultar estado, fecha y hora, luces y multimedia. `adult`: lo mismo mas
  clima y ventilador (H09P9).
- Nada destructivo para ningun perfil en esta entrega; la confirmacion humana
  se aplaza hasta que un consumidor la necesite (H09P10).

## Que se pide

1. Que un perfil de dominio pueda declarar herramientas de proveedores MCP
   declarados en la configuracion del core, y que el modelo de ese dominio las
   reciba y pueda llamarlas dentro del turno (`prompt.*`, el modo `directo`).
2. Que el core aplique antes de invocar: proveedor declarado, herramienta en la
   lista del perfil del interlocutor (`interlocutor_profile`), y ninguna marcada
   como destructiva. Lo que no pasa no se invoca, y el modelo recibe un rechazo
   que puede explicar.
3. Que cada llamada, invocada o rechazada, conste en la telemetria con
   identidad, perfil, proveedor y herramienta (criterio 7 de H09).
4. Que el streaming hacia el cliente siga igual: el cliente ve solo el texto
   final, no las llamadas.

## Forma sugerida (no vinculante)

    tool_providers:
      home_assistant:
        transport: mcp_streamable_http
        url: <de la configuracion del despliegue>
        trust_annotations: true      # solo para proveedores declarados

    profiles:
      domotica:
        tools:
          provider: home_assistant
          allow:
            child: [<herramientas de consulta, luces y multimedia>]
            adult: [<lo de child mas clima y ventilador>]
          deny_destructive: true     # endurece; nunca relaja lo que marca la capa

La capa marca cada herramienta con las anotaciones de MCP (`readOnlyHint`,
`destructiveHint`) segun el tipo de entidad. El core puede endurecer esas
marcas, nunca relajarlas.

## Impacto

- Core: capacidad nueva y seccion nueva de configuracion -> MINOR (`v0.5.0`)
  segun `meta docs/POLITICA_SEMVER.md`; tocar el core requiere aprobacion del
  usuario. La capa origen necesitaria `core >=0.5 <0.6`.
- Extended: transporta el turno como hoy; si el core emite eventos nuevos,
  extended los reenvia por defecto (meta ADR 0007). A confirmar por el core.
- Borde: ninguno. Sigue rechazando `tools` del cliente (edge ADR 0002): las
  herramientas son del ente, no del consumidor.

## Alternativas consideradas

- Contrato propio por HTTP entre core y capa: reinventa MCP y obliga a cada
  capa a implementar algo a medida (descartado en H09P8).
- El core directo contra el servidor MCP de Home Assistant, sin capa: mete
  logica de Home Assistant en el core. Ese servidor marca casi todo como
  destructivo, incluida la lectura de estado, y usa la misma herramienta para
  una luz que para una cerradura (H09P9, descartado en H08P3).
- Herramientas en todos los dominios: todos sus modelos tendrian que saber
  llamarlas y cada turno llevaria mas contexto (descartado en H09P7).

## Tension doctrinal

- `core ADR 0022`: se cumple su condicion para planificar `tool_contracts`.
- `core ADR 0033`: la capa no dirige el pensar; el core decide si invoca.
- `core ADR 0031`: RAG, memoria y datos web siguen siendo enriquecimiento, no
  herramientas.

Notificacion a la capa destino: ikseth/ia_nest_core#51 (2026-11-06).

## Resolucion

ACEPTADO, REFORMULADO, ENTREGADO y PUBLICADO el 2026-11-06 en `core v0.5.0`
(MINOR por envergadura; `core ADR 0053`, core#54 y core#55). Reformulado en dos
puntos: las herramientas se declaran en el DOMINIO (`domains[].tools`), no en un
perfil de parametros del modelo, que el core comparte entre dominios; y sobra
`trust_annotations`: declarar un proveedor en `tool_providers` es confiar en sus
anotaciones, y la politica del core solo endurece. El bucle vive en
`prompt.run` y `prompt.stream`, con limite de vueltas y una ultima llamada sin
herramientas para que siempre haya respuesta; el cliente solo ve texto
(`done.trace.tool_calls`, aditivo); telemetria `tools_offered` y `tool_call`.
Sin eventos D2 nuevos (aviso a extended en ia_nest_extended#101). Comprobado en
el laboratorio con un proveedor MCP de prueba (criterio del ADR, puntos 1-5 y 7;
el 6 por tests) y desplegado sin paso a mano (meta `H09`, paso H09P12). La capa
origen necesita `core >=0.5 <0.6`.
