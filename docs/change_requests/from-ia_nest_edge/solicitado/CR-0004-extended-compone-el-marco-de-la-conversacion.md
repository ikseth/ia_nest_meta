# CR-0004: extended transporta los hechos del saludo y compone el marco

- id: CR-0004
- fecha: 2026-11-20
- capa origen: `ia_nest_edge`
- capa destino: `ia_nest_extended`
- estado: propuesto
- tanda: B de `H12` (meta H12P6)

## Caso de uso motor

El ente tiene que saber con quien habla antes de responder (hito `H12`): quien
es el interlocutor, desde donde habla y en nombre de quien. Hoy el borde solo
afirma `user_id`, `service` y el perfil; la voz de la casa y la tableta comun
llegan iguales, y el lugar no se sabe. Criterios 2, 3 y 5 de `H12`.

Lo decidido en meta: meta ADR 0017 (el borde afirma hechos observables, extended
compone el marco y lo guarda en la sesion; saludo implicito en cada peticion),
meta H12P3 (entities y `session_frames`) y meta H12P5 (que hechos lleva cada
superficie). El borde afirmara desde su proxima version, en las tres caras,
tres hechos con su origen (H12P5, D2):

| Hecho | Origen | Que es |
|---|---|---|
| `account` | `authenticated` | la cuenta de servicio autenticada |
| `person` | `validated` | uid del directorio, o ausente |
| `device` | `asserted_by` la cuenta | id que manda el consumidor, acotado en forma, o ausente |

## Que se pide

1. **Transportar** los tres hechos en la identidad por CLI, REST y MCP, y
   reenviarlos donde haga falta. Hoy `MemoryIdentity` descarta lo que no
   conoce.
2. **No usarlos como clave de memoria**: son procedencia, como `service`
   (`FORMA_ENRIQUECIMIENTO`, decision 1). `user_id` sigue siendo la clave.
3. **Componer el marco siempre**, tambien sin enriquecimiento (como el perfil
   desde `edge CR-0002`): resolver `person` con la clave `directory:<uid>` y
   `device` con `account:<cuenta>/<id>` contra entities; deducir `place`
   (area del dispositivo, casa del dispositivo, casa de la persona); cada parte
   `known`, `unresolved` o `unknown`, con su origen (H12P3, D3 y D6). Si la
   entidad de la persona es un `device` con `shared` (el usuario compartido),
   la persona es `unknown` y el dispositivo es esa entidad; si ademas llega una
   cabecera de dispositivo, gana la cabecera (H12P5, D5).
4. **Guardarlo en la sesion**: `session_frames`, `seq 0` en el primer turno y
   una copia por cambio con su causa (H12P3, D6). Lo que no cambia no escribe.
5. **Pasarlo al core** como contexto del turno, cuando el core lo acepte
   (`extended CR-0006`, tramo B).
6. **No memorizarlo**: el marco nunca se escribe en entities ni en engramas
   (0017, punto 4).
7. **Declararlo en telemetria**: que el evento diga si el marco se compuso y
   con que estados, para contar el criterio 5.

## Impacto

- Extended: aditivo en la identidad y nuevo estado guardado -> MINOR. Depende
  del minimo de entities de H12P3 (brief a extended, tanda A).
- Borde: su version con los tres hechos (H12P5, D1-D3) se despliega a la vez
  que la de extended que los entiende; antes, el borde no los manda.

## Limites que el borde asume

- Un dispositivo no sembrado llega como `unresolved` y no crea entidad: lo
  siembra el operador (H12P3, D3).
- Nada de los tres hechos concede: los permisos siguen en el directorio
  (criterio 7 de `H12`).

## Tension doctrinal

Con el alcance de extended: componer y guardar el marco es memoria y mecanismo,
no juicio (0017, punto 4); como tratar a cada interlocutor sigue destinado a
conscience (0017, punto 10). Toca extended ADR 0015 (la sesion gana el marco) y
0004 (resolver contra entities): lo dispone extended en su ADR.
