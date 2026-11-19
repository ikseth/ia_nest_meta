# H12P3: practica habitual para interlocutores como entidades

Estado: activo
Version: 1.0 - 2026-11-19

Contraste pedido por `H12` antes de fijar el minimo de `entities` de extended
(meta ADR 0017, D2) y como entra en el catalogo (meta ADR 0018). Solo lo que
orienta ese minimo; el contraste general del establecimiento esta en
`H12P2_practica_habitual.md` y el del catalogo en
`H12P4_practica_habitual_catalogo.md`. Las fuentes se citan al final; las
versiones son las vigentes al 2026-11-19.

## Lo comun a todas

1. **Pocas clases, un mismo patron.** Persona, dispositivo, lugar (casa y sala)
   y lo que hay en el lugar se guardan con la misma forma: id propio, nombre,
   alias y un perfil pequeno. Lo que cambia entre clases son los campos, no el
   mecanismo.
2. **El enlace con fuera es una clave con espacio de nombres.** Ninguna practica
   usa el id de un sistema externo como id propio: guarda pares
   `(origen, id externo)`, unicos dentro de su origen, y resuelve por ellos.
3. **El lugar es una referencia, no un texto.** El dispositivo apunta a su sala
   y la sala a su casa; la sala la mantiene quien mantiene la casa.
4. **Cambiar un hecho no borra el anterior.** Los almacenes de memoria de
   agentes guardan el historico de cada cambio (que habia, que hay, quien y
   cuando) o invalidan con fecha en vez de sobrescribir.
5. **Los permisos no estan en el perfil.** La cuenta autentica y autoriza; el
   perfil de persona o de dispositivo solo orienta.

## Una por una

| Practica | Como lo hace | Que toma IA_NEST | Que no toma, y por que |
|---|---|---|---|
| **Registros de Home Assistant** (dispositivos, entidades, areas, plantas) | Un registro por clase con la misma forma. El dispositivo tiene id propio y `identifiers` como pares `(dominio, id)`, unicos dentro de su integracion; `area_id` apunta a un area, y el area a su planta. Un hijo sin area hereda la del padre. `created_at` y `modified_at`, sin historico | Id propio mas claves externas con espacio de nombres; el area como referencia a lo que mantiene el operador en Home Assistant (meta ADR 0018, D2) | Un registro por clase: en IA_NEST las clases comparten mecanismo, version e interfaz. Y la falta de historico |
| **Asistentes de voz comerciales** (Alexa: `deviceId`, `userId`, `personId`; Google: estructura, sala y dispositivo en Home Graph, `roomHint` y `structureHint`) | Tres ids distintos para cuenta, dispositivo y persona; la persona solo si se reconoce la voz. El dispositivo vive en una sala de una estructura (la casa); la sala la fija el usuario, el proveedor solo la sugiere | Persona, dispositivo y casa como clases; la casa como entidad con sus salas; el dispositivo sin persona como caso normal | El reconocimiento de hablante (mejora futura, H08) |
| **Memoria de agentes: Letta** (bloques `human` y `persona`) | El perfil del interlocutor es un bloque de texto acotado que el propio agente reescribe con herramientas | Un perfil corto y acotado que entra en el contexto | Que el modelo reescriba el perfil durante la conversacion: en IA_NEST lo escribe el operador y despues conscience (extended ADR 0002) |
| **Memoria de agentes: Mem0** (`history`, ambitos `user_id`, `agent_id`, `run_id`) | Cada recuerdo guarda su historico: evento `ADD`, `UPDATE` o `DELETE`, valor anterior y nuevo, fecha y la entrada que lo causo | Un historico de solo anadir por entidad, con valor anterior, quien y por que | Que una llamada a un modelo decida el cambio: aqui decide el operador |
| **Memoria de agentes: Zep y Graphiti** (grafo temporal) | Bitemporal: cuando fue verdad un hecho (`valid_at`, `invalid_at`) y cuando lo supo el sistema (`created_at`, `expired_at`); lo contradicho se invalida, no se borra | No borrar nunca: un hecho sustituido sigue direccionable con su fecha | El tiempo de validez extraido del texto: los hechos sembrados son declarados, y basta con cuando se escribieron |
| **Configuracion declarativa** (Kubernetes `kubectl apply` con `--prune` y etiquetas de propiedad; Terraform `plan` y `apply`) | El fichero dice el estado deseado; aplicarlo dos veces no cambia nada; lo que desaparece del fichero solo se retira si lleva la marca de que lo creo ese fichero; se puede ver el plan antes de aplicar | Siembra declarativa e idempotente, con propiedad: retirar solo lo que sembro el fichero, nunca lo creado por otro escritor | Un servidor de estado aparte: el estado es la propia tabla de extended |

## Lo que sale para el minimo

1. **Un registro, pocas clases**: persona, dispositivo y casa para el marco;
   las areas y lo descubierto, con el mismo patron (meta ADR 0018).
2. **Claves externas con espacio de nombres**, unicas dentro de su origen.
3. **El lugar se cita**: dispositivo a area, area a casa.
4. **Historico de solo anadir**, con autor y motivo; sin tiempo de validez.
5. **Siembra declarativa**, idempotente y con propiedad.
6. **Nada del perfil concede**.

## Fuentes

- Home Assistant, documentacion para desarrolladores: "Device registry"
  (`identifiers`, `connections`, `area_id`, herencia de area),
  contributors.home-assistant.io/core/integrations/device-registry, leida el
  2026-11-19; y `ConversationInput` (`context`, `device_id`, `satellite_id`),
  ya citado en H12P2.
- Amazon Alexa Skills Kit, "Request and Response JSON Reference"
  (`context.System.device.deviceId`, `user.userId`, `person.personId`).
- Google Home Developers, Home Graph `devices.sync` (`roomHint`,
  `structureHint`) y Smart Device Management API (estructuras, salas y
  `parentRelations`).
- Letta, documentacion de bloques de memoria (`label`, `value`, `limit`,
  `read_only`; bloques `human` y `persona`).
- Mem0, API "Memory History" (`event` `ADD`/`UPDATE`/`DELETE`, `old_memory`,
  `new_memory`), docs.mem0.ai/api-reference/memory/history-memory.
- P. Rasmussen et al., "Zep: A Temporal Knowledge Graph Architecture for Agent
  Memory", arXiv:2501.13956, 2025.
- Kubernetes, "Declarative Management of Kubernetes Objects Using Configuration
  Files" (`kubectl apply`, `--prune` con selector); HashiCorp Terraform,
  `plan` y `apply`.
