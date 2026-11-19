# H12P2: practica habitual al establecer una conversacion

Estado: activo
Version: 1.0 - 2026-11-19

Contraste pedido por `H12` antes de redactar meta ADR 0017. Una pagina: como
establecen sesion y contexto seis practicas conocidas, y que toma IA_NEST de
cada una y que no. Las fuentes se citan al final; las versiones son las vigentes
al 2026-11-19.

## Lo comun a todas

Todas separan tres cosas que en IA_NEST hoy van mezcladas en `service` y
`interlocutor_profile`:

1. **Quien habla** (persona, cuenta o agente), que puede ser desconocido.
2. **Desde donde** (dispositivo, area, canal), que suele conocerse aunque no se
   sepa quien habla.
3. **Que se puede hacer** (capacidades o permisos), que nunca se toma de lo que
   el interlocutor dice de si mismo, sino de quien lo autentico.

Y todas guardan lo establecido en un estado de sesion que dura mas que un turno,
con un modo de renegociar o de pedir lo que falta.

## Una por una

| Practica | Como establece | Que toma IA_NEST | Que no toma, y por que |
|---|---|---|---|
| **SIP** (RFC 3261, oferta y respuesta de RFC 3264) | `INVITE` con una oferta de lo que cada extremo puede; la respuesta fija la interseccion; un nuevo `INVITE` dentro del dialogo renegocia si algo cambia | La sesion como estado establecido y nombrado, distinto de cada mensaje; renegociar cuando cambia un participante o el lugar | La vuelta previa antes de la primera palabra: en voz cuesta latencia (criterio 7 de H08). Y la negociacion de capacidades entre iguales: el consumidor no negocia lo que el ente le ofrece, lo decide el directorio |
| **MCP** (especificacion 2025-06-18) | `initialize` por conexion: version, `capabilities` y `clientInfo`/`serverInfo`; despues `tools/list` y aviso `notifications/tools/list_changed` si el servidor lo anuncio; `elicitation` deja al servidor pedir al usuario un dato que falta | El inventario de un proveedor declarado se descubre y se mantiene vivo por aviso de cambios (catalogo, H12P4). `elicitation` es la forma estandar de "preguntar lo que falta" | El campo `instructions` del servidor y los textos de herramientas como guia del modelo: lo descubierto es dato, no instruccion (envenenamiento de herramientas). `clientInfo` es autodeclarado: no identifica a nadie |
| **A2A** (especificacion 1.0) | Tarjeta de agente publicada (`/.well-known/agent-card.json`: habilidades, capacidades, esquemas de seguridad); `contextId` agrupa tareas y mensajes en una misma sesion; la tarea pasa a `INPUT_REQUIRED` o `AUTH_REQUIRED` si le falta algo y se continua con el mismo `contextId` | `contextId` es nuestra sesion (extended ADR 0015). "Falta un dato" como estado explicito, no como texto inventado. La tarjeta es el modelo para cuando el interlocutor sea otro ente (comunicacion entidad-a-entidad) | Un ciclo de tarea por cada turno de charla: pesado para la voz. Y confiar en las habilidades que se declara a si misma una tarjeta: se contrasta con lo declarado por el operador |
| **Assist de Home Assistant** (`ConversationInput`, `homeassistant/components/conversation/models.py`) | Cada peticion lleva `text`, `context` (con el usuario de Home Assistant), `conversation_id`, `device_id`, `satellite_id` y `language`. Assist deduce el area y la planta del dispositivo y las usa como preferencia al buscar entidades ("enciende la luz" es la del area) | El marco viaja implicito en cada peticion, desde quien lo conoce, sin vuelta previa; dispositivo, area y planta como hechos del marco; la exposicion de entidades a Assist como acto declarado | Home Assistant como cerebro (descartado en H09). Y `extra_system_prompt`: texto libre del consumidor hacia el modelo es instruccion, no dato |
| **Asistentes de voz comerciales** (Alexa: `context.System.device`, `user` y `person`; Google: Voice Match) | Tres identificadores distintos: la cuenta, el dispositivo (con su direccion o sala) y la persona, que solo aparece si se reconoce la voz. Sin persona reconocida, el dispositivo responde en modo generico y sin datos personales | Cuenta, persona y dispositivo como tres campos; "persona desconocida en un dispositivo compartido" como estado valido del marco, no como perfil `child` fijo. Los permisos salen de la cuenta, no de lo que se dice | El reconocimiento de hablante, hoy: es una mejora futura ya anotada en H08 |
| **Terreno comun en la conversacion** (Clark y Brennan 1991; Clark 1996) | Los participantes acumulan lo que saben que el otro sabe y buscan evidencia de que se han entendido; si falta algo, reparan (preguntan) en vez de seguir; principio del menor esfuerzo colaborativo: no preguntar lo que ya se sabe | El criterio de fondo del hito: fijar el terreno comun al abrir, preguntar solo cuando la respuesta depende de lo que falta, y no volver a preguntar lo ya establecido en la sesion | Nada en contra; es teoria, no mecanismo |

Y una guia de seguridad que cruza todas: **OWASP Top 10 para LLM, LLM06:2025
(exceso de autonomia)**: ofrecer al modelo solo las funciones y permisos que la
tarea necesita, y aprobacion humana para lo de alto impacto. Es el fondo del
incidente del 2026-11-19 (una pregunta de lectura puso una pelicula): el turno
tenia ofrecida una accion que la pregunta no pedia. Toca al catalogo (H12P4) y a
la confirmacion aplazada en H09P10, no al saludo.

## Lo que sale para el ADR

1. **El marco se afirma, no se negocia.** Como en Alexa y en Assist, el
   consumidor aporta hechos observables (que dispositivo, que usuario) y el
   borde los afirma; ni el consumidor ni el modelo deciden lo que se ofrece.
2. **El marco viaja con la peticion y se fija en la sesion.** Como en Assist
   (implicito) y en A2A (`contextId`); el saludo explicito de SIP o MCP queda
   como mejora para consumidores que lo puedan dar. Decision D1 del ADR.
3. **Quien, donde y que falta son campos separados**, y "desconocido" es un
   valor declarado.
4. **Lo que falta se pregunta como estado**, como `INPUT_REQUIRED` de A2A o
   `elicitation` de MCP, no como texto que el modelo decide inventar o no.
5. **Lo descubierto es dato.** Ni `instructions` de MCP, ni la tarjeta de un
   agente, ni `extra_system_prompt` dirigen al modelo.
6. **Permisos de quien autentica.** Ninguna de las seis practicas toma permisos
   del perfil o de lo dicho en la conversacion.

## Fuentes

- J. Rosenberg et al., RFC 3261, "SIP: Session Initiation Protocol", 2002;
  J. Rosenberg y H. Schulzrinne, RFC 3264, "An Offer/Answer Model with SDP", 2002.
- Model Context Protocol, especificacion 2025-06-18: "Lifecycle" (`initialize`,
  negociacion de capacidades, `listChanged`), "Tools" y "Elicitation".
  modelcontextprotocol.io/specification/2025-06-18.
- Agent2Agent (A2A) Protocol, especificacion 1.0.0: Agent Card, `contextId`,
  `TaskState`. a2a-protocol.org/latest/specification.
- Home Assistant, `homeassistant/components/conversation/models.py`
  (`ConversationInput`) y `homeassistant/helpers/llm.py` (preferencia de area y
  planta del dispositivo al buscar entidades), rama `dev`, leidos el 2026-11-19.
- Amazon Alexa Skills Kit, "Request and Response JSON Reference"
  (`context.System.device`, `user`, `person`); Google Assistant, Voice Match.
- H. H. Clark y S. E. Brennan, "Grounding in communication", en Perspectives on
  Socially Shared Cognition, APA, 1991; H. H. Clark, Using Language, Cambridge
  University Press, 1996.
- OWASP Top 10 for LLM Applications 2025, LLM06: Excessive Agency.
