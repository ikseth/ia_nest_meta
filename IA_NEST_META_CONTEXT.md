# IA_NEST Meta - Contexto

Estado: activo
Version: 1.0 - 2026-07-26

## Que es el ente IA_NEST

IA_NEST define un ENTE con identidad propia (simulada), construido por capas,
cada una en su repo, con dependencias fijadas por SemVer (core ADR 0032/0033).
El ente no es un programa: es un conjunto de capas que se contratan entre si.

## Mapa de repos

En `docs/REGISTRO_CAPAS.md`: quien existe, en que zona, quien depende de quien,
que version tiene publicada y en que estado esta (meta ADR 0003).

Aqui no se repite: una segunda tabla de las mismas filas, en el repo cuyo
trabajo es eliminar duplicaciones, seria indefendible.

Tres zonas: **Ente** (el pack basico con identidad propia simulada) y
**Exterior** (lo que consume el ente por sus contratos publicos), ambas de
`core ADR 0033`; y **Taller**, que es este repo y no cabe en ninguna de las dos.

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

- Las convenciones transversales (`docs/CONVENCIONES_TRANSVERSALES.md`) y la
  doctrina multi-IA (`docs/DOCTRINA_MULTI_IA.md`), re-hogadas desde el core en
  meta ADR 0002. Gobiernan a todas las capas, luego no podian vivir dentro de
  una de ellas sin obligar a las demas a citarla.

- El registro de capas y la regla de vinculo (`docs/REGISTRO_CAPAS.md`),
  re-hogados en meta ADR 0003. El argumento decisivo: la tabla tiene zonas
  "Ente" y "Exterior", y este repo no cabe en ninguna de las dos.

- La politica de SemVer (`docs/POLITICA_SEMVER.md`) y los concerns del ente sin
  repo asignado (`docs/CAPAS_FUTURAS.md`), re-hogados en meta ADR 0004. Cada
  capa conserva lo unico que no puede ser comun: QUE cuenta como su contrato
  publico.

- La plantilla de brief (`docs/PLANTILLA_BRIEF.md`), levantada en regimen de
  uso cuando aparecio su consumidor (meta ADR 0005). Fija la FORMA del brief;
  el QUE debe llevar sigue en `docs/DOCTRINA_MULTI_IA.md`.

Con eso queda agotada la migracion abierta en meta ADR 0001. Diferido por falta
de consumidor: la plantilla de handoff.

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
