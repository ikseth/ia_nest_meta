# H12P4: practica habitual del catalogo de recursos

Estado: activo
Version: 1.0 - 2026-11-19

Contraste pedido por `H12` antes de decidir el catalogo. Una pagina: como
descubren, describen y ofrecen recursos cinco practicas conocidas, y que toma
IA_NEST de cada una y que no. Las fuentes se citan al final; las versiones son
las vigentes al 2026-11-19.

## Lo comun a todas

1. **Separan confiar de conocer.** Alguien declara a quien se habla (el
   operador configura un servidor, una URL, un registro); lo que ese alguien
   ofrece se lee de el y puede cambiar.
2. **No meten todo en el contexto.** Con pocos recursos, un indice compacto;
   con muchos, una busqueda o una preseleccion antes del modelo. Meterlo todo
   empeora la eleccion, no solo el coste.
3. **Lo leido de un proveedor no es de fiar como instruccion.** Las
   anotaciones y descripciones solo valen si el servidor es de confianza, y aun
   asi pueden cambiar despues de aprobarlo.

## Una por una

| Practica | Como lo hace | Que toma IA_NEST | Que no toma, y por que |
|---|---|---|---|
| **MCP** (especificacion 2026-07-28: Tools, Resources) | `tools/list` paginado y cacheable (`ttlMs`), en orden determinista; `listChanged` con aviso `notifications/tools/list_changed`; la lista puede variar segun la autorizacion de la peticion, no segun la conexion. Los recursos son un canal de datos aparte de las herramientas. "Los clientes DEBEN considerar no fiables las anotaciones salvo que vengan de servidores de confianza" | El inventario de un proveedor declarado se lee con el protocolo; las herramientas por `tools/list` (ya hoy, una sesion por turno, core ADR 0053, D7) y los datos como datos. Las anotaciones valen porque el proveedor esta declarado (core ADR 0053, D6) | Las `instructions` del servidor y los `prompts` como guia del modelo: son texto del proveedor dirigido al modelo. El aviso de cambios no hace falta mientras no haya cache entre turnos |
| **Servidor MCP de Home Assistant** (`components/mcp_server`, `components/homeassistant/llm.py`, rama `dev`) | Publica las herramientas de Assist, solo sobre entidades expuestas (la exposicion es el acto del operador en Home Assistant). Su agente pone en el prompt un **indice estatico sin estado** (nombres, alias, dominio y areas de cada entidad expuesta) y deja el estado a `GetLiveContext`, filtrable por nombre, dominio y area. El mismo indice sale como `prompt` MCP y el estado como recurso `homeassistant://assist/context-snapshot`. Tiene modo `read_only` que publica solo lo anotado de lectura. Su otra API (`homeassistant`) busca acciones por palabras y devuelve las 3 mejores | La forma: indice compacto sin estado para saber que hay, lectura filtrada para saber como esta. El indice es dato estructurado (nombre, dominio, area), no prosa. El modo de solo lectura como precedente de ofrecer menos | Su texto de instrucciones (`DYNAMIC_CONTEXT_PROMPT`): es del proveedor y dirige al modelo. Reenviar todo su catalogo: ya descartado (external_home_assistant ADR 0002) |
| **Seleccion por recuperacion** (Anthropic, herramienta de busqueda de herramientas con `defer_loading`; RAG-MCP, Gan y Sun 2025) | Las herramientas diferidas no entran al contexto; el modelo busca (BM25 o expresion regular sobre nombre, descripcion y argumentos, hasta 5 por busqueda) y solo entonces se cargan. RAG-MCP preselecciona antes del modelo con un recuperador: mas de un 50 % menos de tokens y 43 % de acierto al elegir frente al 14 % con todas en el prompt | Que la causa del techo bajo esta medida en la literatura: con muchas opciones el modelo elige peor. La preseleccion antes del modelo, si el indice no cabe | Que el modelo formule la busqueda: los modelos pequenos del laboratorio no reintentan con el nombre (H12P1b). Un recuperador por similitud en el core: ya descartado como andamio (core ADR 0043) mientras un juicio baste |
| **Tarjetas de agente de A2A** (especificacion 1.0, "Agent Discovery") | Cada agente publica su tarjeta (`/.well-known/agent-card.json`) con habilidades (`id`, `name`, `description`, `tags`, `examples`), interfaces y seguridad; se descubren por URL conocida, por registro curado o por configuracion directa. No hay API estandar de registro | La forma de una entrada de agente cuando exista: el operador declara la URL (configuracion directa), la tarjeta se lee y se contrasta | Un registro hoy: el ente no llama a ningun agente; sin consumidor no se construye (core ADR 0035). Fiarse de las habilidades que la tarjeta dice de si misma |
| **OWASP Top 10 para LLM, LLM06:2025** (exceso de autonomia) y **envenenamiento de herramientas** (Invariant Labs, 2025) | LLM06: minimizar extensiones, su funcionalidad y sus permisos; evitar las abiertas; autorizar fuera del modelo; aprobacion humana en lo de alto impacto; registrar. Envenenamiento: instrucciones ocultas en la descripcion de una herramienta; "rug pull", la descripcion cambia despues de aprobarla; remedio propuesto, fijar versiones y comprobar una huella de las definiciones | Ofrecer en cada peticion solo lo que pide (el incidente del 2026-11-19 es exactamente "funcionalidad excesiva"). Lo descubierto entra como dato acotado, nunca como sistema. Registrar con que lista se ofrecio cada turno (ya hoy, `tools_offered`) | Pedir aprobacion de nuevo cuando cambia una descripcion: declarar es confiar (core ADR 0053, D6). La aprobacion humana de lo destructivo sigue aplazada (H09P10) |

## Lo que sale para el ADR

1. **Declarar sigue siendo el acto de confianza**; descubrir es leer con el
   protocolo lo que el declarado ofrece, en cada turno o con cache cuando la
   haya.
2. **Saber que hay y saber como esta son dos cosas**: un indice compacto sin
   estado (como Home Assistant) y una lectura filtrada. Si el indice no cabe,
   preseleccion antes del modelo, no busqueda que tenga que formular el modelo.
3. **Lo descubierto es dato estructurado**: nombres, clases y areas, en un
   bloque acotado; nunca la prosa del proveedor como instruccion.
4. **Menos es mas seguro y mas preciso**: ofrecer solo lo que la peticion
   necesita reduce el riesgo (LLM06) y mejora la eleccion (RAG-MCP).
5. **Agentes, con forma y sin construir**: tarjeta declarada por URL y
   contrastada, cuando el ente llame a un agente.

## Fuentes

- Model Context Protocol, especificacion 2026-07-28: "Tools" (`tools/list`,
  `listChanged`, anotaciones no fiables) y "Resources".
  modelcontextprotocol.io/specification/latest.
- Home Assistant, `homeassistant/components/homeassistant/llm.py`
  (`async_get_exposed_entities` con `include_state=False`, `GetLiveContextTool`,
  `SearchActionsTool`) y `homeassistant/components/mcp_server/server.py`
  (prompt, recurso `context-snapshot`, modo `read_only`), rama `dev`, leidos el
  2026-11-19.
- Anthropic, "Tool search tool", documentacion de la plataforma Claude
  (`defer_loading`, `tool_search_tool_bm25`, `tool_search_tool_regex`).
- T. Gan y Q. Sun, "RAG-MCP: Mitigating Prompt Bloat in LLM Tool Selection via
  Retrieval-Augmented Generation", arXiv:2505.03275, 2025.
- Agent2Agent (A2A) Protocol, especificacion 1.0 y "Agent Discovery".
  a2a-protocol.org/latest.
- OWASP Top 10 for LLM Applications 2025, LLM06: Excessive Agency.
  genai.owasp.org/llmrisk/llm062025-excessive-agency.
- Invariant Labs, "MCP Security Notification: Tool Poisoning Attacks",
  2025-04-01.
