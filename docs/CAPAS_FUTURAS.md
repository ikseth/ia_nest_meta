# Capas futuras del ente

Estado: activo
Version: 1.0 - 2026-07-26

Concerns del ENTE que no tienen repo asignado. Se registran aqui para no
perderlos: son fronteras futuras del ente, no trabajo pendiente de ninguna capa
existente.

Origen historico: `core docs/CAPAS_FUTURAS.md`, que mezclaba tres cosas de
duenos distintos (meta ADR 0004).

## Que entra aqui, y que no

**Entra:** un concern del ente sin repo asignado. Cuando se le asigna repo, sale
de aqui y entra como fila "prevista" en `REGISTRO_CAPAS.md`.

**No entra:**

- Trabajo pendiente del core (senales que debe exponer, capacidades que otra
  capa le pedira). Es backlog del motor y vive en `core docs/CAPAS_FUTURAS.md`
  y su `PLAN.md`.
- Diseno interno de una capa ya prevista con repo asignado (conscience, pulse,
  web). Vive donde este marcado como deuda declarada hasta que se siembre su
  repo, y entonces muda alli. Meta gobierna COMO se construye el ente, no que
  hace cada pieza.
- Lo que una capa ya existente ha absorbido (memoria avanzada ->
  `ia_nest_extended`, RAG y datos web -> `ia_nest_extended`).

## Comunicacion entidad-a-entidad

Varios entes IA_NEST comunicandose entre si (`core ADR 0033`). Es una frontera
futura del ente: no es una capa dentro de un ente, sino el protocolo por el que
un ente completo habla con otro.

Sin diseno asignado y sin repo. Se registra para no perderlo.

Nota de alcance, para cuando se aborde: `core ADR 0033` situa el exterior como
lo que CONSUME el ente sin dirigir su pensar. Otro ente no es un consumidor: es
un par. La zona que le corresponde en `REGISTRO_CAPAS.md` esta por decidir, y
probablemente exija revisar el mapa de dos zonas.
