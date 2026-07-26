# IA_NEST Meta - Contexto

Estado: activo
Version: 1.0 - 2026-07-26

## Que es el ente IA_NEST

IA_NEST define un ENTE con identidad propia (simulada), construido por capas,
cada una en su repo, con dependencias fijadas por SemVer (core ADR 0032/0033).
El ente no es un programa: es un conjunto de capas que se contratan entre si.

## Mapa de repos

Zona segun el mapa del core (core ADR 0033). El registro autoritativo de capas,
con su grafo de dependencias y su estado, vive hoy en el core
(`docs/FRONTERAS.md`); su re-hogacion a este repo esta propuesta y pendiente de
reconciliacion (meta ADR 0001).

| Zona | Repo | Que es |
|---|---|---|
| Taller | `ia_nest_meta` | Este repo. Gobernanza del ente. No es capa |
| Ente | `ia_nest_core` | El motor: enruta, infiere, orquesta `task.run` |
| Ente | `ia_nest_extended` | La memoria/conocimiento: enriquecimiento (via 2) |
| Ente | `ia_nest_core_conscience` | La mente voluntaria: etica y personalidad. Prevista |
| Ente | `ia_nest_core_pulse` | La mente involuntaria: regula parametros. Prevista |
| Ente | `ia_nest_web` | La cara: GUI de gestion y presentacion. Prevista |
| Exterior | `ia_nest_agents`, `ia_nest_external_*`, `ia_nest_module_*` | Consumen el ente por sus contratos publicos |

## Por que meta esta fuera del mapa ente/exterior

El core ADR 0033 parte el mundo en dos zonas: lo que ES el ente y lo que lo
CONSUME desde fuera. Meta no es ninguna de las dos:

- No es capa del ente: no infiere, no recuerda, no delibera, no presenta. Si
  desaparece en tiempo de ejecucion, el ente sigue funcionando igual.
- No es exterior: no consume ningun contrato publico del ente.

Meta es el TALLER: el sitio donde se disena y se gobierna la construccion del
ente. Es un plano distinto, no una posicion dentro del mismo plano. Por eso su
zona propia es "Taller", y por eso este repo no aparece en la tabla del core:
un registro que no puede listar al repo que lo hospeda es, de hecho, el
argumento de que ese registro deberia vivir aqui.

## Que vive aqui

Regla de corte completa en `README.md` (charter). En resumen: lo que es del
ente, o de COMO se construye el ente, y tiene consumidor hoy.

Hoy, con consumidor real:

- El proceso de Change Request entre capas y los CR emitidos
  (`docs/change_requests/`). Es el caso puro de artefacto bilateral: un CR no
  pertenece ni a la capa que lo emite ni a la que lo decide, sino a las dos.
  Nacio sin hogar en `ia_nest_extended`, con su propio README declarando que su
  version autoritativa debia "elevarse al core". Ese sintoma es el motivo de
  este repo.

Propuesto y pendiente de reconciliacion (meta ADR 0001): registro de capas,
convenciones transversales, doctrina multi-IA, SemVer como politica de ente,
capas futuras, plantillas de brief.

## Que NO vive aqui

- El contrato publico de una capa. Vive en su repo, versionado por ella.
- Los ADR internos de una capa, incluidos los del core. Son historia de esa
  capa y no se mueven.
- El PLAN, el estado, el codigo, los tests y el despliegue de una capa.
- Cualquier feature. El mandato de meta es por CONCERN (gobernanza), no por
  repo: desde aqui se toca gobernanza en repos hermanos, nunca su feature.

## Disciplina

- Arranque minimo: primero lo que tiene consumidor hoy. No hay andamio
  especulativo (core ADR 0035).
- Migracion deliberada: la doctrina de ente que vive en el core se re-hoga por
  pasos reconciliados, no de un tiron.
- Los ADR son historia: se les apendiza un puntero, no se reescriben.
- Docs pequenos, normativos, en ASCII sin acentos. Identificadores en ingles
  snake_case (core ADR 0016). Repo publico sin datos internos.
- Modo ciego multi-IA con reconciliacion del usuario: solo lo reconciliado se
  registra.
