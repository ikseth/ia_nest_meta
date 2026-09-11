# Respuesta del core a extended CR-0004

Destinatario: hilo de gobernanza (`ia_nest_meta`) y, por su medio,
`ia_nest_extended`.
Autor: hilo de `ia_nest_core`, rol disenador.
Estado: reconciliado por el usuario (2026-09-11).
Base: `ia_nest_core` en `main`, commit `acc3c8b`, mas los cambios de contrato de
esta respuesta (`core ADR 0052`, sin publicar todavia).
Fecha: 2026-09-11
Responde a: `docs/change_requests/from-ia_nest_extended/solicitado/CR-0004-render-en-el-catalogo.md`

## Resolucion

**ACEPTADO**, en la forma gruesa que el propio CR proponia. `core ADR 0052`,
decisiones D3 y D4.

## El argumento con el que se iba a rechazar, y por que se cayo

El CR nombra la tension con honestidad: la presentacion no es logica de motor, y
el core puede decir que su contrato describe capacidades y no pieles. Esa era la
respuesta que este rol tenia preparada.

No se sostiene, y lo tumba el propio catalogo. Medido en `v0.4.0`, una capacidad
publica hoy `description`, `epilog`, `metavar`, `flag_help` y `order`:
presentacion pura, publicada desde `core ADR 0046`. **El core ya publica su piel;
lo que hace es publicar solo la mitad de ENTRADA y llamar privada a la de
SALIDA.** El catalogo describe exhaustivamente como se LLAMA a una capacidad y
dice de la respuesta una sola cosa: si es streaming o no.

Esa asimetria no es solo incomoda para quien reenvia. Es la razon de que el gate
de ADR 0046 -que existe para que las interfaces no deriven, y que destapo `tags`
sin que nadie lo tocara- no pudiera ver el caso de `CR-0005`: ese gate compara
superficies de entrada, y nada vigila la salida.

## Que se entrega

Un descriptor de presentacion por capacidad, en su proyeccion de CLI, con cuatro
formas sacadas de lo que los renderizadores HACEN hoy y no de lo que seria bonito
soportar:

- `text`: un campo que se imprime tal cual (`prompt.run` -> `response`);
- `row`: una fila de campos (`domain.route` -> `domain`, `model`, `reason`);
- `table`: una fila por elemento de una lista, con sus columnas (`task.plan` ->
  `plan[]` con `index`, `domain`, `prompt`);
- `opaque`: el core no declara presentacion para esa respuesta.

`opaque` es la parte que este CR pedia con esas palabras y que conviene subrayar:
**es informacion, no una escapatoria.** Medido, tres renderizadores no caben en
las otras formas -`runtime.detect` deriva una etiqueta de un booleano,
`model.pull` pinta dos listas con un prefijo literal, `config.validate` imprime
una constante que ni siquiera esta en la respuesta-. Declararlos opacos le dice a
quien reenvia "aqui se cae a JSON a proposito", que es distinto de no saberlo.

El gate del catalogo se extiende a la salida (D4): toda capacidad con proyeccion
de CLI declara presentacion, y lo declarado se compara con lo que su renderizador
hace. Sin eso, esto naceria siendo documentacion que envejece.

## La frontera, que el core SI defiende

El core declara **que se lee**; nunca **como se pinta**. Ni formato, ni anchos,
ni colores, ni idioma. Si mas adelante hiciera falta algo de eso, no entra por
ampliacion silenciosa de este descriptor: entra por su CR.

Fuera de alcance tambien, y explicito: la presentacion del PROGRESO de las
capacidades de streaming. Eso es una vista en vivo de eventos que la capa de
encima ya modela, no la presentacion de una respuesta.

## Version y entrega

Adicion compatible: **PATCH**. Va en la misma linea del PLAN que `CR-0003` y
`CR-0005`, porque los tres son el mismo punto ciego.
