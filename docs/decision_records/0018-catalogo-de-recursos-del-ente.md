# Decision 0018: catalogo de recursos del ente

Fecha: 2026-11-19
Estado: reconciliado por el usuario (2026-11-19), decision a decision (D1-D5),
con la confianza matizada el mismo dia (punto 2)

Hito `H12` de la hoja de ruta (meta ADR 0014), paso H12P4. Practica habitual
contrastada en `docs/plan/pasos/H12P4_practica_habitual_catalogo.md`.
Completa la costura que dejo con nombre meta ADR 0017, punto 6.

## Contexto

Lo que el ente puede hacer esta repartido y escrito a mano. El core declara
modelos, dominios con su `description`, proveedores (`tool_providers`) y la
politica por perfil, y lee las herramientas del proveedor en cada turno
(core ADR 0053). El enrutador clasifica con el texto y las descripciones de los
dominios (core ADR 0043). Nadie sabe que hay en la casa.

La linea base de `H12` (H12P1b, 2026-11-19) lo mide. Las preguntas sobre datos
de la casa no llegan a `domotica` ni por la voz ni por Open WebUI (0/24 en las
dos): "va a llover?" acaba en `general` y "las placas" en `energia`. Con el
dominio fijado, el techo es 6/24. Sin filtro, la lectura devuelve "demasiados
datos", y el modelo no reintenta con el nombre, dice no tener herramientas para
la energia o se inventa el nombre de la entidad. Pesa mas el catalogo que el
enrutado. El mismo dia, una pregunta de lectura por la voz puso una pelicula:
el turno ofrecia una accion que la pregunta no pedia (OWASP LLM06:2025).

Es tambien la pregunta 1, sin dueno, de `docs/CAPAS_FUTURAS.md` ("que capacidad
usar"), en su parte de recursos.

## Decision

### 1. Principio (decidido antes de abrir `H12`)

- Lo que se confia se declara; lo que ofrece se descubre. Un proveedor no
  declarado nunca se consulta. Matizado por el usuario en este paso: declarar
  separa el techo (a que se puede conectar) de la decision de confianza, que
  es juicio (punto 2).
- Lo descubierto es dato, no instruccion (punto 7).
- Los permisos salen del directorio por el borde, nunca del marco ni del
  catalogo (meta ADR 0017, punto 3).
- Lo ofrecido en un turno es la interseccion de los permisos y el catalogo,
  calculada en cada turno (meta ADR 0017, punto 6), acotada ademas por lo que
  pide la peticion (punto 6 de este ADR).

### 2. Hogar: extended, como memoria (D1)

Decidido por el usuario el 2026-11-19. Un catalogo se guarda y se recuerda: es
memoria y vive en `ia_nest_extended`. Extended lo guarda y da la interfaz para
darlo de alta, de baja y modificarlo; no decide nada. Quien usa esa interfaz es
hoy el operador (el usuario, o el agente que trabaja con el) y mas adelante
conscience: extended da los mandos, y quien conduce es primero el operador y
despues conscience (punto 8). Es la autoridad de escritura que extended
ADR 0004 da a las entidades: crear y resolver es juicio de conscience, con el
operador como respaldo. Es tambien el
patron de meta ADR 0017, D2, que ya puso en entities la casa y sus areas.

**La confianza: se guarda en extended, la decide el operador (despues
conscience) y la aplica el core, dentro de un techo.**
Decidido por el usuario el 2026-11-19, al revisar este ADR. Hoy escribir un
proveedor en `tool_providers` es a la vez poder conectarse y confiar en el
(core ADR 0053, D6). Se separa en tres:

- **Techo (operacion).** A que proveedores se puede conectar el core y con que
  secreto. Lo pone el instalador desde la configuracion del laboratorio, como
  `model.pull`: es operacion, no capacidad del ente (core ADR 0046, D5).
- **Confianza (juicio).** En que proveedores y recursos se confia, dentro del
  techo. Se guarda en el catalogo de extended y se cambia por su interfaz. La
  decide hoy el operador y despues conscience, que es de quien es el juicio
  (core ADR 0033 y 0034).
- **Aplicar (mecanismo).** El core recibe lo confiado, se conecta, lista,
  interseca con los permisos y el efecto, y nunca ofrece lo destructivo.

Nada de la memoria puede ampliar el techo: si el catalogo marca como confiado
un proveedor que no esta en el techo, el core no lo consulta. La memoria solo
resta.

Criterio falsable: retirar la confianza de un proveedor por la interfaz de
extended hace que deje de ofrecerse en el siguiente turno sin tocar el core;
marcar como confiado un proveedor que no esta en el techo no hace que el core
lo consulte nunca.

Lo que no cambia:

- **Los permisos.** Quien puede que sigue saliendo del directorio, y la
  politica por perfil (`allow`, `deny`) sigue en el core, como hoy.
- **Descubrir es automatico.** Lo manual es la curacion (alias, clase, "esto no
  se ofrece"), no el alta de cada aparato (criterio 6 de `H12`).
- **El estado se lee en vivo.** El catalogo sabe que hay; como esta se pregunta
  al proveedor en el turno. La memoria no es fuente del estado.
- **El core no conoce entidades** (meta ADR 0017, punto 5): recibe lo que le
  toca del catalogo ya compuesto, como dato.

Como entra en el almacen de extended (dentro de entities o junto a el) lo fija
H12P3 con el minimo de entities.

**Quien descubre: extended, directamente.** Decidido por el usuario. Extended
lee del proveedor declarado lo que ofrece, para refrescar el catalogo; el core
sigue leyendo sus herramientas en cada turno para ejecutarlo. Con dos
condiciones:

- **Una sola declaracion del proveedor (el techo)**, en la configuracion del
  laboratorio, que leen los instaladores del core y de extended. Asi no
  pueden divergir y nadie da de alta nada a mano (criterio 8 de `H12`).
- **Extended entra con un token de solo lectura.** El proveedor le deja leer el
  inventario, no llamar a herramientas (minimo privilegio, extended ADR 0010).

Motivo del hogar: lo que el ente sabe de si mismo y de su entorno, y en que
confia, es memoria curada y juicio, no configuracion del motor. El motor
ejecuta lo decidido dentro del techo que pone la operacion.

### 3. Que se declara y que se descubre (D2)

| Recurso | Se declara (operador) | Se descubre (dato) | Curacion |
|---|---|---|---|
| Modelos y dominios | configuracion del core | extended los lee del core (`model.list`, `domain.list`) | - |
| Proveedores | techo: una declaracion en la configuracion del laboratorio; politica por perfil en el core | - | confianza (punto 2) |
| Herramientas | - | del proveedor: nombre, esquema y anotaciones (`tools/list`) | - |
| Aparatos de la casa | en Home Assistant: exponer a Assist es el acto del operador | del proveedor: nombre, alias, tipo y clase, area; sin estado ni prosa | alias, clase, "no se ofrece" |
| Areas | - | del proveedor (D2) | - |
| La casa y lo que no esta en el proveedor | fichero de siembra de entities (meta ADR 0017, D2) | - | - |
| Agentes | la URL, cuando exista uno | su tarjeta, contrastada | - |

**Areas: del proveedor; la siembra solo las cita** (decidido por el usuario).
Las areas se descubren de Home Assistant, donde el operador ya las mantiene. La
siembra de entities dice en que area esta un dispositivo citando un area
descubierta. El marco y el inventario hablan del mismo sitio por construccion,
no por coincidencia de nombres.

Las plantas no salen hoy por la API de Assist, que solo da nombre, tipo y
areas. Si hacen falta, sera por otra via del proveedor; no se fuerza ahora.

Agentes: el ente no llama hoy a ningun agente, y sin consumidor no se construye
(core ADR 0035). Cuando lo haga, la forma sera la de A2A: el operador declara
la URL y la tarjeta se lee y se contrasta. Lo que la tarjeta dice de si misma
no concede nada.

Criterio falsable: renombrar un area en Home Assistant la cambia en el catalogo
en el siguiente refresco, sin tocar la siembra. Una siembra que cita un area
que el proveedor no tiene hace fallar la instalacion: falla a la vista, no en
silencio.

### 4. Como llega la casa al modelo: resumen y preseleccion (D3)

Decidido por el usuario el 2026-11-19. Extended compone para cada turno de un
dominio con proveedor:

- **un resumen fijo y corto** de la casa: que clases hay en cada area, sin
  nombres ni estado;
- **una preseleccion**: las pocas entradas del catalogo que tocan a la
  peticion, con su nombre exacto y su area. Las recupera con la recuperacion
  que ya tiene (extended ADR 0004: espacio semantico y el area como gate). El
  area del marco puntua primero, como hace Assist con el area del satelite, sin
  excluir otras (criterio 2 de `H12`).

El modelo lee el estado con la herramienta de lectura, filtrando por el nombre
real que ha recibido.

Descartados: el indice entero de lo expuesto (lo que hace Home Assistant), que
con 115 entidades ronda la ventana de 4096 tokens del modelo de `domotica` y
crece con la casa; y una busqueda que haga el propio modelo, que tendria que
formular y reintentar, justo lo que los modelos pequenos del laboratorio no
hacen (H12P1b).

Criterio falsable: en el techo de `marco-v1` (dominio fijado) no queda ningun
fallo del grupo A por nombre inventado, por "no tengo herramientas" ni por
"demasiados datos" sin reintento. Lo que falle sera por otra causa, y queda
escrita. El bloque de la casa cabe en la ventana del modelo del dominio con
sitio para la respuesta, contado en tokens en cada turno.

### 5. Que ve el enrutador: lo mismo que el modelo (D4)

Decidido por el usuario el 2026-11-19. Extended compone una sola vez el marco
(meta ADR 0017), el resumen y la preseleccion: recuerda antes de clasificar,
como ya preve `docs/EL_CAMINO.md`. Lo manda al core en dos sitios: como pista al
enrutar y como datos al responder. Sin nada de la casa que toque a la
peticion, la pista es solo el resumen. Es la misma costura que 0017 ya pide al
core (contexto al enrutar), sin forma nueva.

La pista no es regla. La `description` de cada dominio la sigue escribiendo el
operador (core ADR 0043), y no se amplia a mano para estas frases (sin parche,
`H12`).

Descartados: pegar el resumen a la descripcion de `domotica` (generico; con la
casa grande arrastraria frases de conocimiento) y que extended fije el dominio
cuando la preseleccion supere un umbral (una regla con umbral, lo que core
ADR 0043 descarto como andamio).

Criterio falsable: en `marco-v1`, las frases del grupo A llegan a `domotica`
por la voz y por Open WebUI. Se comprueba con `domain.route`, sin ejecutar
nada. El grupo D (conocimiento con palabras parecidas) sigue en 24/24 y el E
(Kodi) en 18/18. Si D cae, la pista se revisa; no se compensa con texto en las
descripciones.

### 6. Ofrecer solo lo que pide: lectura u orden (D5)

Decidido por el usuario el 2026-11-19. Toda clasificacion de un turno con
herramientas devuelve tambien su **efecto**, lectura u orden, como resultado
declarado (como preguntar lo que falta en meta ADR 0017, punto 7).

- Si es lectura, el core ofrece solo las herramientas anotadas como de lectura
  (`readOnlyHint`). Si es orden, lo permitido y no destructivo, como hoy.
- En la duda, lectura.
- En un dominio fijo con herramientas, donde `prompt.*` no enruta
  (core ADR 0043), se pide igualmente esa clasificacion, con una llamada corta
  al modelo del enrutador. Asi vale tambien si Home Assistant pasa a dominio
  fijo (H12P5).
- **El efecto solo resta.** El core interseca lo permitido con el efecto y
  nunca anade: un error de clasificacion no da mas permiso del que habia
  (criterio 7 de `H12`). `tools_offered` registra el resultado, como hoy.

Coste declarado: una llamada mas en los turnos de dominio fijo con
herramientas, que se mide con v1-ha.

Aplazado: acotar ademas por la clase de aparato que toca la preseleccion. Por
si sola no resuelve el incidente (el Kodi del salon toca multimedia y saldrian
sus scripts); entra si la medida lo pide. La confirmacion de lo destructivo
sigue en H09P10.

Criterio falsable: la frase del incidente ("que se esta viendo ahora en el Kodi
del salon?"), repetida por la voz y por Open WebUI, no tiene en `tools_offered`
ninguna herramienta que no sea de lectura en ninguna vuelta, y las preguntas
del grupo A de `marco-v1` tampoco. Las ordenes siguen llegando: para medirlo
hay que recrear en Home Assistant la luz y el ventilador de prueba de la
bateria v2, con permiso del usuario. v1-ha no empeora en tiempo frente a su
ultima fila.

### 7. Lo descubierto es dato

- El resumen, la preseleccion y la pista del enrutador se componen de campos
  estructurados (nombre, alias, tipo y clase, area), con longitud acotada, en
  un bloque de datos delimitado. Nunca van como instrucciones del sistema.
- Ningun texto libre del proveedor dirigido al modelo entra en el catalogo: ni
  `instructions` ni `prompts` de MCP, ni el texto de instrucciones del agente
  de Home Assistant.
- Las descripciones de herramientas llegan al modelo como parte de la
  herramienta ofrecida, como hoy, porque el proveedor esta en el techo y es de
  confianza (punto 2). No entran en la pista del enrutador.

### 8. Lo que es del sistema y lo que es provisional hasta conscience

Del sistema, y no cambia cuando llegue conscience:

- Extended guarda el catalogo y da la interfaz para darlo de alta, de baja y
  modificarlo, incluida la confianza. Es mecanismo: no decide nada.
- El core aplica lo confiado dentro del techo; la operacion pone el techo; los
  permisos salen del directorio.

Provisional hasta conscience (como meta ADR 0017, punto 10):

- **Quien usa la interfaz**: el operador, es decir, el usuario o el agente que
  trabaja con el.
- **La siembra**: un fichero de la configuracion del laboratorio que el
  instalador aplica por esa misma interfaz, para que no haya paso a mano
  (criterio 8 de `H12`).
- **El juicio de cada cambio**: alta, baja, alias, clase, "esto no se ofrece"
  y confianza.

Cuando conscience exista, usara la misma interfaz que el operador: ni extended
ni el core cambian por ello.

Criterio falsable: un sustituto de prueba de conscience da de alta un recurso,
lo da de baja y retira la confianza de un proveedor por la misma interfaz que
usa el operador, sin cambiar extended ni el core.

## Motivos

- **Es la practica habitual.** Home Assistant separa saber que hay (indice sin
  estado) de saber como esta (lectura filtrada); la seleccion por recuperacion
  (RAG-MCP, busqueda de herramientas) mide que ofrecer menos elige mejor; OWASP
  LLM06 pide ofrecer el minimo y autorizar fuera del modelo; MCP y el
  envenenamiento de herramientas piden tratar lo leido como no fiable. El
  detalle esta en `docs/plan/pasos/H12P4_practica_habitual_catalogo.md`.
- **Usa lo que ya existe.** La memoria y la recuperacion de extended
  (extended ADR 0003 y 0004), el marco y su costura con el core (meta ADR 0017),
  el cliente MCP y la politica del core (core ADR 0053), y el catalogo cerrado
  de la capa de Home Assistant (external_home_assistant ADR 0002). Ninguna capa
  nueva.
- **Trata la causa.** El techo bajo era que el modelo no sabe que hay en la
  casa, y el enrutado fallido, que el enrutador tampoco. Una sola composicion
  arregla los dos sin escribir frases a mano en las descripciones.
- **El juicio a quien juzga, el mecanismo a quien ejecuta.** La confianza va
  con la memoria, camino de conscience; el core la aplica dentro de un techo
  que pone la operacion. Permisos del directorio y politica del core quedan
  donde estaban, y el catalogo solo puede restar de lo ofrecido.

## Consecuencias

CR que salen de este ADR (se redactan en H12P6, con el minimo de entities de
H12P3 y la forma por superficie de H12P5):

- **Extended:** guardar el catalogo con el patron de entities (perfil
  versionado, con historico), incluida la decision de confianza; una interfaz
  para darlo de alta, de baja y modificarlo, que usan el operador, el
  instalador con la siembra y despues conscience; refrescarlo
  del proveedor declarado como cliente de solo lectura; leer modelos y
  dominios del core; componer el resumen y la
  preseleccion, con el area del marco como preferencia; mandar al core lo
  confiado y la composicion, al enrutar y al responder.
- **Core:** su configuracion de proveedores pasa a ser techo y recibe de
  extended lo confiado, sin poder salir del techo (reformula core ADR 0053,
  D6, que lo dispone el core); recibir la composicion como contexto al enrutar
  y al responder (la misma costura que meta ADR 0017); devolver el efecto en
  la clasificacion; clasificar tambien en los turnos de dominio fijo con
  herramientas; intersecar la oferta con el efecto. Tocar el core requiere
  aprobacion del usuario.
- **Capa de Home Assistant** (`ia_nest_external_home_assistant`): publicar su
  inventario (nombre, alias, tipo y clase, area; sin estado) y aceptar un
  segundo token que solo lo lea. Anadir a la capa es MINOR en su contrato
  (external_home_assistant ADR 0002).
- **Laboratorio:** una sola declaracion del proveedor (el techo), que leen
  los dos instaladores; fichero de siembra; recrear la luz y el ventilador de
  prueba para medir ordenes (permiso dado por el usuario el 2026-11-19).

Otras:

- `docs/CAPAS_FUTURAS.md`, pregunta 1: queda respondida en su parte de
  recursos. Elegir entre atomico y descomponible sigue en el confirmador
  (`docs/EL_CAMINO.md`, salida "complejo", hito `H10`).
- Meta ADR 0017 recibe un `## Estado posterior` con el puntero a esta decision.
- Se mide con `marco-v1` y v1-ha, y cada pasada suma su fila al historico
  (H08P24).

## Lo que esta decision NO hace

- No mueve fuera del core la declaracion de modelos y dominios ni el techo de
  proveedores (a que se conecta y con que secreto).
- No guarda el estado de la casa en la memoria.
- No construye la confirmacion de lo destructivo (H09P10).
- No construye un registro de agentes ni anade plantas.
- No cambia quien autentica ni de donde salen los permisos.

## Impacto de version

Ninguno en el contrato publico de ninguna capa: es contrato entre capas. Cada
CR que salga de el (H12P6) se versiona en su capa.
