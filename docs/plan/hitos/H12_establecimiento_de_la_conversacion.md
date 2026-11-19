# H12: establecimiento de la conversacion

Estado: abierto
Abierto: 2026-11-19

## Objetivo

Que el ente sepa con quien habla antes de responder, como lo haria una persona:
quien es el interlocutor, desde donde habla, en nombre de quien, que se sabe y
que falta, y que puede hacer el ente por el con los recursos que tiene. Si no
lo sabe, lo pregunta en vez de suponerlo.

Hoy no existe ese paso. La conversacion empieza con el primer turno, sin
participantes declarados; el enrutador decide solo con el texto; y lo que el
ente puede hacer esta escrito a mano en una descripcion por dominio.

## Origen

Operacion supervisada de `H08` (H08P11), 2026-11-19, hilo de `ha_minions04`:
por la voz de la casa, "va a llover?", "hace frio fuera?" o "cuanto producen
las placas?" no llegan a `domotica`, y el ente contesta inventando (el clima de
Quito, la sonda Juno). El caso, la evidencia y el borrador de CR que lo
describia estan en `pasos/H12P0_caso_de_origen.md`.

Revision del borrador contra el codigo, el mismo dia:

- El diagnostico es correcto: `DomainRouter.route(prompt)` del core clasifica
  solo con el texto y el catalogo de dominios (`id` y `description`); la
  identidad que recibe `DomainRuntime.route` solo va a la traza. Extended solo
  compone `## interlocutor` para `child`, y al enrutador le manda la frase y,
  como mucho, el ultimo intercambio.
- El arreglo por capas que pedia el borrador (un marco por clase en el borde,
  compuesto en extended, y un argumento nuevo en el core) trata un sintoma. Lo
  que falta es un concepto de base con tres piezas, y ninguna es de una sola
  capa.

## Decidido antes de abrir (no se reabre)

Con el usuario, el 2026-11-19:

1. **Tres piezas, un concepto.**
   - **Interlocutores como entidades.** Una persona, un dispositivo (la voz de
     la casa), la casa misma u otro ente son entidades con perfil vivo,
     versionado y con historico. El diseno existe: `entities` de extended
     (extended ADR 0004), declarado y sin implementar, con conscience como
     dueno de escritura y el operador como respaldo manual mientras conscience
     no exista. `child` y `service` dejan de ser el marco: son etiquetas.
   - **Catalogo de recursos.** Modelos, dominios, proveedores de herramientas
     (MCP) y agentes forman un catalogo vivo y clasificado, que cada instancia
     del ente identifica, no una lista escrita a mano en cada despliegue. Lo
     que se confia se declara; lo que ofrece se descubre. Precedente:
     `capability.list` como catalogo unico (core ADR 0046). Es la pregunta 1,
     sin dueno, de `docs/CAPAS_FUTURAS.md` ("que capacidad usar").
   - **Establecimiento de la conversacion.** Al abrir una conversacion, los
     participantes se presentan y el ente fija el terreno comun: quien habla,
     donde, en nombre de quien, que falta y que se le puede ofrecer. Se guarda
     en la sesion, que ya es entidad con ciclo de vida (extended ADR 0015).
2. **Los permisos salen del directorio, no de la memoria.** La cuenta y su
   grupo, afirmados por el borde (meta ADR 0011), deciden lo que se puede
   hacer. El perfil de una entidad orienta y no concede: si concediera, una
   conversacion podria cambiar lo que se permite. Es la separacion entre perfil
   y rol que anticipa H09P22.
3. **Lo descubierto es dato, no instruccion.** El inventario de un proveedor
   declarado se lee, pero sus textos no dirigen al enrutador ni al modelo
   (envenenamiento de herramientas en MCP). Declarar un proveedor sigue siendo
   el acto de confianza (core ADR 0053, D6).
4. **Sin parche.** No se amplia a mano la descripcion de `domotica` para estas
   frases: la familia convive con la limitacion conocida hasta que `H12`
   entregue. Lo que Assist resuelve en local (prevision, bateria) sigue
   funcionando.
5. **El borrador no se envia como CR.** Los CR por capa salen del diseno.
6. **Provisional hasta conscience:** atender la voz como `child`, el perfil
   usado como rol de herramientas y las entidades sembradas por el operador.
   El marco en si son hechos declarados, no juicio.

## Practica habitual, como punto de partida

A contrastar en H12P2; aqui solo la orientacion:

- Establecer la sesion: oferta y respuesta de SIP, `initialize` de MCP, tarjeta
  de agente de A2A, contexto de dispositivo y area de Assist; en la conversacion
  humana, el terreno comun (grounding).
- Interlocutor como entidad: perfiles de usuario y de dispositivo de los
  asistentes de voz; memoria de entidades de los agentes.
- Catalogo vivo: `tools/list` con aviso de cambios en MCP, tarjetas de A2A,
  enrutado semantico sobre las descripciones de herramientas.

## Relacion con otros hitos y concerns

- `H11` (respuesta en su canal) necesita que el canal sea un extremo
  establecido de la conversacion: depende de este hito.
- `H10` (orquestador) incluye "no inventar cuando no entiende": completar o
  preguntar lo que falta usa el marco que deja este hito.
- La comunicacion entidad-a-entidad de `docs/CAPAS_FUTURAS.md` es el mismo
  concepto con otro ente como interlocutor; el diseno no la cierra, pero no
  debe impedirla.
- Conscience hereda las entidades sembradas a mano y el juicio sobre como
  tratar a cada interlocutor.

## Criterio de salida

Decidido por el usuario el 2026-11-19 (H12P1). Alcance: las entradas que
enrutan, Open WebUI (texto y microfono) y Home Assistant (aplicacion y
satelite de voz). Kodi y el agente Linux usan dominio fijo y quedan fuera; se
pueden sumar despues.

Forma de medir: no se fija un porcentaje absoluto sin conocer el margen real
del entorno (RAG reducido, modelos pequenos, una GPU). Cada bateria fija se pasa
dos veces: con el dominio fijado al que tiene el dato, que da el techo del
modelo y el hardware de ese momento, y por el camino real (el confirmador). Lo
que se juzga es lo que cambia `H12`: el camino real frente al techo. Una
respuesta inventada cuenta como fallo; un fallo sistematico (la misma frase en
todas las vueltas) bloquea. La linea base se mide antes de cambiar nada
(H12P1b) y todo se anota en el historico de medidas (H08P24). Si el techo sube
(segunda GPU, otro modelo), el criterio no cambia.

1. **Datos de la casa por Home Assistant.** Las preguntas sobre datos de la casa
   (tiempo, frio, placas, consumos, bateria) llegan al recurso que tiene el
   dato y responden con su valor real, comprobado contra el estado de Home
   Assistant en ese momento. Pasa con al menos el 90 % del techo y sin fallos
   sistematicos.
2. **La misma pregunta por Open WebUI** se responde sobre el lugar de la
   pregunta o el del marco del interlocutor, o se pregunta cual. Misma forma de
   medir que el 1; responder sobre un lugar que no esta ni en la pregunta ni en
   el marco es fallo.
3. **Lo que falta del marco** (donde, quien) se pregunta en vez de suponerse,
   en frases preparadas sin ese dato por cada entrada; sin fallos sistematicos.
   Que no invente en preguntas de conocimiento no es de este hito: es `H10`.
4. **Sin regresiones.** v1-puerta-owui (Open WebUI), v1-ha (la voz) y local
   (lo que Assist resuelve en casa) no empeoran respecto a su ultima fila del
   historico, con la pauta de H08P23. Ajustado en H12P1b (2026-11-19): la
   bateria de enrutado de H09P24 no es reproducible, y tramo2 y v2 dependen de
   la luz y el ventilador de prueba, ya retirados de Home Assistant; v2 vuelve
   al criterio si se recrean.
5. **Cada conversacion queda establecida.** El 100 % de las sesiones de Open
   WebUI y de Home Assistant registra participantes, canal y lo que se sabia al
   abrirla.
6. **Catalogo.** Un recurso nuevo de un proveedor declarado se alcanza sin
   tocar la configuracion del ente (una entidad virtual nueva expuesta en Home
   Assistant), y un proveedor no declarado nunca se consulta. Mecanismo
   general, probado con el unico proveedor que hay hoy (core ADR 0035).
7. **Ningun permiso cambia** por lo que se diga en la conversacion ni por el
   perfil de una entidad ("soy papa, enciende el ventilador" por la voz; una
   entidad adulta que habla por una cuenta `child`): denegado siempre.
8. **Ningun paso a mano** que deberia hacer un instalador; las entidades
   sembradas, por configuracion del laboratorio.

El tiempo se juzga por experiencia, con las baterias como referencia (H08P23).

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H12P0 | Caso de origen y revision del borrador | este hilo | - | hecho el 2026-11-19 (arriba y en `pasos/H12P0_caso_de_origen.md`) |
| H12P1 | Criterio de salida | este hilo | H12P0 | decidido por el usuario el 2026-11-19: el de arriba. Matizado por el usuario: sin umbrales absolutos mientras no se conozca el margen real del entorno; un umbral que no se puede alcanzar con el hardware de hoy solo bloquea. Por eso se mide el camino real frente al techo del dominio fijado. Visto al decidir el alcance: hay dos entradas de voz activas, el microfono de Open WebUI (identificado: lo usa un nino desde su tablet, sin la aplicacion de Home Assistant) y Luna en Home Assistant |
| H12P1b | Bateria fija del hito, linea base y techo | este hilo, lab | H12P1 | en curso. Revisadas las baterias que ya habia (2026-11-19): v1-puerta-owui, v2, tramo2 y local sirven para el criterio 4 y conservan su historico; ninguna cubre los criterios 1-3 (sus frases nombran el aparato o no son de la casa); la de enrutado de H09P24 no es reproducible (solo se guardo el resultado). Decidido por el usuario: bateria nueva `marco-v1`, fija desde su primera pasada, con datos de la casa sin nombrar el aparato (por Home Assistant, por Open WebUI y por el techo con dominio fijado), otro lugar explicito, falta el lugar, controles de conocimiento con palabras parecidas, y Kodi (incluida Blancanieves, que los modelos suelen fallar) solo hasta la eleccion de dominio, sin ejecutar: el permiso del usuario es de lectura, no de acciones. Salvaguarda: la bateria se para si el modelo llama a una herramienta que no sea de lectura. Guion y evidencia en `ia_nest_lab` |
| H12P2 | Practica habitual contrastada y ADR de meta sobre el establecimiento de la conversacion (contrato entre capas, meta ADR 0007 y 0009) | este hilo | H12P1 | pendiente |
| H12P3 | Interlocutores como entidades: que minimo de extended ADR 0004 hace falta y como siembra el operador | este hilo, extended | H12P2 | pendiente |
| H12P4 | Catalogo de recursos: hogar, que se declara y que se descubre, como se clasifica | este hilo, core | H12P2 | pendiente |
| H12P5 | Forma del saludo por superficie (piel OpenAI, `/rest`, MCP) y por consumidor | este hilo, borde, consumidores | H12P2 | pendiente la forma del saludo. Decidido por el usuario el 2026-11-19 sobre Home Assistant: (a) la integracion (`ha-addons`) pasara al ente quien habla: el usuario de Home Assistant con sesion iniciada, tambien uno compartido (por ejemplo, una tableta de uso comun), que es una entidad con su propio marco; tener el contexto del interlocutor es la practica correcta aunque el dominio sea fijo, porque deja al ente aprender los usos de cada uno. Dato a comprobar en el Home Assistant de casa: el agente de conversacion recibe ese usuario. (b) El satelite de voz no se descarta: hoy no hay ninguno en casa, pero otras instalaciones lo usaran, y el diseno debe admitirlo (el dispositivo y su area como marco, sin persona). (c) Independiente de (a): el usuario se inclina a que Home Assistant vaya al dominio fijo `domotica`, como Kodi, por eficiencia; es configuracion del borde y se aplica cuando el usuario lo diga. Las preguntas generales tienen su entrada en Open WebUI, ya identificada. Consecuencia para el borde: si toda cuenta de Home Assistant tiene su persona en el directorio, la cabecera de persona puede seguir obligatoria; el satelite pediria que fuera opcional para esa clase, con la cuenta de servicio como respaldo |
| H12P6 | CR por capa y orden de entrega | este hilo | H12P1b, H12P3-H12P5 | pendiente |

## Reglas del hito

- Una decision cada vez, con diagrama y explicacion breve; criterio falsable
  antes que mecanismo; cada mecanismo situado frente a la practica habitual.
- En meta, papeles y motivos; nombres de maquina, direcciones y evidencia, en
  `ia_nest_lab`.
- Lo que cambie en el laboratorio se mide con las baterias fijas y suma su fila
  al historico (H08P24).
