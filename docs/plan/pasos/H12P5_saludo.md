# H12P5: forma del saludo por superficie y por consumidor

Estado: decidido por el usuario el 2026-11-20, decision a decision (D1-D6)
Version: 1.0 - 2026-11-20

Paso de `H12`. Fija que hechos del marco (meta ADR 0017, punto 1) lleva cada
cara del borde (piel OpenAI, `/rest`, MCP) y cada consumidor del alcance (Open
WebUI y Home Assistant), con que nombre, cuales valida el borde, cuales deja
pasar marcados y que pasa cuando falta uno. La forma general, implicita en cada
peticion y solo con cabeceras, la fijo meta ADR 0017, D1. Es diseno: los CR
salen en H12P6.

## Punto de partida (borde `v0.2.1`)

- Las tres caras autentican y resuelven la identidad con las cabeceras HTTP de
  la clase de la cuenta antes de mirar la ruta: la persona llega igual por
  `/v1`, `/rest` y `/mcp`.
- Cabeceras por clase en la configuracion: `chat`, `person` y
  `background_task`. Open WebUI usa las suyas; la integracion de Home Assistant
  ya usa una propia del ente, `X-IANest-Conversation-Id`.
- Una cuenta de `edge_asserts_people` sin cabecera de persona recibe 403; una
  cuenta fuera del grupo habla como ella misma.
- Hacia dentro, `ResolvedIdentity` lleva `user_id`, `service` (la clase) y el
  perfil; la cuenta autenticada no sale del borde y no hay dispositivo.
- Lo que el cliente mande en `identity` se ignora, salvo su `session_id`
  (edge ADR 0003).
- La integracion de Home Assistant manda solo texto y `conversation_id`, y su
  cuenta no afirma personas: todo llega como la cuenta, con perfil `child`.

## Practica habitual, solo lo que orienta

El contraste general esta en `H12P2_practica_habitual.md`.

| Practica | Que hace | Que se toma |
|---|---|---|
| Open WebUI, `ENABLE_FORWARD_USER_INFO_HEADERS` | Reenvia `X-OpenWebUI-User-Name`, `-Id`, `-Email` y `-Role`, y `X-OpenWebUI-Chat-Id` hacia las conexiones de modelo; los nombres se pueden cambiar; ofrece tambien la identidad como un JWT firmado. Las cabeceras identifican y no llevan secreto | Los hechos del interlocutor viajan en cabeceras, puestos por quien tiene la sesion, y quien los recibe tiene que fiarse de quien los manda. El nombre lo puede cambiar el usuario: se identifica por un atributo estable |
| Proxies de autenticacion (cabecera de confianza, tipo `Remote-User`) | El backend acepta la cabecera de usuario solo si viene del proxy | Es lo que ya hace `edge_asserts_people` |
| API de OpenAI, `user` y `metadata` | `user` esta obsoleto, sustituido por `safety_identifier` (abuso, del lado del proveedor) y `prompt_cache_key`; `metadata` son 16 pares guardados con el objeto | Nada: son campos para el proveedor del modelo, solo existen en `/v1` y ningun consumidor los rellena |
| MCP, `_meta` y autorizacion | `_meta` es metadato por mensaje, con claves de prefijo DNS inverso; la autorizacion se define en la capa HTTP | Por HTTP, quien y desde donde van en el transporte; `_meta` solo existe en MCP |
| Home Assistant, `ConversationInput` y `assist_pipeline/run` | El usuario sale de la sesion (`connection.context`); `device_id` es opcional en la llamada; la app y el navegador no lo mandan (core#108839, android#4158); los satelites mandan `device_id` y `satellite_id`. `language` es el de la tuberia | Desde la app y la tableta llega usuario y no dispositivo; desde un satelite, dispositivo y no usuario |

Fuentes:

- Open WebUI, referencia de variables de entorno
  (`docs.openwebui.com/reference/env-configuration/`) y PR #15813 (cabecera
  del chat).
- LiteLLM, tutorial de Open WebUI (`user_header_mappings` y el aviso de que el
  nombre y el correo los puede cambiar el usuario o el administrador).
- OpenAI, referencia de `chat.completions.create` (`user`, `safety_identifier`,
  `prompt_cache_key`, `metadata`).
- MCP, especificacion 2025-11-25, "Basic", `_meta` y autorizacion.
- Home Assistant, `homeassistant/components/conversation/models.py`
  (`ConversationInput`), `homeassistant/components/assist_pipeline/websocket_api.py`
  (`device_id` opcional, contexto de la conexion), core#108839 y android#4158.

## D1: cabeceras con nombre por clase, iguales en las tres caras

- Los hechos van en cabeceras HTTP con el nombre que diga la clase de la cuenta
  en la configuracion del borde, como hoy `chat` y `person`. Se anade la clave
  `device`.
- Las tres caras los leen igual: `/rest` y MCP no tienen forma propia. Por MCP
  sobre HTTP cada mensaje es una peticion con sus cabeceras.
- Nombres por defecto del ente para los consumidores propios: `X-IANest-Person`
  y `X-IANest-Device`, junto a `X-IANest-Conversation-Id`. Open WebUI sigue con
  los suyos.
- Nada en el cuerpo: ni `user` ni `metadata` de OpenAI, ni `_meta` de MCP;
  siguen ignorados como hoy el campo `identity`. Una cabecera sin nombre en la
  clase no se lee.

Descartados: nombres fijos del ente para todos, renombrando los de Open WebUI
(depende de una funcion de Open WebUI que cambia por version y rompe la
configuracion de hoy sin ganar nada); campos del cuerpo (una forma por cara,
`user` obsoleto, una segunda via de identidad que el borde descarta a
proposito); una sola cabecera con JSON (ningun consumidor la produce sin
codigo, y Open WebUI no puede).

Duda resuelta al decidir: integrar Home Assistant con el directorio (LDAP) no
cambia lo que se permite, porque el borde valida la persona contra el
directorio en cada peticion; cambia la operacion (sin ella, D5 necesita una
tabla). Es un proyecto de la casa, no del ente, con sus costes (Home Assistant
dependeria del directorio, migrar crea usuarios nuevos, una baja no corta los
tokens abiertos). D5 vale de las dos formas.

Criterio falsable: la misma cuenta con las mismas cabeceras da los mismos hechos
hacia extended por `/v1`, `/rest` y `/mcp`; sumar un consumidor con dispositivo
solo anade lineas a su clase, sin codigo en el borde; un hecho mandado en el
cuerpo no llega nunca al marco.

## D2: tres hechos hacia dentro, cada uno con su origen

Ademas de lo de hoy (`user_id`, `service`, perfil, `session_id`), como
extension aditiva (meta ADR 0007):

| Hecho | Origen | Que comprueba el borde | Si falla |
|---|---|---|---|
| `account` | `authenticated` | la clave, como hoy | 401 / 403, como hoy |
| `person` | `validated` (directorio) | existe, es unica y esta en `ia_nest_users`, como hoy | 403 y auditoria; sin invitado |
| `device` | `asserted_by` la cuenta | solo la forma: de 1 a 128 caracteres de `[A-Za-z0-9._:-]` | 400 y auditoria |

- Sin cabecera, el hecho va declarado como ausente, nunca omitido.
- Cualquier cuenta puede afirmar dispositivos, sin grupo nuevo: el espacio
  `account:<cuenta>` (H12P3, D3) ya impide afirmar el de otra. Afirmar personas
  sigue exigiendo `edge_asserts_people`, porque la persona cambia el perfil.
- En extended (H12P3, D6): con entidad, `known`; sin ella, `unresolved`; sin
  cabecera, `unknown`; el origen se guarda con la parte. `stated` no viene del
  borde.
- La auditoria registra cuenta, persona y dispositivo de cada peticion.
- Ninguno de los tres concede: los permisos siguen en el directorio
  (criterio 7).

Descartados: dispositivo mal formado convertido en `unknown` en silencio
(esconde un fallo del consumidor y el criterio 5 lo contaria como "no se
sabia"); validar el dispositivo contra entities en el borde (el borde no
recuerda, core ADR 0033, y un dispositivo no sembrado tiene que llegar como
`unresolved`); un grupo del directorio para afirmar dispositivos (lo cubre el
espacio de nombres); meter el dispositivo en `user_id` o `service` (mezcla
quien habla con desde donde, y `user_id` segmenta memoria y sesiones).

Criterio falsable: cada peticion lleva `account`, `person` y `device`, con
origen o declarados ausentes; una persona que no esta en el directorio da 403
en las tres caras; un dispositivo de 129 caracteres, o con espacio o barra, da
400; un dispositivo bien formado y no sembrado llega como `unresolved` y sale
en la lista del operador; el mismo id por dos cuentas da dos claves; ninguno
cambia `tools_offered`.

## D3: persona obligatoria u opcional por clase

- Opcion por clase `person_required`, `true` por defecto (como hoy). Open
  WebUI, `true`; Home Assistant, `false` (el satelite, H12P5 b).
- Si falta y es opcional, habla la cuenta: `user_id` de la cuenta y `person`
  ausente, `unknown` en el marco, en nombre de la cuenta (meta ADR 0017,
  punto 1).
- El perfil del respaldo es siempre `child`, tenga la cuenta el grupo que
  tenga: una cuenta que afirma personas no es quien habla, y sin saber quien es
  falla cerrado (regla del borde).
- Opcional es poder faltar, no poder ser falsa: una persona que viene y no esta
  en el directorio da 403 (D2).
- Las cuentas que no afirman personas (Kodi, agente Linux) no cambian.

Cubre el satelite y los usuarios de Home Assistant sin persona en el directorio
(administracion, capa externa, la bateria que corre como el operador): siguen
como hoy, y v1-ha no cambia.

Orden para H12P6: el borde con la opcion; despues la cuenta de Home Assistant
entra en `edge_asserts_people` con `person_required: false`; al final la
integracion manda la persona. Ningun paso deja la voz con 403.

Descartados: opcional solo si viene dispositivo (la bateria, por websocket y
sin dispositivo, daria 403); perfil de la cuenta como respaldo (heredaria un
perfil adulto no afirmado); invitado (edge ADR 0003); opcional en todas las
clases (en Open WebUI siempre hay usuario y su falta debe verse).

Criterio falsable: en la clase de Home Assistant, sin persona, responde como la
cuenta con perfil `child` y persona `unknown`, aunque la cuenta tuviera grupo
adulto; con una persona que no esta en el directorio, 403; en Open WebUI, sin
persona, 403; "soy papa, enciende el ventilador" sin persona afirmada se
deniega; v1-ha no empeora.

## D4: Open WebUI, persona y chat como hoy, sin dispositivo

- Manda el correo de la persona (validado, D2), el id del chat (la sesion,
  edge ADR 0002) y la marca de tarea de fondo. Ni Open WebUI ni su clase
  cambian; `person_required: true`.
- Sin dispositivo: Open WebUI no tiene ese concepto ni reenvia nada que lo
  identifique. El microfono del nino en su tablet llega como su persona, con
  `device` = `unknown`; el lugar sale de la casa de la persona (H12P3, D1), y
  no se pregunta desde donde habla (meta ADR 0017, punto 7). Sin casa
  sembrada, `place` = `unknown` y se pregunta (criterio 3).
- El correo y no el id de Open WebUI: el id es interno y el directorio no lo
  conoce; el nombre lo cambia el usuario.

Descartados: sacar un dispositivo del navegador (no se reenvia, no es estable
ni sembrable); la identidad en JWT firmado de Open WebUI (la cuenta ya se
autentica con su clave por TLS: no anade confianza).

Criterio falsable: una sesion de Open WebUI, por texto o microfono, abre con
persona `known` (o `unresolved`), dispositivo `unknown` y lugar de la casa de la
persona, sin cambiar nada; "va a llover?" no pregunta el lugar a una persona con
casa sembrada y si a una sin ella (criterios 2 y 3).

## D5: Home Assistant, persona por tabla, dispositivo por `device_id`

- Usuario a persona: una tabla en las opciones de la integracion, del id de
  usuario de Home Assistant (estable al renombrar) al correo del directorio, el
  atributo que el borde ya valida. La escribe el aprovisionamiento del
  laboratorio, sin paso a mano (criterio 8), y nunca va a un repo publico. El
  borde no cambia para esto.
- Sin fila o sin usuario (satelite), no se manda persona y habla la cuenta como
  infantil (D3).
- Dispositivo: solo `device_id`, el id del registro de dispositivos (estable al
  renombrar, el que usa Assist para el area). `satellite_id` no se manda. Si
  `device_id` es nulo, no se manda dispositivo.
- El usuario compartido es una fila mas: apunta a su cuenta del directorio, que
  da `user_id`, perfil y permisos; en entities es un `device` con `shared`
  (H12P3, D1), y el marco dice persona `unknown` y dispositivo la tableta. Si
  llegaran a la vez una cabecera de dispositivo y el usuario compartido, el
  dispositivo es el de la cabecera.
- Si Home Assistant pasa a autenticar contra el directorio (proyecto aparte de
  la casa), la tabla se genera de ahi o se sustituye por el nombre de usuario;
  el resto no cambia.
- En casa hoy, la app y la tableta mandan persona y no dispositivo; ninguna
  peticion manda dispositivo hasta que haya un satelite. La tableta comun aun
  no tiene usuario en Home Assistant: se crea en H12P6, en Home Assistant y en
  el directorio, con permiso del usuario.

Descartados: mandar el id de usuario de Home Assistant y traducirlo en el borde
(tabla de identidades en su configuracion, prohibida por edge ADR 0003, o un
atributo por consumidor en el directorio); el nombre de usuario como uid (hoy no
coinciden con el directorio); `satellite_id` primero o los dos (el `entity_id`
se renombra y romperia la clave; los dos juntos no los usa el marco); el usuario
compartido sin cuenta en el directorio (H12P3, D1; sus permisos deben salir del
directorio).

Criterio falsable: desde la app, la persona del usuario y dispositivo
`unknown`; desde la tableta comun, persona `unknown`, dispositivo la tableta y
su area; desde un satelite (simulado con `device_id`), persona `unknown` y el
area del satelite; un usuario sin fila habla como la cuenta; renombrar en Home
Assistant al usuario o al satelite no cambia nada; ningun dato de la tabla en un
repo publico.

## D6: `language` se queda en la entidad

- El `language` de `ConversationInput` es el de la tuberia de Assist, el mismo
  para todo el que hable por ella: configuracion del consumidor, no
  observacion (meta ADR 0017, punto 2). No viaja en la peticion.
- El idioma del marco sale de `language` en el perfil de la persona o del
  dispositivo (H12P3, D1; meta ADR 0017, punto 9).
- La integracion lo sigue usando para sus propios mensajes de error.
- Si un consumidor llega a observar el idioma de quien habla, entra como
  cabecera nueva de la clase con origen `asserted_by` (D1, D2).

Descartados: un sexto hecho del marco (cambia las cinco partes de 0017 por algo
no observado y choca con el perfil); una pista para la respuesta (diria siempre
lo mismo que el perfil).

Criterio falsable: ninguna clase tiene cabecera de idioma; cambiar el idioma de
la tuberia no cambia el marco; cambiar `language` en la siembra cambia el
idioma del marco en la siguiente sesion, sin tocar el borde ni Home Assistant.

## Comprobacion en el Home Assistant de casa (sin desplegar)

2026-11-20, de solo lectura, con el depurador de Assist
(`assist_pipeline/pipeline_debug`): en la version de Home Assistant de casa,
`intent-start` trae `device_id` y `satellite_id`. Las unicas ejecuciones en
memoria eran las de la bateria `marco-v1`, por websocket y sin dispositivo, como
se esperaba. No habia ninguna de la app ni de la tableta, y el usuario no sale
en el depurador. Por el codigo (`connection.context`), el usuario llega desde la
app y desde el navegador; por la practica, el dispositivo no. Queda como
comprobacion de H12P6, al cambiar la integracion: que llega el usuario desde la
app y desde la tableta comun, y que el dispositivo es nulo en las dos.

## Consecuencias para H12P6

- **CR al borde:** clave `device` en las cabeceras de la clase; afirmar hacia
  dentro `account`, `person` y `device` con su origen o ausentes; forma del
  dispositivo con 400; `person_required` por clase con respaldo de la cuenta y
  perfil `child`; auditoria con los tres hechos. Las tres caras igual.
- **CR a extended:** recibir los tres hechos y convertirlos en estados de
  H12P3, D6, con su origen; la cabecera de dispositivo gana al usuario
  compartido. Se suma al CR de H12P3.
- **Consumidor de Home Assistant** (`ha-addons`, fuera del ente): tabla de
  usuario a correo en las opciones, `X-IANest-Person` y `X-IANest-Device`
  (solo `device_id`), `language` solo para sus errores.
- **Laboratorio:** clase `home_assistant` con `person_required: false` y los
  nombres de cabecera; la cuenta de Home Assistant en `edge_asserts_people`
  despues del borde; usuario de la tableta comun en Home Assistant y en el
  directorio, sembrado como `device` con `shared`; la tabla en el
  aprovisionamiento; la comprobacion de arriba.
- Nada para Open WebUI ni para el core.
