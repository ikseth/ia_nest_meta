# Capas futuras del ente

Estado: activo
Version: 1.1 - 2026-08-14

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

## Autenticacion de los interlocutores

Hoy nadie autentica. La identidad del request (`core CORE_CONTRACT.md`) es una
AFIRMACION de quien llama, no una prueba, y basta: en un CLI, quien tiene shell
en la maquina ya fue autenticado por el sistema operativo.

Y basta ademas porque `user_id` SEGMENTA, no AUTORIZA: la autoridad en este ente
la dan el principal en codigo (`extended ADR 0002`) y el GRANT del motor
(`extended ADR 0010`), no el usuario. Ningun `user_id` concede ni deniega nada;
solo separa hilos de recuerdo.

**Disparador**: deja de bastar cuando exista la primera superficie expuesta MAS
ALLA DE LA MAQUINA (una GUI, o una REST publicada). Ahi la identidad afirmada
pasa a ser afirmable por cualquiera.

**Dos hogares candidatos**, sin decidir:

1. **La superficie mas externa expuesta.** Autentica quien recibe del exterior, y
   afirma la identidad hacia dentro; las capas interiores la consumen como hoy.
   Encaja con el contrato uniforme (`meta ADR 0007`): entra como capa por encima
   sin editar a nadie. Contra: hoy esa superficie no existe.
2. **`ia_nest_extended`.** Es la capa de recursos extendidos del ente, y un repo
   dedicado solo a login es dificil de justificar. Contra: autenticar en una capa
   INTERMEDIA obliga a las de arriba a decidir si reautentican o confian, y la
   autenticacion no es enriquecimiento (su `ALCANCE.md`).

No se decide ahora, de forma deliberada: no hay consumidor que ejerza la costura,
y una costura sin consumidor real se pudre (`core ADR 0035`).
