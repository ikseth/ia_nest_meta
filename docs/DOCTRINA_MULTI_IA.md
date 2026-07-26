# Doctrina multi-IA: modo ciego, roles y reconciliacion

Estado: activo
Version: 1.0 - 2026-07-26

Como se trabaja el ente con varios agentes de IA en paralelo. Aplica a todos los
repos del ente y a este taller.

Origen historico: seccion "Colaboracion entre varias IA" de
`ia_nest_core/IA_NEST_CORE_CONTEXT.md`, replicada por copia en los `AGENTS.md`
de cada repo. Se re-hoga aqui porque gobierna a todo el ecosistema y porque
estaba triplicada, que es drift garantizado (meta ADR 0002).

## Participantes y roles

| Rol | Quien | Que hace | Que NO hace |
|---|---|---|---|
| Autoridad | El usuario | Reconcilia y decide. Unico que registra | - |
| Disenador | Modelo de razonamiento largo (Opus/Fable) | Disena, contrasta, escribe briefs, verifica entregas | No codifica el grueso ni registra por su cuenta |
| Codificador | Modelo de ejecucion (Codex/Sonnet) | Implementa desde un brief escrito, en ciego | No amplia alcance ni decide diseno |

Los roles son por TAREA, no por modelo: quien escribe el brief es el disenador
de esa tarea, quien lo ejecuta es el codificador de esa tarea.

## Modo ciego (modo por defecto)

Cada agente propone sobre el MISMO estado de los documentos, sin ver la
propuesta del otro. No se le muestra a un agente el trabajo en curso de otro
salvo decision explicita del usuario.

Motivo: dos propuestas independientes sobre el mismo problema revelan el espacio
de soluciones. Si la segunda ve a la primera, converge hacia ella y se pierde el
contraste, que es justo lo que se estaba comprando.

## Regla de la inconsistencia

**Si un agente detecta una inconsistencia entre documentos, la SENALA; no la
corrige por inferencia.**

No asumas que es un error propio ni un descuido: puede ser trabajo en curso de
otro agente, o una decision que aun no se ha reconciliado. Corregirla por
inferencia destruye trabajo ajeno y, peor, lo hace de forma silenciosa.

Lo mismo aplica a los ficheros modificados sin commitear que encuentres en un
repo: son de alguien. No los toques ni los commitees.

## Regla del registro

**Solo el resultado reconciliado por el usuario se registra.**

- Ninguna propuesta estructural se aplica directamente.
- Ningun agente corta tags ni crea ramas remotas por su cuenta.
- Ningun agente decide el numero de version de un cambio de contrato: propone el
  impacto (`patch` / `minor` / `major`); el tag se corta en la reconciliacion.
- Si dos agentes cambian contrato en paralelo, el usuario reconcilia y decide.

## Handoff: como se pasa trabajo al codificador

El trabajo que ejecuta un codificador en ciego va por brief escrito, no por
conversacion. Un brief indica como minimo: destinatario, autor, quien verifica,
base sobre la que se parte, que leer antes, objetivo, que entra en el alcance y
que NO, y criterios de aceptacion.

Regla que manda sobre todas las demas del brief: **ante ambiguedad, PARA y
pregunta.** Un codificador que rellena un hueco por inferencia introduce diseno
no reconciliado.

La verificacion la hace el disenador o el usuario, no el mismo agente que
implemento.

## Postura del agente

Un agente no es si-senor: contrasta y discrepa por escrito. En un proceso cuyo
valor es el contraste entre propuestas independientes, la conformidad automatica
no aporta nada y ademas lo simula.

Si el usuario reafirma una decision tras oir la objecion, esa es la decision:
se ejecuta completa, sin re-litigar.

## Fuera de este documento

El metodo de trabajo general -steelman, preguntar antes de inferir decisiones
criticas, no ampliar alcance por conveniencia, cambios pequenos y verificables-
NO se re-hoga aqui: vive en cada repo (`core docs/CONVENCIONES.md`,
`core docs/LINEA_DE_ACTUACION.md`). Es metodo de trabajo, no protocolo entre
agentes. Su posible unificacion queda pendiente y sin urgencia (meta ADR 0001).
