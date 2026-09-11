# Capas futuras del ente

Estado: activo
Version: 1.2 - 2026-08-21

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

## Quien decide y quien orquesta lo que el ente hace

Registrado el 2026-08-21, al disenar la fase de datos web de
`ia_nest_extended`. Son DOS preguntas con la misma forma, y ninguna tiene dueno:

1. **Que capacidad usar.** Hoy nadie decide si una peticion es atomica
   (`prompt.run`) o descomponible (`task.run`): el core no lo hace por diseno y
   extended reexpone ambas sin elegir, de modo que le toca saberlo al operador.
   Observado en uso real (`extended docs/PLAN.md`, "fuera de este plan").
2. **Cuando el ente sale a buscar fuera, y quien lleva ese bucle.** Al querer que
   el ente investigue -formular una consulta, buscar, abrir, leer, y reformular
   si no encontro-, alguien tiene que decidir que se sale y cuando se para.

No es un fallo de ninguna capa existente. El core no elige por diseno; extended
enriquece, y elegir no es enriquecer (su `ALCANCE.md` manda el juicio a
conscience). Es una funcion del ente que quedo entre las dos.

**Lo que NO entra aqui:** el MECANISMO de la busqueda web -redactar la consulta,
traer, extraer, buscar dentro del documento, suelo y presupuesto- es de
`ia_nest_extended`, que ya tiene datos web asignados, y vive en su
`docs/PLAN.md` (Fase 6, aparcada). Aqui solo esta quien decide y quien orquesta.

**Disparador**: cuando exista un consumidor que ejerza la costura. Hoy el
operador suple las dos decisiones a mano y basta; deja de bastar en cuanto el
ente deba iniciar trabajo sin que alguien se lo dicte paso a paso.

**Nota de alcance, para cuando se aborde:** ojo con confundirlo con conscience.
Conscience decide QUE clase de ente es y que merece consolidarse; esto decide
COMO se ejecuta un encargo concreto. Son juicios de naturaleza distinta y
mezclarlos daria una capa que lo decide todo.

**Un obstaculo ya conocido**: iterar exige re-planificar con lo hallado, y hoy
re-planificar y enriquecer son mutuamente excluyentes (`extended docs/PLAN.md`,
coste declarado de su Fase 7b). Quien aborde esto se lo encontrara, y el canal
para resolverlo es un CR al core.

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
