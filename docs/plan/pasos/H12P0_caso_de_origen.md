# H12P0: caso de origen (borrador de edge CR-0004, no enviado)

Estado: historia. Borrador redactado el 2026-11-19 en el hilo de `ha_minions04`
como `edge CR-0004`. Revisado el mismo dia en meta con el usuario: el problema no
es de una capa sino de base (quien habla, que recursos hay, como se establece
la conversacion), asi que no se envia como CR y abre el hito `H12`. Se conserva
tal cual por su caso y su evidencia; los CR por capa saldran del diseno de
`H12`. La revision esta en el fichero del hito. El numero `edge CR-0004` no
se reutiliza, para que este texto no se confunda con un CR posterior.

---

## CR-0004: marco del interlocutor, enrutado con contexto y completar lo que falta

- id: CR-0004
- fecha: 2026-11-19
- capa origen: `ia_nest_edge` (en nombre de sus consumidores; caso motor: la voz de Home Assistant, Luna)
- capa destino: `ia_nest_extended` (compone y entrega el marco); necesita un CR derivado al
  `ia_nest_core` (enrutado con contexto y paso de completar lo que falta), como `edge CR-0003`
- estado: propuesto

## Caso de uso motor

Desde H09 (cerrado el 2026-11-09) la voz de la casa, Luna, pasa al ente: Assist resuelve en local lo que
entiende y el resto llega por el borde con la cuenta de Home Assistant y perfil `child`. El ente ve y controla
la casa por `ia_nest_external_home_assistant`, pero solo si el enrutador elige el dominio `domotica`.

Pruebas del 2026-11-19 por la API de conversacion de Home Assistant (produccion, solo lectura), con la
entidad del tiempo y los sensores de energia expuestos a Assist:

| Frase | Dominio | Respuesta |
|---|---|---|
| "Se esta cargando la bateria?" | `domotica` | correcta: 30 %, no carga (30 s) |
| "Va a llover hoy?" | otro | "no tengo acceso a informacion meteorologica" |
| "Hace frio fuera?" | otro | igual |
| "Que tiempo hace hoy y va a llover?" | otro | inventa el clima de Quito |
| "Cuanta energia estan produciendo las placas ahora?" | otro | inventa: la sonda Juno en Jupiter |

La misma pregunta significa cosas distintas segun quien la haga. "Va a llover?" dicho por el asistente de voz
de una casa es una pregunta sobre esa casa, y su ubicacion se conoce. Escrito por el operador en el chat puede
ser sobre otro sitio, y entonces lo normal es averiguar donde antes de contestar. Una persona sabe con quien
habla; si no lo sabe, lo pregunta. Hoy el ente no hace ninguna de las dos cosas: decide solo por el texto y
contesta aunque le falte un dato, lo que lleva a inventar.

## Lo que hay hoy (leido en el codigo el 2026-11-19)

- **Borde** (`v0.2.1`): afirma hacia dentro `user_id`, `service` (clase del consumidor) e
  `interlocutor_profile`. La voz de Home Assistant es un dispositivo compartido: `child`, persona desconocida.
- **Extended** (`v0.6.5`): transporta la identidad. Usa `service` como procedencia (memoria, telemetria). Al
  modelo solo le compone `## interlocutor` con perfil `child` ("El interlocutor es un nino o una nina",
  `edge CR-0002`). Para enrutar, `_route_prompt` manda a `domain.route` la frase y, como mucho, el ultimo
  intercambio.
- **Core** (`v0.5.0`): `DomainRuntime.route` recibe la identidad, pero `DomainRouter.route(prompt)` clasifica
  solo con el texto frente al catalogo de dominios (`id` y `description`). La identidad no llega al
  clasificador.
- No hay un paso que detecte que falta un dato necesario (lugar, persona) y lo obtenga o lo pregunte.
- No hay presentacion de sesion: el consumidor no declara que es, donde esta ni que puede hacer.

## Que se pide

1. **Marco del interlocutor, declarado y no inferido.** Cada clase de consumidor (y, cuando la haya, cada
   persona) tiene un marco: que es el canal, donde esta, en nombre de quien habla, que dominios o
   herramientas le son propios y que datos le faltan (por ejemplo, la voz de la casa no sabe quien habla).
   Lo declara la configuracion, no un modelo.
   Para un consumidor que es una casa, el marco incluye su plano: areas y plantas. Hoy `get_live_context` da el
   area de cada entidad pero no su planta, y "cuanto han gastado los enchufes de la primera planta?" no se puede
   resolver sin ese dato. Mientras tanto, en Home Assistant la planta va en el nombre de las entidades de consumo.
2. **El enrutador lo recibe como orientacion, no como regla.** Junto a la frase, el clasificador recibe el
   marco. Con el de la voz de la casa, "va a llover?" se inclina a `domotica`; una pregunta de cultura general
   sigue yendo a la charla.
3. **Completar lo que falta antes de responder.** Si la respuesta depende de un dato que no esta, el ente lo
   busca (marco, memoria, una herramienta) y, si no lo encuentra, lo pregunta en vez de suponerlo.
4. **Presentacion de sesion.** Al empezar una conversacion, el consumidor puede presentarse (marco de la
   sesion) y el ente lo conserva durante la sesion. Lo que siga sin saberse (quien habla, en la voz) puede
   preguntarse. El reconocimiento de hablante es una mejora futura ya anotada en H08.

## Forma sugerida (no vinculante)

- **Borde:** sin inferencia nueva. Declara en `classes` un marco estatico por clase (o una referencia a el) y
  lo afirma hacia dentro, igual que el perfil. Opcional: una cabecera de presentacion de sesion por clase.
- **Extended:** compone el marco como seccion propia del contexto (ampliacion de `## interlocutor`), lo
  compone siempre (no depende del enriquecimiento, como el perfil en `edge CR-0002`), no lo memoriza como
  hecho del usuario y lo pasa tambien a `domain.route`.
- **Core (CR derivado):** `domain.route` acepta un contexto del interlocutor y el clasificador lo usa como
  pista; paso de "datos que faltan" antes de responder, con salida "preguntar". Declarar en telemetria si el
  marco se uso al enrutar.
- **Consumidor de Home Assistant** (`ia_nest_conversation`, repo `ha-addons`, fuera del ente): si se adopta
  la presentacion de sesion, la integracion la envia al abrir cada conversacion.

## Impacto SemVer previsto

- Extended: aditivo (MINOR), por la nueva entrada a `domain.route` y la seccion de contexto.
- Core: aditivo (MINOR), por el argumento opcional de `domain.route` y el paso de completar. Lo dispone el
  core en su CR.
- Borde: PATCH o MINOR segun si la presentacion de sesion entra como cabecera nueva.

## Alternativas consideradas

- **Ampliar la descripcion de `domotica`** (tiempo, energia, consumos) en la configuracion del core: arregla
  las frases de hoy, pero trata el sintoma; desde el chat, "va a llover?" iria a la casa aunque se pregunte
  por otro sitio. Sirve como parche temporal mientras se decide este CR.
- **Que el consumidor reescriba la frase** ("en casa, va a llover?"): mete inferencia en el consumidor y no
  sirve para el resto de canales.
- **Fijar el dominio por clase** (la voz siempre a `domotica`): rompe la charla por voz.
- **Que el ente haga de modelo para Home Assistant** (con la API LLM de HA): descartado en H09.

## Relacion con el perfil infantil

La voz de la casa entra hoy siempre como `child` (H09P1, H08P19), con politica provisional hasta conscience (H09P9,
H09P22). Es la misma pregunta de este CR, quien habla: el marco deberia decir "persona desconocida en un dispositivo
compartido" en vez de fijar un perfil, y el perfil deberia salir de identificar al hablante (o de preguntarlo) cuando
exista esa capacidad. El propietario entendia que `child` era temporal, de pruebas; conviene revisar si sigue
vigente en produccion y hasta cuando.

## Tension doctrinal

- El borde no infiere (`core ADR 0033`): el marco es declarado, no deducido.
- "Lo arma la capa" (`core ADR 0031`): la composicion es de extended.
- El juicio sobre como tratar al interlocutor sigue destinado a conscience; este CR solo hace que el ente
  sepa con quien habla y que le falta.
- La memoria no se parte por canal (`docs/FORMA_ENRIQUECIMIENTO.md`, decision 1): el marco es procedencia,
  no clave.

## Origen

Hilo de `ha_minions04` del 2026-11-19 (Deck #195 y #199). Estado de Luna verificado en
`ha_minions04/docs/09_capa_de_uso/LUNA_ESTADO_ACTUAL_2026-11-19.md`.
