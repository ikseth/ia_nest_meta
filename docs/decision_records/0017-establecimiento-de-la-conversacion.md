# Decision 0017: establecimiento de la conversacion, contrato entre capas

Fecha: 2026-11-19
Estado: reconciliado por el usuario (2026-11-19), decision a decision (D1-D3)

Hito `H12` de la hoja de ruta (meta ADR 0014). Practica habitual contrastada en
`docs/plan/pasos/H12P2_practica_habitual.md`.

## Contexto

Hoy una conversacion con el ente empieza con el primer turno, sin participantes
declarados. El borde afirma `user_id`, `service` e `interlocutor_profile`
(meta ADR 0011); extended usa `service` como procedencia y solo compone
`## interlocutor` para `child`; el enrutador del core clasifica con el texto y
nada mas. Lo que el ente puede hacer esta escrito a mano por dominio.

La linea base de `H12` (H12P1b, 2026-11-19) lo mide: las preguntas sobre datos
de la casa no llegan a la casa ni por la voz ni por Open WebUI (0/24 en las
dos), y con el dominio fijado el modelo sigue sin saber que hay en la casa
(techo 6/24). Falta un concepto de base con tres piezas -interlocutores como
entidades, catalogo de recursos y establecimiento de la conversacion- decidido
antes de abrir `H12`. Este ADR fija la tercera como contrato entre capas y
deja nombradas las costuras con las otras dos.

## Decision

### 1. El marco de la conversacion

Establecer una conversacion es fijar su **marco**: los hechos que el ente
necesita para saber con quien habla. Tiene cinco partes, cada una con valor
declarado o `unknown`, nunca por omision (extended ADR 0015, punto 3):

| Parte | Que es | Ejemplo |
|---|---|---|
| `account` | la cuenta autenticada por el borde | la cuenta de servicio de Home Assistant |
| `person` | quien habla, si se sabe | una persona del directorio, o `unknown` |
| `device` | desde donde habla: canal o dispositivo, si se sabe | la tableta comun, un satelite, o `unknown` |
| `place` | el lugar que se deduce de `device` o de `person` | la casa y su area, o `unknown` |
| `missing` | lo que el marco no sabe y una respuesta podria necesitar | `person`, `place` |

"En nombre de quien" es la relacion entre `account` y `person`: una cuenta de
servicio que afirma a una persona habla en nombre de ella; sin persona, habla
en nombre de la cuenta.

**Lo que se le puede ofrecer NO es parte del marco que se afirma.** Sale de
cruzar los permisos (punto 3) con el catalogo de recursos (punto 6) y se
calcula en cada turno.

### 2. Quien lo afirma: el borde, con hechos observables

El borde afirma `account`, `person` y `device` hacia dentro, igual que hoy
afirma la identidad (meta ADR 0011). El consumidor aporta hechos que observa
(que usuario tiene sesion, que dispositivo recibio la voz), nunca
conclusiones: ni lugar, ni perfil, ni lo que el ente debe hacer. El borde no
infiere (core ADR 0033): valida contra el directorio lo que afirma y deja
pasar como dato lo que no puede validar, marcado como afirmado por la cuenta.

### 3. Los permisos salen del directorio, nunca del marco

La cuenta y sus grupos, afirmados por el borde, deciden lo que se puede hacer.
El marco y el perfil de una entidad orientan y no conceden: si concedieran,
una frase podria cambiar lo que se permite (criterio 7 de `H12`). Es la
separacion entre perfil y rol que anticipa H09P22.

### 4. Quien lo compone y lo guarda: extended

- **La sesion guarda el marco** con el que se abrio (extended ADR 0015) y sus
  cambios con fecha. Es lo que hace medible el criterio 5 de `H12`.
- **Cada parte identificada apunta a una entidad** con perfil vivo y
  versionado (extended ADR 0004): la persona, el dispositivo, la casa. De ahi
  sale `place` (el area del dispositivo, la casa de la persona).
- **Extended compone el marco siempre**, con el enriquecimiento apagado
  tambien, como el perfil desde `edge CR-0002`; no lo memoriza como hecho del
  usuario (el marco es procedencia, no clave: `extended FORMA_ENRIQUECIMIENTO`,
  decision 1).

### 5. Como lo ve el core: dato, no instruccion

El core recibe el marco compuesto como contexto del turno, en dos sitios: al
enrutar (pista para elegir dominio, no regla) y al responder (contexto del
modelo). No conoce entidades ni sesiones; recibe hechos ya resueltos. Nada del
marco cambia la politica de herramientas, que sigue en el dominio y en el
perfil del directorio (core ADR 0053, D6). La forma la dispone el core en su
CR (meta ADR 0007, regla 3: extension aditiva).

### 6. El catalogo de recursos: costura con nombre, ADR propio (D3)

Que puede hacer el ente (modelos, dominios, proveedores de herramientas,
agentes) es el catalogo, con su principio ya decidido: lo que se confia se
declara, lo que ofrece se descubre (core ADR 0046, core ADR 0053, D6).

Decidido por el usuario el 2026-11-19: este ADR fija solo la costura. Lo
ofrecido en un turno es la interseccion de los permisos del directorio
(punto 3) y el catalogo, y se calcula en cada turno; lo descubierto es dato,
no instruccion. El resto -hogar, que se declara, que se descubre, como se
clasifica, ofrecer en cada peticion solo lo que pide (OWASP LLM06:2025)- va a un
ADR propio en H12P4.

Motivo: el catalogo cambia por motivos distintos que el marco y puede tener otro
hogar (su precedente es del core, y meta no decide el contrato de una capa);
tiene practica propia (`tools/list` y aviso de cambios de MCP, enrutado sobre
descripciones de herramientas, envenenamiento de herramientas); y es lo que mas
pesa en la linea base de H12P1b. Tenerlo dentro dejaria este ADR abierto hasta
disenarlo con el core.

Criterio falsable: con este ADR cerrado se redactan los CR del borde, de
extended y del consumidor de Home Assistant sin conocer la forma interna del
catalogo, y los criterios 5 y 7 de `H12` se miden sin el.

### 7. Si falta algo: preguntar, como estado

Si una respuesta depende de una parte del marco que es `unknown`, el ente la
pregunta en vez de suponerla, y la respuesta del interlocutor completa el marco
de la sesion. Preguntar es un resultado declarado del turno, como
`INPUT_REQUIRED` de A2A o `elicitation` de MCP, no un texto que el modelo
decide escribir o no. Lo que ya esta en el marco no se vuelve a preguntar
(menor esfuerzo colaborativo).

### 8. Forma del saludo: implicito en cada peticion (D1)

Decidido por el usuario el 2026-11-19. Cada peticion trae los hechos del marco
(cuenta, persona y dispositivo, afirmados por el borde). La primera vez que
extended ve una sesion compone el marco y lo guarda en ella; en los turnos
siguientes lo reusa, y si un hecho cambia (otra persona en la tableta comun)
anota el cambio con fecha. No hay verbo de apertura ni vuelta previa.

Es como lo hacen Assist (cada `ConversationInput` trae usuario, dispositivo y
satelite) y los asistentes de voz comerciales, y como agrupa A2A por
`contextId`. El saludo explicito de SIP o MCP (`INVITE`, `initialize`) se
descarta hoy: la piel OpenAI que hablan Open WebUI y Home Assistant no tiene
verbo de apertura, y una vuelta mas antes de la primera palabra cuesta en voz
(criterio 7 de `H08`). Se podra anadir despues, sin romper este, para agentes o
para otro ente (meta ADR 0007, regla 3).

Criterio falsable: el 100 % de las sesiones de Open WebUI y de Home Assistant
registra su marco (criterio 5 de `H12`) sin que ningun consumidor gane un verbo
nuevo, solo cabeceras.

### 9. Donde vive el marco de cada entidad: entities de extended (D2)

Decidido por el usuario el 2026-11-19. Los hechos que orientan y no llegan en
la peticion (en que area esta un dispositivo, de que casa es esa area, donde
esta la casa, en que idioma habla una persona) viven en el registro de
entidades de extended (extended ADR 0004), con perfil vivo, versionado y con
historico. El directorio sigue diciendo quien es cada cuenta y que puede; no
gana atributos de marco. Une las dos cosas una referencia: el uid de la
persona, y el id de dispositivo que manda el consumidor y afirma el borde.

El operador siembra las entidades con un fichero de la configuracion del
laboratorio que aplica el instalador, sin paso a mano (criterio 8 de `H12`).
El minimo de entities que hace falta lo decide H12P3.

Es la practica habitual: Home Assistant guarda area y planta en el registro de
dispositivos, no en los usuarios; Alexa y Google, la sala en el perfil del
dispositivo, aparte de la cuenta que autentica. Descartados: el directorio
(mezcla lo que orienta con lo que concede, sin historico, y la casa o un
satelite sin cuenta no tienen sitio) y un marco fijo por clase en el borde (el
arreglo del borrador de H12P0: la clase no es la entidad).

Criterio falsable: cambiar el area de la tableta comun en el fichero de siembra
cambia el lugar del marco en la siguiente sesion sin tocar el directorio ni el
borde; y ningun dato de entities cambia lo que se ofrece en un turno
(criterio 7 de `H12`).

### 10. Provisional hasta conscience

- Las entidades las siembra el operador por configuracion del laboratorio
  (criterio 8 de `H12`), como respaldo manual que extended ADR 0004 ya preve.
- La voz de la casa se atiende como `child` mientras no se sepa quien habla.
- El perfil del directorio hace de rol de herramientas (H09P22).

El marco en si no es provisional: son hechos declarados, no juicio. El juicio
sobre como tratar a cada interlocutor sera de conscience.

## Motivos

- **Es la practica habitual, no un invento.** Las seis practicas contrastadas
  (SIP, MCP, A2A, Assist, asistentes de voz comerciales y el terreno comun de
  la conversacion) separan quien habla, desde donde y que se puede hacer;
  guardan lo establecido en una sesion; y ninguna toma permisos de lo que el
  interlocutor dice de si mismo. El detalle esta en
  `docs/plan/pasos/H12P2_practica_habitual.md`.
- **Usa lo que ya existe.** La sesion (extended ADR 0015), las entidades
  (extended ADR 0004), la identidad afirmada (meta ADR 0011) y la politica de
  herramientas (core ADR 0053) estaban decididas; faltaba el contrato que las
  une. No se crea ninguna pieza nueva.
- **Cumple el invariante de meta ADR 0007.** El marco es una extension aditiva:
  una capa que no lo entiende lo reenvia, y ningun consumidor gana un verbo.
- **Trata la causa, no el sintoma.** El arreglo por clase de H12P0 hacia que
  "va a llover?" fuera siempre a la casa desde la voz, aunque la pregunta fuera
  de otro sitio; con el marco, el lugar sale de quien habla y donde, y si no se
  sabe se pregunta.

## Consecuencias

CR que salen de este ADR (se redactan en H12P6, con el minimo de entities de
H12P3, el catalogo de H12P4 y la forma por superficie de H12P5):

- **Borde:** afirmar `device` (y `person` cuando la haya) como hechos hacia
  dentro, en las superficies que lo tengan. Para un satelite de voz sin persona,
  la cabecera de persona pasa a opcional en esa clase, con la cuenta de servicio
  como respaldo (H12P5).
- **Extended:** guardar el marco en la sesion con sus cambios; resolver las
  partes contra entities; componer el marco siempre; pasarlo al core; sembrar
  entities desde la configuracion.
- **Core:** recibir el marco como contexto al enrutar y al responder, sin que
  toque la politica de herramientas; preguntar como resultado declarado del
  turno. Tocar el core requiere aprobacion del usuario.
- **Consumidor de Home Assistant** (`ha-addons`, fuera del ente): mandar el
  usuario con sesion y el dispositivo o satelite, que Home Assistant ya entrega
  en `ConversationInput`. Ya decidido en parte en H12P5.

Otras:

- La voz de la casa deja de tener un perfil fijo como marco: su marco dice
  "persona desconocida en un dispositivo compartido". El perfil `child` sigue
  como rol de herramientas hasta conscience (punto 10).
- `docs/CAPAS_FUTURAS.md`, comunicacion entidad-a-entidad: otro ente sera un
  interlocutor con su propio marco, y su tarjeta (como la de A2A) se contrasta
  con lo declarado por el operador. No se edita ahora: el concern sigue sin
  diseno.

## Lo que esta decision NO hace

- No autentica nada nuevo ni cambia quien autentica (meta ADR 0011).
- No construye el reconocimiento de hablante.
- No decide como trata conscience a cada interlocutor.
- No cierra la comunicacion entidad-a-entidad: otro ente sera un interlocutor
  con su propio marco, y este contrato no debe impedirlo.

## Impacto de version

Ninguno en el contrato publico de ninguna capa: es contrato entre capas. Cada
CR que salga de el (H12P6) se versiona en su capa.

## Estado posterior

2026-11-19: el catalogo de recursos (punto 6) queda decidido en meta ADR 0018
(H12P4). Vive en extended como memoria, con la decision de confianza, y lo ofrecido en un turno es permisos
por catalogo, acotado por el efecto de la peticion (lectura u orden), que solo
resta.

2026-11-20: la forma por superficie y por consumidor (punto 8 y consecuencias
para el borde y el consumidor de Home Assistant) queda decidida en
`docs/plan/pasos/H12P5_saludo.md`: cabeceras por clase iguales en las tres
caras, cuenta, persona y dispositivo con su origen, persona opcional por clase
con respaldo infantil, y `language` fuera de la peticion.

2026-11-21: el respaldo infantil (punto 10 y H12P5, D3) pasa a nivel de permiso
`restricted` con tono apto para todos; el tono infantil queda para una persona
validada del grupo infantil del directorio. Ver meta ADR 0019.
