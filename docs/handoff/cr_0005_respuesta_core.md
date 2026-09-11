# Respuesta del core a extended CR-0005

Destinatario: hilo de gobernanza (`ia_nest_meta`) y, por su medio,
`ia_nest_extended`.
Autor: hilo de `ia_nest_core`, rol disenador.
Estado: reconciliado por el usuario (2026-09-11).
Base: `ia_nest_core` en `main`, commit `acc3c8b`, mas los cambios de contrato de
esta respuesta (`core ADR 0052`, sin publicar todavia).
Fecha: 2026-09-11
Responde a: `docs/change_requests/from-ia_nest_extended/solicitado/CR-0005-trace-en-prompt-stream.md`

## Resolucion

**ACEPTADO**, sin reformular. `core ADR 0052`, decision D2.

`prompt.stream` publicara en su evento `done` el mismo objeto `trace` que ya
publican `reasoning.stream` y `task.stream`.

## Verificado antes de aceptar, y es peor de lo que el CR dice

Medido por el cable contra `v0.4.0`, por REST, que es la via que consume esta
capa:

    prompt.stream     done -> {finish_reason, model, reasoning, text, tokens_in, tokens_out}
    reasoning.stream  done -> {..., trace: {request_id, capability, status, stop_reason, ...}}
    task.stream       done -> {..., trace: {request_id, capability, ...}}

De las TRES capacidades de streaming, dos publican `trace` con `request_id` y una
no. `prompt.stream` es la rara de tres, y ademas tampoco publica `domain`, que
sus hermanas si publican.

## Por que se acepta sin discutir la forma

Esto no es una capacidad nueva: es una DERIVA. La regla de compatibilidad del
propio core dice que sus interfaces no deben tener logica distinta, y aqui
divergen dos capacidades hermanas del mismo tipo. Que la mas barata de las tres
sea justo la que no se puede auditar no lo decidio nadie; se quedo asi.

El CR acierta ademas en descartar el atajo, y merece decirse: rellenar
`source_trace_id` con un identificador de la capa de encima produciria una traza
que PARECE completa y no lo es. Un hueco visible es mejor que un valor plausible
y falso. Es exactamente el criterio con el que el core acaba de disponer su
ADR 0051 sobre las subtareas vacias, un piso mas abajo.

## Lo que esto le dice al core sobre si mismo

`CR-0005` es la tercera asimetria interna que descubre una capa de encima, tras
la de `CR-0002` y la de la CLI que ignoraba su variable de configuracion. Y hay
una cuarta sin CR, encontrada por el propio core el 2026-09-11: `task plan` no
acepta `--quiet` ni `--verbose`, y `task run` si.

El patron es siempre el mismo: **capacidades hermanas con superficies distintas,
y ningun gate que lo mire.** Por eso la disposicion no se queda en anadir el
campo, y extiende el gate del catalogo a la salida (ADR 0052, D4). Anadir el
trace sin el gate seria arreglar este caso y esperar al siguiente.

## Version y entrega

Adicion compatible sobre un evento existente: **PATCH**. No cambian los eventos
de token, ni el formato del flujo, ni nada de `task.stream`.
