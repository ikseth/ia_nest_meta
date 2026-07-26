# ia_nest_meta

Gobernanza del ente IA_NEST. Este repo es el TALLER donde se construye el ente:
un plano distinto del de las capas, no una capa mas.

Estado: activo
Version: 1.0 - 2026-07-26

## Proposito

El ente IA_NEST se construye por capas, cada una en su repo, con dependencias
fijadas por SemVer. El core, por ser el primero, absorbio gobernanza que no
pertenece al motor. Este repo la desenreda y le da hogar:

- Hogar NEUTRAL para artefactos bilaterales: los que no son de ninguna capa
  concreta porque son de dos (los Change Requests entre capas).
- Hogar de la doctrina de ENTE (como se construye el ente), frente a la
  doctrina de cada capa (que hace y como se contrata cada una).

Meta esta FUERA del mapa ente/exterior del core (core ADR 0033): no es una capa
del ente ni un consumidor exterior. Es el taller donde ambos se disenan. Ver
`IA_NEST_META_CONTEXT.md`.

## Charter (regla de corte)

Regla estricta, deliberada, para evitar el repo-Dios. Ante la duda, se queda en
la capa.

### Vive aqui

- Proceso y ficheros de Change Request (CR) entre capas.
- Registro de capas y grafo de dependencias del ente.
- Convenciones transversales: docs en ASCII puro (sin acentos ni enye),
  identificadores en ingles snake_case, SemVer.
- Doctrina multi-IA y roles de trabajo.
- Capas futuras: concerns de ente que aun no tienen repo.
- Plantillas de brief y handoff.

### Se queda en cada capa

- Su contrato publico.
- Sus ADR internos.
- Su PLAN y su estado.
- Su codigo, sus tests, su despliegue.

### Prueba de corte

Ante un artefacto nuevo, dos preguntas:

1. Es de una sola capa? Vive en esa capa. Fin.
2. Es del ente, o de COMO se construye el ente? Puede vivir aqui.

Y una tercera que manda sobre las dos: tiene consumidor HOY? Si no lo tiene, no
se construye. Es la leccion del core ADR 0035: una costura sin consumidor real
se pudre. Este repo arranca minimo a proposito, y crece por migraciones
reconciliadas, no por anticipacion.

## Estado actual

Arranque minimo. Lo unico con consumidor real hoy es el proceso de Change
Request, que vivia sin hogar en `ia_nest_extended`:
`docs/change_requests/`.

El resto de la doctrina de ente listada en el charter sigue viviendo en el core
y se re-hoga por pasos reconciliados, no de un tiron. Los ADR del core NO se
reescriben ni se mueven: son historia. Ver meta ADR 0001.

## Orden de lectura

Ver `AGENTS.md`.
