# H09: Home Assistant al ente

Estado: abierto
Abierto: 2026-11-06

## Objetivo

El asistente de voz de Home Assistant pasa por el ente, y el ente ve el estado
de la casa y la controla. `H08` espera a que este hito cierre entero (criterio 8
de `H08`).

Decidido antes de abrir (no se reabre):

- Mecanismo (H08P3): el ente actua sobre la casa por su frontera de
  herramientas. Una capa externa nueva, `ia_nest_external_home_assistant`
  (convencion de `core docs/FRONTERAS.md`), es cliente del servidor MCP que
  Home Assistant publica: estado con `GetLiveContext` y control por
  intenciones, limitado a las entidades expuestas. El core decide y la invoca
  por `tool_contracts` (core ADR 0007: denegar por defecto, permisos
  explicitos, confirmacion humana en lo destructivo). Descartado: que el ente
  haga de modelo para Home Assistant.
- Dos tramos (H08P3). Tramo 1: integracion fina en Home Assistant, solo texto,
  con la raiz del laboratorio y sin herramientas; Assist resuelve en local las
  frases conocidas y pasa el resto al ente; la voz es un dispositivo compartido
  y se atiende como infantil; cierra C6 para Home Assistant (meta ADR 0015); no
  toca el core. Tramo 2: primer contrato de herramientas del core (MINOR, con
  aprobacion del usuario), extended lo transporta, la capa externa y la
  politica de herramientas por perfil, provisional hasta conscience; espera a
  core#47 (H08P12).
- El borde no cambia: la integracion fina solo manda texto (edge ADR 0002,
  punto 6, y sus consecuencias).
- Modo (H08P17): la voz entra por el confirmador con enrutador (`directo` con
  `domain: auto` y la salida de charla).
- Perfil (H08P19): se comprueba que `child` llega de punta a punta. Si la
  respuesta es apta para un nino no se mide: es de conscience; riesgo asumido.
- Los tiempos de espera hacia la eleccion de dominio del core se quitan con la
  segunda GPU (H08P21, Deck #126). Si aparecen en la voz, son ese fallo, no de
  este hito.

Home Assistant es el de produccion de la casa: nada que cambie su
comportamiento sin decirselo antes al usuario.

## Que mueve del camino

De `docs/EL_CAMINO.md`, H09 abre el confirmador con enrutador a un consumidor
nuevo, la voz (tramo 1), y anade al core su primera frontera de herramientas
(tramo 2), que hoy no aparece en los mapas: el ente pasa de contestar a actuar.
El orquestador no se mueve.

## Criterio de salida

Decidido por el usuario el 2026-11-06 (H09P1). Cada punto dice que criterio de
`H08` o de meta ADR 0015 cubre.

Tramo 1, "la voz pasa por el ente":

1. **Lo local sigue local.** Una bateria fija de ordenes que Assist ya conoce
   no deja ninguna entrada en la auditoria del borde, y la casa responde como
   antes.
2. **Perfil de punta a punta** (criterio 2 de `H08`). Cada frase de una bateria
   fija de voz que Assist no conoce consta en la auditoria del borde con la
   cuenta de Home Assistant y `child`, y como contexto compuesto en la
   telemetria de extended.
3. **Sesiones** (criterio 4 de `H08`). Cada conversacion de Home Assistant es
   una sesion de extended: dos conversaciones no la comparten y un seguimiento
   conserva la suya.
4. **La voz responde** (criterio 7 de `H08`). La bateria de voz contesta todas
   sus frases, con p90 hasta el primer token de 10 s o menos. Una frase que no
   se contesta se oye como aviso legible y consta en la telemetria de extended
   y en la auditoria del borde. Motivo de los 10 s: con el texto en streaming,
   Home Assistant empieza a hablar con el primer trozo, asi que ese tiempo es
   el silencio que oye la persona, y pasado ese rato quien habla a un altavoz
   suele repetir la orden o darla por perdida. Es referencia: con el hardware
   de 2026-11-06 no se cumple (el confirmador da p90 de 26,7 s), y se revisa al
   montar la segunda GPU (Deck #126), con la mejora medida.
5. **Solo configuracion y su integracion** (C6). Home Assistant entra con su
   cuenta del directorio, la integracion fina con la raiz dentro de su propia
   instalacion y su configuracion, sin tocar el borde ni a otros consumidores.

Tramo 2, "el ente ve y controla la casa":

6. **Estado.** En una bateria fija de preguntas de estado sobre entidades
   expuestas, la respuesta coincide con el estado real leido en el mismo
   momento.
7. **Control.** Las ordenes de una bateria fija sobre entidades de prueba
   expuestas y no destructivas cambian su estado, y cada llamada consta en la
   telemetria del core con identidad, perfil y herramienta.
8. **Denegar por defecto** (core ADR 0007). Nada fuera de las entidades
   expuestas ni de la lista de herramientas del perfil se ejecuta; el rechazo
   queda registrado.
9. **Lo destructivo** (cerraduras, alarma, puertas) no se ejecuta sin
   confirmacion humana.
10. **Sustitucion** (criterio 8 de `H08`). El asistente de voz de la casa pasa
    por el ente y deja de usar el recurso de modelos directamente, con aviso al
    usuario antes del cambio.

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H09P1 | Criterio de salida por tramos y limite de tiempo de la voz | este hilo | - | decidido por el usuario el 2026-11-06: el de arriba. El limite de la voz, 10 s de p90 hasta el primer token, queda como referencia y se revisa al montar la segunda GPU con la mejora medida |
| H09P2 | Servidor MCP de Home Assistant: usuario no administrador con token propio | este hilo | - | comprobado el 2026-11-06 en el codigo de Home Assistant (`mcp_server`): las vistas usan la autenticacion normal, asi que vale un token de larga duracion de un usuario sin OAuth. La opcion "requerir administrador" (activa por defecto) solo afecta a la URL que sirve todas las API configuradas; la de la API de Assist (`/api/mcp/assist`, solo entidades expuestas) nunca exige administrador. A confirmar en la version instalada en produccion al abrir el tramo 2 |
| H09P3 | Donde vive cada pieza | este hilo | H09P1 | decidido por el usuario el 2026-11-06. La integracion fina va en `ha-addons`, el repo del usuario para integraciones propias de Home Assistant, con un dominio distinto del `ia_nest_assist` del `ia_nest` original, para que no herede su configuracion si sigue instalado. Es codigo del consumidor, como el adaptador de Kodi en su repo, y no entra en el registro de capas. El cliente del borde (TLS con la raiz, clave, sesion, streaming y errores) va en un modulo sin dependencias de Home Assistant, para sacarlo a su propio paquete cuando un segundo consumidor lo necesite (core ADR 0035). Coste aceptado: HACS instala una sola integracion por repo y `ha-addons` tiene varias, asi que la instalacion la hace un script del laboratorio desde un tag, como el complemento de Kodi (H07P8). La clase, el menu y la cuenta de Home Assistant son configuracion del borde y del directorio, en el laboratorio. Para el tramo 2, segun la doctrina: la capa externa es un repo nuevo sembrado desde meta con su fila en el registro; como depende del core y necesita `tool_contracts`, lo pide por un CR suyo (la capa de abajo propone) y el core decide la forma con su ADR, como MINOR y con aprobacion del usuario (core ADR 0022 lo difiere hasta tener una herramienta concreta); la politica de herramientas por perfil es configuracion del core, provisional hasta conscience. Se concreta al abrir el tramo 2 |
| H09P4 | Tramo 1 montado en el laboratorio | lab, `ha-addons` | H09P3 | en curso, montado el 2026-11-06 (evidencia en `ia_nest_lab`). Integracion `ia_nest_conversation` 0.1.1 en `ha-addons` (rama local, sin publicar), del codificador y revisada en dos rondas: la primera corrigio que la voz no empezaba a hablar en streaming (Assist solo pasa texto al TTS tras un trozo con rol de asistente), una letra acentuada partida entre dos lecturas de red, el formulario de la raiz y un error sin aviso; la segunda, tras probarla en vivo, que el primer turno de cada conversacion abria una sesion suelta en el borde. En el borde, solo configuracion: clase `home_assistant` con menu `casa` (confirmador) y cuenta de servicio con perfil `child` en el directorio. En Home Assistant de produccion, con aviso al usuario: la integracion, instalada por un script del laboratorio desde un commit, y una canalizacion de Assist aparte que no es la preferida; la de la familia no cambia. Comprobado: puntos 1 (una orden local no llega al borde), 2 (`child` en la auditoria del borde) y 3 (una conversacion, una sesion) del criterio, el 1 despues con su bateria fija (4 consultas de solo lectura, todas en local y ninguna en el borde); punto 5 con la integracion como unico codigo. Las frases de Assist en espanol necesitan tildes: escritas sin ellas no casan y van al ente; la voz las trae del reconocimiento. Comprobado despues: `child` tambien en la telemetria de extended como contexto compuesto (punto 2 entero). Bateria v1 sin cambiar por la canalizacion de Home Assistant (H09P6) |
| H09P6 | Bateria v1 por la voz y el limite de 10 s | este hilo, lab | H09P4 | medido el 2026-11-06 (evidencia en `ia_nest_lab`): las 20 frases de la v1 sin cambiar, 3 vueltas, como texto por la canalizacion de prueba y medidas dentro de Home Assistant. Primer token con mediana de 12,3 s y p90 de 15,3 s (el confirmador, el mismo dia por extended: 17,9 y 26,7 s); 18 de 54 por debajo de 10 s. Home Assistant empieza a hablar al pasar de 60 caracteres: mediana de 12,7 s y p90 de 17,5 s; las respuestas mas cortas se dicen al final. El punto 4 no se cumple: ni el p90 ni contestar todo. 6 de 60 fallan, siempre en `matematicas` y `razonamiento`: la respuesta llega entera, y despues extended corta el turno con error porque escribir la memoria del turno (llamada al core) agota 30 s por el cambio de modelo en la GPU de 12 GB. La persona oye la respuesta y despues el aviso de fallo. Las baterias de H08 no lo veian porque no escriben memoria. Decidido por el usuario el mismo dia: aviso a extended (extended#100) para que un fallo al escribir la memoria despues de entregar la respuesta no convierta el turno en error; la causa, el cambio de modelo, sigue en la segunda GPU (H08P21). Descartado: que la voz no escriba memoria, porque los seguimientos perderian el hilo. La bateria se repite cuando se corrija extended#100 y al montar la segunda GPU |
| H09P7 | Tramo 2, D1: donde entran las herramientas en el flujo del ente | este hilo | H09P1 | decidido por el usuario el 2026-11-06: solo el dominio `domotica` las ve, en su perfil; el confirmador manda alli lo que es de la casa y los demas dominios no cambian. Motivo: es la practica habitual (enrutador barato delante y agente especializado con herramientas) y encaja con el enrutado por dominio, base del proyecto. El dominio ya existe y lo sirven dos modelos que saben llamar herramientas. Coste aceptado: reescribir su descripcion (hoy habla de configurar Home Assistant, no de manejar la casa) y comprobar con una bateria que el confirmador enruta bien las ordenes. Descartado: herramientas en todos los dominios (todos sus modelos tendrian que saber llamarlas y cada turno llevaria mas contexto). Quedan, en orden: D2 contrato core-capa, D3 politica por perfil, D4 confirmacion por voz, D5 siembra, CR y ADR del core |
| H09P8 | Tramo 2, D2: contrato entre el core y la capa externa | este hilo | H09P7 | decidido por el usuario el 2026-11-06: MCP. El core es cliente MCP y cada capa externa es un servidor MCP. Motivo: cumple el invariante de meta ADR 0007 (una integracion nueva es configuracion del core, sin codigo), es el estandar de herramientas para agentes, lo habla Home Assistant y el core ya usa su libreria. La capa marca lo destructivo con las anotaciones de MCP (`readOnlyHint`, `destructiveHint`) segun el tipo de entidad, que es logica de Home Assistant: su servidor MCP no lo distingue. El core solo se fia de esas marcas si vienen de capas declaradas en su configuracion, y su politica puede endurecerlas, nunca relajarlas. Descartado: un contrato propio por HTTP (reinventa MCP) |
| H09P9 | Tramo 2, D3: politica de herramientas por perfil | este hilo | H09P8 | decidido por el usuario el 2026-11-06, provisional hasta conscience, en la configuracion del core y con denegar por defecto. `child`: consultar estado, fecha y hora, luces (encender, apagar, brillo, color) y multimedia (pausa, seguir, volumen); nada de clima, ventilador, cerraduras, alarma, puertas ni persianas; scripts y anuncios solo si se anaden uno a uno. `adult`: todo, y lo destructivo con confirmacion (D4). La voz se atiende como `child` (dispositivo compartido), asi que esta es la politica de la voz. Coste aceptado: hoy el asistente de voz, contra el recurso de modelos directamente, puede hacer todo lo expuesto; por el ente, por voz nadie toca clima ni cerraduras hasta que el ente sepa quien habla. Visto al leer las herramientas del servidor MCP de Home Assistant (solo `tools/list`): marca como destructivo casi todo, incluso la lectura de estado, y `HassTurnOn`/`HassTurnOff` sirven igual para una luz que para una cerradura. Confirma D2: la capa clasifica cada llamada por el tipo de entidad a la que va |
| H09P5 | Tramo 2: contrato de herramientas, capa externa y politica por perfil | core, extended, capa nueva | H09P4, core#47 | pendiente |

## Reglas del hito

- Una decision cada vez, con diagrama y explicacion breve; primero la solucion
  corta; criterio falsable antes que mecanismo.
- En meta, papeles y motivos; nombres de maquina, direcciones y evidencia, en
  `ia_nest_lab`.
- En Home Assistant de produccion, nada que cambie lo que oye o hace la familia
  sin avisar antes al usuario; lo nuevo entra al lado de lo actual, no en su
  lugar, hasta que el hito lo sustituya.
