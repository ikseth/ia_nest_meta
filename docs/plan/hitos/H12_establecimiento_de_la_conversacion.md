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

Por decidir (H12P1), falsable antes que mecanismo. Propuesta de partida:

1. Por la voz de la casa, las frases del caso de origen llegan al dominio que
   tiene el dato y responden con el dato real, sin tocar la descripcion de
   ningun dominio.
2. La misma pregunta desde el chat, sobre otro lugar, se responde sobre ese
   lugar o se pregunta cual; nunca se inventa un lugar.
3. Cada sesion registra sus participantes y lo que se sabia al abrirla.
4. Un recurso nuevo expuesto por un proveedor declarado se alcanza sin editar
   configuracion del ente, y uno no declarado nunca se consulta.
5. Ningun permiso cambia por lo que diga una conversacion.

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H12P0 | Caso de origen y revision del borrador | este hilo | - | hecho el 2026-11-19 (arriba y en `pasos/H12P0_caso_de_origen.md`) |
| H12P1 | Criterio de salida | este hilo | H12P0 | pendiente |
| H12P2 | Practica habitual contrastada y ADR de meta sobre el establecimiento de la conversacion (contrato entre capas, meta ADR 0007 y 0009) | este hilo | H12P1 | pendiente |
| H12P3 | Interlocutores como entidades: que minimo de extended ADR 0004 hace falta y como siembra el operador | este hilo, extended | H12P2 | pendiente |
| H12P4 | Catalogo de recursos: hogar, que se declara y que se descubre, como se clasifica | este hilo, core | H12P2 | pendiente |
| H12P5 | Forma del saludo por superficie (piel OpenAI, `/rest`, MCP) y por consumidor | este hilo, borde, consumidores | H12P2 | pendiente |
| H12P6 | CR por capa y orden de entrega | este hilo | H12P3-H12P5 | pendiente |

## Reglas del hito

- Una decision cada vez, con diagrama y explicacion breve; criterio falsable
  antes que mecanismo; cada mecanismo situado frente a la practica habitual.
- En meta, papeles y motivos; nombres de maquina, direcciones y evidencia, en
  `ia_nest_lab`.
- Lo que cambie en el laboratorio se mide con las baterias fijas y suma su fila
  al historico (H08P24).
