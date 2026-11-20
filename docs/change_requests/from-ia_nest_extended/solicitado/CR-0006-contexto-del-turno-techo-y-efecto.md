# CR-0006: contexto del turno, proveedores como techo y efecto de la peticion

- id: CR-0006
- fecha: 2026-11-20
- capa origen: `ia_nest_extended`
- capa destino: `ia_nest_core`
- estado: propuesto
- tandas: tramo A en la tanda A de `H12`, tramo B en la tanda B (meta H12P6)
- ajuste 2026-11-20 (meta H12P7, D1): el punto 1 del tramo A espera a meta `H13`

## Caso de uso motor

La linea base de `H12` (H12P1b, 2026-11-19). Por la voz y por Open WebUI, las
preguntas sobre datos de la casa no llegan a `domotica` (0/24 en las dos):
"va a llover?" acaba en `general`. `DomainRouter.route(prompt)` clasifica solo
con el texto y las descripciones de los dominios. Con el dominio fijado, el
techo es 6/24: el modelo no sabe que hay en la casa. Y el mismo dia, una
pregunta de lectura por la voz puso una pelicula: el turno ofrecia una accion
que la pregunta no pedia (OWASP LLM06:2025).

Lo decidido en meta: meta ADR 0017 (el marco de la conversacion, puntos 5 y 7)
y meta ADR 0018 (el catalogo de recursos, puntos 2, 4 y 6). Extended compone el
marco y el catalogo; el core los recibe como dato y decide con ellos lo que es
suyo: enrutar, ofrecer herramientas y responder.

## Que se pide

### Tramo A (tanda A: catalogo y efecto)

1. **Contexto del turno, como dato.** Recibir de extended una composicion del
   catalogo (resumen fijo por area y clase, y preseleccion de entradas con su
   nombre exacto) y usarla en dos sitios: al enrutar, como pista y no como
   regla (0018, D4); y al responder, como contexto del modelo. No cambia la
   politica de herramientas (0017, punto 5).
2. **Proveedores como techo.** La configuracion de `tool_providers` pasa a ser
   el techo: a que se puede conectar y con que secreto. Extended manda en cada
   turno lo confiado; el core consulta solo la interseccion. Lo confiado fuera
   del techo no se consulta nunca (0018, punto 2). Reformula core ADR 0053, D6.
3. **Efecto de la peticion.** Toda clasificacion de un turno con herramientas
   devuelve su efecto, `read` u `order`, como resultado declarado; en la duda,
   `read`. Con `read`, solo las herramientas con `readOnlyHint`; con `order`,
   lo permitido y no destructivo, como hoy. Tambien en dominio fijo con
   herramientas, con una llamada corta al modelo del enrutador (0018, D5).
4. **El efecto solo resta.** Interseccion de lo permitido con el efecto, nunca
   suma; `tools_offered` y la traza registran el efecto y el resultado.

### Tramo B (tanda B: el marco)

5. **El marco en el mismo contexto.** Recibir el marco compuesto (cuenta,
   persona, dispositivo, lugar y lo que falta, cada parte con su estado; meta
   H12P3, D6) y usarlo como el catalogo: pista al enrutar, contexto al
   responder, sin tocar la politica.
6. **Preguntar como resultado declarado.** Si la respuesta depende de una parte
   del marco que falta, el turno termina con un resultado que lo dice (como
   `INPUT_REQUIRED` de A2A o `elicitation` de MCP), no con un texto que el
   modelo decide escribir o no (0017, punto 7). Lo que ya esta en el marco no
   se pregunta.

## Forma sugerida (no vinculante)

- Un campo opcional en las peticiones de `prompt.*` (y de la clasificacion),
  por ejemplo `turn_context`, con partes tipadas: `catalog` (resumen y
  preseleccion), `trusted_providers` (tramo A) y `frame` (tramo B). Sin el
  campo, el core se comporta como hoy (meta ADR 0007, regla 3).
- El efecto como campo de `RouteResult` y de la traza, junto al dominio.
- Preguntar como un `finish` o estado nuevo del turno, con la parte que falta.
- El core no conoce entidades ni sesiones: recibe hechos ya resueltos
  (0017, punto 5).

## Impacto SemVer

- Core: extension aditiva del contrato y cambio de semantica de
  `tool_providers` (de confianza a techo). Lo dispone el core; previsto MINOR
  por tramo (`v0.6.0` el A, `v0.7.0` el B, o uno solo si el core lo prefiere).
  Tocar el core requiere aprobacion del usuario.
- Extended necesitaria el tramo A para la tanda A y el B para la B.

## Alternativas consideradas

- **Ampliar a mano la descripcion de `domotica`.** Es el parche descartado al
  abrir `H12` (punto 4 de "decidido antes de abrir"): trata la frase, no la
  causa.
- **Que extended reescriba la peticion antes de llegar al core.** Mezcla dato
  con instruccion y el enrutador seguiria sin ver el catalogo.
- **Que el core lea entities.** El core no conoce entidades ni sesiones
  (0017, punto 5); la memoria es de extended (0018, D1).
- **Acotar tambien por clase de aparato.** Aplazado en 0018, D5: por si sola no
  resuelve el incidente; entra si la medida lo pide.

## Tension doctrinal

- Core ADR 0053, D6 (declarar un proveedor es el acto de confianza): 0018 lo
  matiza, declarar es el techo y la confianza se guarda en extended. Lo dispone
  el core en su ADR.
- Core ADR 0043 (el enrutador clasifica con texto y descripciones): gana una
  entrada mas, como pista.
- Coste declarado: una llamada mas en los turnos de dominio fijo con
  herramientas, que se mide con v1-ha.

## Como se mide

Tramo A: `marco-v1` por Home Assistant frente al techo (criterio 1 de `H12`),
una entidad virtual nueva alcanzable sin tocar la configuracion (criterio 6),
la frase del incidente sin herramientas que no sean de lectura en ninguna vuelta
(0018, D5), y v1-ha sin empeorar. Tramo B: criterios 2, 3, 5 y 7 de `H12`.

## Ajuste de la capa origen (2026-11-20)

Meta H12P7, D1 (decidido por el usuario): el conector entre Home Assistant e
IA_NEST pasa al hito `H13` de meta, porque la lectura de Assist no da ids de
entidad ni de area, y `extended CR-0007` queda en espera. Sin inventario no hay
resumen ni preseleccion de aparatos, asi que:

- **Tramo A, ahora:** puntos 2, 3 y 4 (proveedores como techo con lo confiado
  desde extended, efecto `read` u `order` tambien en dominio fijo, y el efecto
  solo resta). Del contexto del turno solo viaja `trusted_providers`.
- **Punto 1 (`catalog`: resumen y preseleccion):** espera a `H13`. La forma
  sugerida no cambia: es la misma parte opcional del mismo campo.
- **Como se mide el tramo A:** la frase del incidente sin herramientas que no
  sean de lectura en ninguna vuelta, un proveedor no declarado nunca consultado
  y v1-ha sin empeorar. El criterio 1 de `H12` se anota como referencia; el 1 y
  la primera mitad del 6 pasan a `H13`.
