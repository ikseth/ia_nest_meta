# Instrucciones para agentes de IA en ia_nest_meta

Antes de proponer o valorar cualquier cambio, lee en este orden:

1. `README.md` (proposito y charter: la regla de corte)
2. `IA_NEST_META_CONTEXT.md` (que es el ente, mapa de repos, frontera del taller)
3. `docs/CONVENCIONES_TRANSVERSALES.md` (reglas de texto, nombres y citas)
4. `docs/DOCTRINA_MULTI_IA.md` (roles, modo ciego, reconciliacion, handoff)
5. `docs/REGISTRO_CAPAS.md` (quien existe, quien depende de quien, regla de vinculo)
6. `docs/POLITICA_SEMVER.md` (esquema, que numero subir, publicacion)
7. `docs/change_requests/README.md` (proceso de CR entre capas)
8. `docs/CAPAS_FUTURAS.md` (concerns del ente sin repo asignado)
9. ADRs en `docs/decision_records/`

La migracion abierta en meta ADR 0001 esta completa (meta ADR 0002/0003/0004).
Este repo esta en regimen de uso: crece cuando entra un CR o se siembra una
capa, no por migracion.

Se queda en el core, por ser del motor: la costura que expone a cada capa
(`core docs/FRONTERAS.md`), que cuenta como su contrato publico
(`core docs/VERSIONADO.md`), su backlog (`core docs/CAPAS_FUTURAS.md`) y su
metodo de trabajo (`core docs/CONVENCIONES.md`).

## Que es este repo, y que no

Es gobernanza. No hay codigo, ni contrato de runtime, ni dependencia de ninguna
capa. Meta no es una capa del ente ni un consumidor exterior (core ADR 0033).

El mandato de este repo es por CONCERN (gobernanza), no por repo: desde aqui se
puede tocar gobernanza en repos hermanos, pero NUNCA el feature de una capa (su
`src/`, sus fases, su contrato). Eso vive en el hilo de esa capa.

## Reglas propias de este repo

Las reglas transversales del ente -texto en ASCII puro, identificadores en
ingles snake_case, citas `<repo> ADR NNNN`, modo ciego, regla de la
inconsistencia, regla del registro (no cortar tags ni ramas remotas), repo
publico- son los puntos 3 y 4 del orden de lectura y aplican tambien aqui. No se
repiten en este fichero: duplicarlas seria justo el error que este repo existe
para corregir.

Ademas, aqui:

- **Charter estricto.** Si un artefacto es de una sola capa, no entra aqui. Si
  no tiene consumidor hoy, no se construye (core ADR 0035).
- **Los ADR son historia.** No se reescriben ni se mueven entre repos. Cuando
  una regla viva se re-hoga, al ADR de origen se le APENDIZA una seccion
  `## Estado posterior` con el puntero. Aqui se registra el movimiento con un
  ADR propio, para que el rastro sea bidireccional.
- **Migracion deliberada, no big-bang.** Cada paso de re-hogacion se propone,
  se reconcilia con el usuario y se ejecuta como commit separado por repo.
- **Docs pequenos y normativos.** Meta no adopta SemVer formal todavia
  (diferimiento explicito, meta ADR 0001).
- **No** toques el core sin aprobacion explicita del usuario.
