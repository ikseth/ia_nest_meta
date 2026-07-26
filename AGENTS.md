# Instrucciones para agentes de IA en ia_nest_meta

Antes de proponer o valorar cualquier cambio, lee en este orden:

1. `README.md` (proposito y charter: la regla de corte)
2. `IA_NEST_META_CONTEXT.md` (que es el ente, mapa de repos, frontera del taller)
3. `docs/change_requests/README.md` (proceso de CR entre capas)
4. ADRs en `docs/decision_records/`

Contexto de las capas: la doctrina viva del ente todavia vive en buena parte en
`ia_nest_core` (`docs/FRONTERAS.md`, `docs/CONVENCIONES.md`, `docs/VERSIONADO.md`,
core ADR 0016/0030/0031-0037). Su re-hogacion a este repo es progresiva y
reconciliada. Ver meta ADR 0001.

## Que es este repo, y que no

Es gobernanza. No hay codigo, ni contrato de runtime, ni dependencia de ninguna
capa. Meta no es una capa del ente ni un consumidor exterior (core ADR 0033).

El mandato de este repo es por CONCERN (gobernanza), no por repo: desde aqui se
puede tocar gobernanza en repos hermanos, pero NUNCA el feature de una capa (su
`src/`, sus fases, su contrato). Eso vive en el hilo de esa capa.

## Reglas

- **Charter estricto.** Si un artefacto es de una sola capa, no entra aqui. Si
  no tiene consumidor hoy, no se construye (core ADR 0035).
- **Los ADR son historia.** No se reescriben ni se mueven entre repos. Cuando
  una regla viva se re-hoga, el ADR de origen conserva su cuerpo intacto y solo
  se le APENDIZA una seccion `## Estado posterior` con el puntero. Aqui se
  registra el movimiento con un ADR propio, para que el rastro sea bidireccional.
- **Migracion deliberada, no big-bang.** Cada paso de re-hogacion se propone,
  se reconcilia con el usuario y se ejecuta como commit separado por repo.
- **Citas de ADR:** siempre `<repo> ADR NNNN` (por ejemplo `core ADR 0032`),
  nunca `ADR NNNN` a secas. La numeracion es por repo y colisiona entre repos.
- **Convencion de texto:** los docs se escriben en ASCII puro, sin acentos y sin
  `n` con virgulilla (`diseno`, `pequeno`, `dueno`). Los identificadores de
  codigo y las claves de configuracion van en ingles snake_case (core ADR 0016).
- **Docs pequenos y normativos.** Cada doc normativo lleva cabecera `Estado:` y
  `Version: X.Y - fecha`. Meta no adopta SemVer formal todavia (diferimiento
  explicito, meta ADR 0001).
- **Multi-IA en modo ciego:** si detectas una inconsistencia entre documentos,
  no asumas que es un error propio: puede ser trabajo en curso de otra IA.
  Senalala, no la corrijas por inferencia. Solo el resultado reconciliado por el
  usuario se registra.
- **Repo publico:** nunca commitees IPs, hosts, rutas ni secretos internos.
- **No** cortes tags, **no** crees ramas remotas y **no** toques el core sin
  aprobacion explicita del usuario.
