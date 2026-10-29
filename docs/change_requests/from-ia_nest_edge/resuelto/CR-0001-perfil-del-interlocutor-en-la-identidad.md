# CR-0001: el contexto de identidad dice que clase de interlocutor es

- id: CR-0001
- fecha: 2026-10-22
- capa origen: `ia_nest_edge`
- capa destino: `ia_nest_core`
- estado: entregado

## Caso de uso motor

El ente se abre a la familia del operador, ninos incluidos (meta ADR 0013). El
criterio duro del borde: ninguna peticion de un nino llega al modelo sin que el
ente sepa que habla con un nino.

El borde sabe quien habla, pero no debe decirle al ente COMO comportarse: eso
seria dirigir su pensar desde el exterior (`core ADR 0033`), y el juicio es de
conscience (`core ADR 0034`). Lo que el borde puede aportar es un HECHO sobre el
interlocutor, como ya aporta `user_id`.

El contrato no tiene donde ponerlo. El contexto de identidad
(`core CORE_CONTRACT.md`) tiene cinco campos cerrados, y hoy un campo desconocido
se descarta EN SILENCIO (`Identity.from_defaults` solo lee los cinco). Para este
caso, el silencio es el peor fallo posible: el atributo desaparece y nadie se
entera.

## Que se pide

Un campo opcional en el contexto de identidad que diga que clase de interlocutor
es, con un vocabulario cerrado. Que viaje y se refleje en la traza como los
otros cinco.

El core NO lo interpreta: no cambia modelo, perfil ni enrutado por el. Lo
transporta, para que lo lean quienes lo necesiten (hoy extended, manana los
checkpoints de conscience).

## Forma sugerida (no vinculante)

    identity.interlocutor_profile: "adult" | "child"   # opcional; ausente = sin dato

Un valor fuera del vocabulario es error tipado, no un descarte silencioso.

## Impacto

- Core: campo aditivo en el contexto de identidad -> PATCH segun
  `meta docs/POLITICA_SEMVER.md` (pre-1.0). Dentro del rango `>=0.4 <0.5` que
  declara extended.
- Borde: no bloquea su primera entrega, que se prueba con el operador. Bloquea la
  entrada de la familia, que ya espera por L5r y L4b de extended.

## Alternativas consideradas

- **Instrucciones de sistema por peticion.** Funcionaria, pero el borde diria al
  ente como pensar. Descartada por `core ADR 0033`.
- **Una variante del modelo con las instrucciones en su `SYSTEM`.** Sin cambio de
  contrato, pero un mensaje de sistema del perfil del core probablemente la
  sustituye: fallo silencioso del criterio duro. Queda como alternativa a
  comprobar en el laboratorio si este CR se atasca.
- **Prefijo en el texto del prompt.** Extended guarda el turno literal: las
  instrucciones contaminarian la memoria.
- **Solo en extended, como parametro de extension.** No llegaria al core, ni a
  su traza, ni a los checkpoints donde supervisara conscience.

## Tension doctrinal

Ninguna de fondo. El core no aprende nada de las capas de arriba: transporta un
dato de identidad, que es para lo que existe el contexto de identidad. Cuida
`core ADR 0033`: el hecho sale del borde, el juicio queda dentro del ente.

Hermano: `edge CR-0002` (extended lo transporta y lo compone).

## Resolucion

ENTREGADO y PUBLICADO el 2026-10-29 en `core v0.4.2` (PATCH; ficha
`core docs/fixes/v0.4/0013`, core#49 y core#50). Forma: la sugerida,
`identity.interlocutor_profile: "adult" | "child"`, opcional; un valor fuera
del vocabulario es error de validacion (400 en REST). El core lo transporta y
lo traza y no lo interpreta. La telemetria sube a `schema_version` 2 (columna
final del CSV; core ADR 0015, estado posterior). Desplegado en el laboratorio
y comprobado en la traza y la telemetria (meta `H08`, paso H08P9).
