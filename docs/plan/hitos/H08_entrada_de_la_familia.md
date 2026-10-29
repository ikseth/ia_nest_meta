# H08: entrada de la familia

Estado: abierto
Abierto: 2026-10-28

## Objetivo

Las personas de la casa, adultas e infantiles, usan el ente de verdad: por
Open WebUI, por el asistente de voz de Home Assistant y por Kodi, con su cuenta
del directorio y su perfil. Es el ultimo hito de la hoja de ruta.

Decidido antes de abrir (no se reabre):

- Identidad en el directorio; Open WebUI por su login LDAP, sin cuentas locales
  salvo el admin de emergencia; menu por perfil desde los grupos del
  directorio; el borde decide y falla cerrado (`H05`, `H07`).
- Kodi y el agente Linux, conectados (`H07`). Home Assistant entra con estado y
  control de la casa por un diseno aparte, vigilado, que bloquea `H08` (H07P7).
- El borde solo cambia para corregir lo prometido o por decision del usuario
  (criterio 1 de `H07`, matizado en H07P8b).

Hereda de meta ADR 0015: C8 como puerta, y los criterios 2, 3 (criterio duro
del perfil infantil) y 4 de meta ADR 0013 con su requisito de entrada (punto 7:
L5r y L4b de extended). De la hoja de ruta: todo el trabajo de capa vigilado
que bloquea `H08`. De `H07`: sus notas para `H08`. De meta ADR 0013
(consecuencias): la excepcion del laboratorio de meta ADR 0011 caduca cuando
entre la primera persona que no sea el operador, y desde entonces el criterio 2
es invariante de despliegue; `user_id` propios para pruebas y aviso a la
familia antes de cada parada.

## Criterio de salida

Decidido por el usuario el 2026-10-29 (H08P1). Cada modo se exige solo donde
el menu de una clase lo ofrece a la familia: el menu es configuracion del
borde por clase y perfil (edge ADR 0002, punto 5; H05P6; H07P3), y cada
consumidor usa el modo que le sirve (Kodi, solo `directo`).

Puerta (nadie de la familia entra antes):

1. **Requisito de entrada.** La puerta de extended pasa L5r (redefinida como
   "cero fragmentos de conocimiento") y L4b, sobre el tag desplegado y con el
   almacen que usara la familia, no con uno reducido.
2. **Criterio duro, de punta a punta** (criterio 3 de meta ADR 0013). Con una
   cuenta infantil, en cada modo de su menu y por cada consumidor que lleve
   personas (Open WebUI, voz, Kodi), consta `child` en la auditoria del borde
   y, como contexto compuesto, en la telemetria de extended (`edge CR-0001` y
   `CR-0002`). Una bateria fija de peticiones no aptas para un nino sale
   adaptada o rechazada en todas con perfil infantil, y respondida con perfil
   adulto. Una peticion infantil sin perfil es un fallo, no un aviso.
3. **Superficie** (criterio 2 de meta ADR 0013). Desde otra maquina de la red
   de casa solo responden el borde y Open WebUI; ni el core ni extended.
4. **Sesiones** (criterio 4 de meta ADR 0013). Dos conversaciones distintas
   nunca comparten sesion de extended, y una misma conversacion conserva la
   suya entre turnos.
5. **Nada en claro** (C8). Cada dispositivo de la familia abre Open WebUI por
   HTTPS sin aviso; consumidores al borde y cualquiera al directorio, con TLS;
   en claro no responde nada.
6. **Cada modo ofrecido, sano.** Por cada modo en un menu de la familia, una
   bateria de peticiones de una linea no devuelve el protocolo interno ni se
   corta por contexto acumulado. Mientras core#45 siga abierto, `razonado` no
   esta en ningun menu de la familia (configuracion); vuelve por configuracion
   cuando su bateria pase.
7. **Primer token.** En `directo`, el p90 del primer token de una bateria fija
   es de 8 s o menos.
8. **Home Assistant.** La voz contesta a traves del ente, con estado y control,
   por el criterio de su diseno aparte, y se atiende como infantil.
9. **Sesion de Open WebUI** con caducidad declarada en su instalador (H08P4:
   en el laboratorio, sin caducidad).

Salida ("la familia ha entrado"):

10. Cada persona de la casa tiene cuenta en el directorio, ha iniciado sesion
    desde su propio dispositivo y ha conversado; su perfil consta en la
    auditoria.
11. Una semana de uso real con cero peticiones infantiles sin perfil en la
    auditoria; las pruebas, con `user_id` propios; el aviso de paradas a la
    familia, dado.
12. **Ningun paso a mano** que un instalador deberia hacer.

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H08P1 | Criterio de salida: que significa "la familia ha entrado" | este hilo | - | decidido: el de arriba. Matizado por el usuario: cada modo se exige donde un menu lo ofrece, porque el modo es eleccion del consumidor (por configuracion de su clase); `razonado` sale de los menus de la familia por configuracion mientras core#45 siga abierto, en vez de esperar al core |
| H08P2 | Mapa de bloqueos y su orden | este hilo | H08P1 | decidido por el usuario el 2026-10-29: (1) desglose en el laboratorio de los ~24 s hasta el primer token, desde este hilo, porque decide el orden interno de extended; (2) `edge CR-0001` en el core, PATCH pequeno, desde meta con aprobacion del usuario; (3) hito siguiente de extended en hilo propio con handoff desde aqui: paso cero (L5r), `edge CR-0002`, Fase 9 (L4b) y la parte de latencia que sea suya; (4) Home Assistant, diseno en este hilo en paralelo, con su codigo de core detras de `CR-0001`. core#45 sale del camino critico (H08P1). La puerta y la semana de uso, al final, en este hilo |
| H08P5 | Desglose del primer token en `directo` | este hilo, lab | H08P2 | medido el 2026-10-29 (evidencia en `ia_nest_lab`): el modelo caliente da el primer token en menos de 0,5 s; el resto es preparacion. Por piezas: eleccion automatica de dominio por una llamada al modelo, ~3 s; el dominio elegido cambia de modelo y el recurso de modelos no tiene memoria para los dos, +3-5 s; memoria y conocimiento, en serie, ~5 s; modelos descargados tras 5 min sin uso, +~10 s en el primer turno tras una pausa. Suma: 7-17 s en caliente, ~18-24 s en frio. Ninguna pieza es del borde |
| H08P6 | Reparto de la latencia | este hilo, lab, extended, core | H08P5 | decidido por el usuario el 2026-10-29: (1) en el recurso de modelos, modelos residentes (sin descarga por inactividad): hecho, quita la carga en frio; (2) en extended, memoria y conocimiento en paralelo e indice vectorial, por delante del reranker en su orden; (3) al core, como aviso junto a `edge CR-0001`: elegir el dominio por embeddings en vez de por una llamada al modelo. Corregido el mismo dia: se propuso tambien un solo modelo para todos los dominios, y se retira. El enrutado de cada dominio a su modelo es una base del proyecto, con criterios medidos, y la tabla de dominios no se toca por la latencia. Comprobado despues: los dos modelos de conversacion mas usados caben juntos en el recurso, pero no con el de embeddings; el cambio de modelo entre dominios (+3-5 s) es un limite del recurso, no del diseno, y se trata en el recurso |
| H08P7 | Segunda GPU del recurso de modelos | este hilo; anfitrion (Deck #126) | H08P6 | decidido por el usuario el 2026-10-29: entra en la hoja de ruta como trabajo vigilado que bloquea `H08` por el criterio 7. Hasta hoy estaba fuera del plan por ser recurso del anfitrion (inventario `H01P1`, B09; `H03` la cita como capacidad); H08P6 le da consumidor. Forma, la de su valoracion: un contenedor de modelos propio para la segunda GPU, que da el control por perfil de uso, y el actual fijado a la principal. En la segunda, residentes, el modelo de embeddings y el pequeno de los dominios de la familia, despues el reranker; en la principal, el de conversacion, y el tecnico alterna con el. Para el ente es configuracion: el endpoint de cada modelo en el core y el de embeddings en extended; ningun dominio cambia de modelo. Cadena previa, en el anfitrion: disco de sistema (Deck #75), fuente, controlador propietario de la ultima rama con soporte para esa GPU (reinicio del anfitrion, con aviso del corte), contenedor, configuracion |
| H08P3 | Home Assistant al ente (estado y control): practica habitual, y si es hito propio o paso de `H08` | este hilo | H08P1 | decidido por el usuario el 2026-10-29: hito propio, `H09`, antes de `H08`. Mecanismo: el ente actua sobre la casa por su frontera de herramientas (`core docs/FRONTERAS.md`: cada integracion es un `ia_nest_external_<nombre>` por `tool_contracts`, core ADR 0007), con una capa externa que es cliente del servidor MCP que Home Assistant publica (la via oficial para que un agente externo vea el estado y controle, limitada a las entidades expuestas). En dos tramos: (1) integracion fina en Home Assistant que solo manda texto y lleva la raiz, con las frases conocidas resueltas en local por Assist y la voz atendida como infantil (C6 para Home Assistant); (2) primer contrato de herramientas del core, la capa externa y la politica de herramientas por perfil, provisional hasta conscience (estado y control). Descartado: que el ente haga de modelo para Home Assistant (es la practica de sus integraciones, pero el borde rechaza instrucciones y herramientas del cliente, edge ADR 0002 punto 6, y un consumidor no dirige como piensa el ente, core ADR 0033). Por comprobar: si el servidor MCP acepta un usuario no administrador con token propio o solo OAuth |
| H08P4 | Sesion de Open WebUI sin caducidad y demas notas de `H07` | este hilo | H08P1 | decidido por el usuario el 2026-10-29: la sesion sigue sin caducidad en el laboratorio. Motivo: las sesiones de los equipos de casa estan controladas y el riesgo de un menor con un dispositivo de adulto es practicamente nulo en este entorno; la caducidad sigue siendo configurable en el instalador para otro entorno. Ningun dispositivo Apple, ni previsto: los certificados de `H06` valen para la familia. Resto de notas: `write_memory` en `/rest` se queda como esta (solo entran servicios de confianza); la calidad del diagnostico del agente es de `directo` (extended); la regla de Kodi con tele queda en las reglas del hito; Home Assistant, a `H09` |

| H08P8 | Handoff del hito siguiente de extended a su hilo: paso cero (L5r), `edge CR-0002`, Fase 9 (L4b), memoria y conocimiento en paralelo e indice vectorial | este hilo -> extended | H08P2, H08P6 | hecho el 2026-10-29: prompt del hilo de extended en el laboratorio, enlazado desde su repo |
| H08P9 | `edge CR-0001` en el core (PATCH) y aviso de la eleccion de dominio por embeddings | core | H08P2; aprobacion del usuario para tocar el core | en curso: ficha `core docs/fixes/v0.4/0013`, del codificador y revisada (323 tests, conformidad 136/136). Decidido por el usuario el 2026-10-29: la telemetria sube a `schema_version` 2 con la columna nueva al final y rotacion del CSV si su cabecera no coincide (core ADR 0015 lo exige; la primera pasada la metia en medio sin subir version y desalineaba el fichero existente). Aviso: core#48. Pendiente: PR, tag `v0.4.2` y despliegue en el laboratorio (aprobacion) |
| H08P10 | Puerta: criterios 1-9 en el laboratorio | este hilo, lab | H09, H08P7-P9 | pendiente |
| H08P11 | Entrada y semana de uso real: criterios 10-12 | este hilo, lab | H08P10 | pendiente |

## Mapa de bloqueos (estado al abrir)

Ninguno cerrado. Cada uno se conduce en su capa.

| Bloqueo | Repo | Estado al abrir | Depende de |
|---|---|---|---|
| `edge CR-0001`: el core transporta el perfil del interlocutor | core#44 | propuesto, sin empezar | - |
| `edge CR-0002`: extended lo transporta y lo compone siempre | extended#81 | propuesto, sin empezar | `edge CR-0001` |
| L5r: una frase de cortesia no trae conocimiento (paso cero) | extended, Deck #125 | hito siguiente de extended sin abrir | - |
| L4b: el sistema se queda con la correccion (Fase 9 bien hecha) | extended, Deck #125 | hito siguiente de extended sin abrir | - |
| `razonado` entrega su JSON interno y corta por contexto acumulado | core#45 | aviso abierto, sin respuesta | - |
| ~24 s hasta el primer token en `directo` | extended (y core) | medido una vez, sin desglose | - |
| Home Assistant al ente con estado y control: hito `H09` | meta, core, capa externa nueva, Home Assistant | mecanismo decidido (H08P3), sin abrir | tramo 2: tocar el core |
| Segunda GPU en su propio contenedor (H08P7) | anfitrion, Deck #126 | valorada, sin montar | disco de sistema (Deck #75) |

## Notas

- Un nino con la sesion abierta de un adulto conversa como adulto. Riesgo
  aceptado en este entorno (H08P4); en otro entorno, la caducidad del
  instalador de Open WebUI es el mando.
- Si algun dia entra un dispositivo Apple, los certificados de `H06` (20 anos)
  no le valen: rechaza los de servidor de mas de 825 dias.
- Contexto de los modelos (2026-10-29, camino abierto por H08P7, no bloquea): el
  recurso de modelos carga cada modelo con una ventana de 4096 tokens, la suya
  por defecto, porque el core le habla por la API compatible con OpenAI y no la
  fija. Los perfiles `razonamiento` y `combinacion` del core declaran 8192: por
  encima de 4096 el recurso recorta la entrada. Pendiente de comprobar si ya
  ocurre. Con los modelos pequenos en la segunda GPU queda memoria en la
  principal para una ventana mayor del modelo de conversacion (la ventana es
  configuracion del contenedor de cada GPU). La longitud de la salida no
  depende de la memoria: es `max_tokens` del perfil.
- El asistente de voz es un dispositivo compartido: sin reconocer a quien
  habla, se atiende como infantil (criterio 3 de meta ADR 0013).

## Reglas del hito

- Una decision cada vez, con diagrama y explicacion breve; primero la solucion
  corta; criterio falsable antes que mecanismo.
- En meta, papeles y motivos; nombres de maquina, direcciones y evidencia, en
  `ia_nest_lab`.
- En un Kodi con tele, nada por su interfaz sin comprobar antes que no hay
  reproduccion activa.
