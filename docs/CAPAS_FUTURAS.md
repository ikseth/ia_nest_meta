# Capas futuras del ente

Estado: activo
Version: 1.4 - 2026-10-27 (pasarela del recinto, H03)

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

**Estado al 2026-10-22 (meta ADR 0013).** `ia_nest_agents` recibe forma, y con
ella el bucle de un encargo que llega desde fuera: es del agente que lo lleva.
Siguen sin dueno la pregunta 1 (la piel OpenAI del borde la suple con una
correspondencia fija, sin resolverla) y lo que el ente haga por iniciativa
propia.

**Un obstaculo ya conocido**: iterar exige re-planificar con lo hallado, y hoy
re-planificar y enriquecer son mutuamente excluyentes (`extended docs/PLAN.md`,
coste declarado de su Fase 7b). Quien aborde esto se lo encontrara, y el canal
para resolverlo es un CR al core.

## Pasarela del recinto (infraestructura del ente)

Registrado el 2026-10-27, en `H03` de la hoja de ruta. La arquitectura de
referencia de IA_NEST es un recinto autocontenido detras de una pasarela
(`gate`): un interfaz delantero que publica solo los puntos de uso del ente y
uno trasero que da a una red contenida sus recursos (directorio, DNS, DHCP,
proxies, CA). Detalle en `docs/plan/hitos/H03_arquitectura_de_sistemas.md`.

No es una capa con contrato de runtime: es infraestructura del ente. Su hogar
previsto es un repo propio, `ia_nest_gate`, con instalador versionado (primer
arranque de openSUSE MicroOS y compose de sus servicios).

Hoy el instalador vive en el laboratorio, sin repo, por decision del usuario
(2026-10-27): no se invierte en el repo mientras la pasarela se descubre
montandola.

**Disparador**: cuando la pasarela tenga que reproducirse fuera del
laboratorio (otro entorno, o una segunda pasarela), o cuando su instalador
deje de cambiar a cada paso. Entonces se crea `ia_nest_gate`, el instalador
muda alli y esto sale como fila "prevista" en `REGISTRO_CAPAS.md`.

## Concerns que ya tienen repo

Se dejan solo como puntero, para que las referencias desde otros repos no queden
colgando.

- **Autenticacion de los interlocutores.** Hogar decidido en meta ADR 0011 (el
  borde); repo asignado en meta ADR 0013: `ia_nest_edge`, zona Exterior.
