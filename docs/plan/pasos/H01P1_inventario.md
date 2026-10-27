# H01P1: inventario de lo vivo de IA_NEST

Estado: hecho; asignacion validada por el usuario en `H01P2` (2026-10-27)
Fecha: 2026-10-27

Solo lectura. Cada entrada lleva su origen y una PROPUESTA de destino, que
decide el usuario en `H01P2`. Destinos posibles:

- `HNN`: entra en ese hito.
- `vigilado (bloquea HNN)`: trabajo de capa que se conduce en su capa y figura
  en la tabla "Trabajo de capa que esta hoja vigila" de la hoja de ruta.
- `capa`: detalle interno de una capa; se queda en su `PLAN.md`, sin hito,
  porque no fija orden entre capas (meta ADR 0014, punto 1).
- `descartado porque ...`.

Las marcadas con `(?)` son dudosas y se consultan una a una.

## A. Hallazgos de la replanificacion (semilla)

Origen comun: comentario de Deck #66 del 2026-10-27, artifact del diagnostico
y evidencia de la visita del borde (`ia_nest_lab/medidas/edge/`).

Validado por el usuario el 2026-10-27, tal cual.

| Id | Entrada | Origen | Propuesta |
|---|---|---|---|
| A01 | Diseno para Kodi: el borde descarta su `system` y no debe memorizar | visita del borde, #123 | H07 |
| A02 | HA sin estado de la casa (tambien viaja en `system`) | visita del borde, #123 | H07 |
| A03 | ~24 s hasta el primer token en `directo` | visita del borde | vigilado en extended (bloquea H08) |
| A04 | Menu de cada persona duplicado en la config de Open WebUI | artifact | H07 |
| A05 | La red critica es accesible desde la de personas y la de servicios | handoff de replanificacion | H03 |
| A06 | pgvector del RAG: servicio con datos en tron | artifact | H03 |
| A07 | Copia desfasada de `extended.setup.conf` en rocinante | #123 | H01P6 |
| A08 | core#45: `razonado` entrega su JSON interno y corta por `max_context_tokens` acumulado | visita del borde | vigilado en core (bloquea H08) |
| A09 | Cuentas locales de Open WebUI (solo laboratorio) | artifact | H04 (login contra el directorio) |
| A10 | Personas y claves Bearer en el fichero del borde | artifact | H05 |
| A11 | Reenvio REST/MCP del borde ("segunda entrega") | edge `PLAN.md`, artifact | H05 |
| A12 | Agente Linux por la piel OpenAI: revisar con el punto REST/MCP | artifact | H07 |
| A13 | "Borde primero con Open WebUI" como orden (meta ADR 0013, punto 5) | artifact | H02 |
| A14 | TLS de los tramos que cruzan el perimetro (proxy, CA propia del borde) | visita del borde | H06 |
| A15 | Tareas de fondo de Open WebUI: 1-3 por turno por el modo del chat | visita del borde | H07 |
| A16 | `iterativo`: `replan_unavailable` y `max_iterations` | visita del borde | vigilado: es `edge CR-0003` (C03) |

## B. Tarjetas de Deck (tablero Casa)

Validado por el usuario el 2026-10-27, tal cual.

| Id | Entrada | Origen | Propuesta |
|---|---|---|---|
| B01 | #66 IA_NEST: el ente (meta y vision) | Deck, En progreso | sigue como paraguas de la hoja de ruta, con los ids en el titulo |
| B02 | #123 IA_NEST: borde de acceso | Deck, En progreso | Terminado con puntero: primera entrega y visita hechas; lo pendiente ya esta en H02, H05, H06, H07 |
| B03 | #124 IA_NEST Agents: capa agentica | Deck, Por Hacer | H07 (titulo `H07 ...`) |
| B04 | #136 IA_NEST Agents: entorno propio del agente Linux | Deck, Futuros | H07: se funde en #124 y pasa a Terminado con puntero |
| B05 | #125 IA_NEST Extended: hito siguiente | Deck, Por Hacer | vigilado (bloquea H08) por L5r y L4b; titulo `H08 ...` |
| B06 | #135 LDAP: estudiar alcance | Deck, Por Hacer | H04; el alcance ya esta fijado (solo IA_NEST, preparado para crecer); entorno `@global` |
| B07 | #96 IA_NEST Conscience: manifiesto y experimento | Deck, Futuros | descartado del plan: semilla sin consumidor; Terminado con puntero al registro |
| B08 | #102 Jarvis: chat web y Nextcloud | Deck, Por Hacer | descartado porque meta ADR 0013 ya lo resolvio (archivar); queda archivar `web_chat` (H01P6) |
| B09 | #126 tron: GTX 1060 para modelos especializados | Deck, Por Hacer, `@casa` | descartado del plan: es recurso del anfitrion; H03 lo cita como recurso |
| B10 | #121 kodi_content_adviser: ajuste fino | Deck, En progreso | descartado porque es proyecto propio; su entrada como consumidor es A01 |
| B11 | Tarjeta del hito abierto | no existe | crear `H01 Migrar plan y tarjetas al modelo` |

## C. Issues y CR abiertos

No hay PR abiertos en ningun repo del ente.

Validado por el usuario el 2026-10-27, tal cual; aprobado cerrar core#36.

| Id | Entrada | Origen | Propuesta |
|---|---|---|---|
| C01 | `edge CR-0001`: perfil del interlocutor | core#44, meta CR | vigilado (bloquea H08); ya en la hoja |
| C02 | `edge CR-0002`: extended lo compone | extended#81, meta CR | vigilado (bloquea H08); ya en la hoja |
| C03 | `edge CR-0003`: modo iterativo de uso normal | extended#82, meta CR | vigilado (no bloquea ningun hito) |
| C04 | Avisos de extended del 2026-08-22 | core#36 | descartado porque lo cerro la tanda v0.4.1 (linea de la senal de vacio); cerrar el issue con aprobacion (H01P6) |
| C05 | CR-0003, CR-0004 y CR-0005 siguen en `solicitado/` | meta `change_requests/` | H01P6: a `resuelto/`, estado entregado en core v0.4.1 |

## D. `PLAN.md` y backlog de cada capa

Validado por el usuario el 2026-10-27, tal cual. El core no se toca en `H01P4`.

| Id | Entrada | Origen | Propuesta |
|---|---|---|---|
| D01 | Degradaciones de fanout sin numero de iteracion | core `PLAN.md`, tanda v0.4.1 | capa |
| D02 | Quiesce (fase v0.2-4), diferido hasta el modo sueno de conscience | core `PLAN.md` | capa |
| D03 | Ficha 0012 aparcada, `capability.route`, `tool_contracts` | core `PLAN.md` | capa |
| D04 | Backlog de lo que el core debe a pulse, conscience y memoria | core `CAPAS_FUTURAS.md` | capa |
| D05 | Reranker, CLI como cliente, cobertura e indice | extended `PLAN.md`, hito siguiente | capa |
| D06 | Fase 6 (datos web) aparcada; D3 (identidad conmutable) | extended `PLAN.md` | capa |
| D07 | Mejoras futuras: jev, modelos especializados | extended `docs/mejoras_futuras/` | capa |
| D08 | "La seleccion de capacidad" sin dueno | extended `PLAN.md` | descartado porque no tiene consumidor hoy (core ADR 0035); el menu de modos del borde la suple |
| D09 | `ALCANCE.md` dice que la autenticacion esta "sin decidir" | extended | H01P4 (commit de gobernanza) |
| D10 | `PLAN.md` desfasado (siguiente paso: el brief) y tabla que fija lo que espera de otras capas | edge | H01P4 |
| D11 | Aparcados: editar o regenerar en Open WebUI, `reasoning_effort`, valoraciones de usuario | edge `PLAN.md` | capa |
| D12 | Opcion `write_back=false` por consumidor (decision 3 del agente) | agents `PLAN.md` | H07 |
| D13 | "Va despues de la primera entrega de edge" | agents `PLAN.md` | H01P4 (orden entre capas, pasa al hito) |
| D14 | Aparcados: mutaciones, aprobador, agente Nextcloud | agents `PLAN.md` | capa |
| D15 | conscience y web: semillas sin plan | sus `README.md` | capa |
| D16 | Entidad-a-entidad, "quien decide y quien orquesta", pulse, `external_*`, `module_*` | meta `CAPAS_FUTURAS.md` | descartado porque no tienen consumidor hoy; siguen en `CAPAS_FUTURAS.md` |
| D17 | Repo `ia_nest` antiguo | GitHub | descartado porque es historia (cantera); agents ya cita lo que reaprovecha |

## E. Orden de la casa

Validado por el usuario el 2026-10-27, tal cual; aprobado borrar las ramas
remotas de E05 y archivar `web_chat` (B08).

| Id | Entrada | Origen | Propuesta |
|---|---|---|---|
| E01 | H00P4 hecho (PR #19 mergeado el 2026-10-27); H00 a `cerrados/` | meta | hecho en este paso |
| E02 | Fila del borde: `v0.1.1` y verificacion en vivo hecha | `REGISTRO_CAPAS.md` | H01P6 |
| E03 | Core registrado `v0.4.1`; rocinante corre `v0.4.0` | handoff de replanificacion | H01P6 (con A07): el registro lleva lo publicado y no cambia; actualizar rocinante es accion del operador |
| E04 | Enlaces de los handoffs de meta en `ia_nest_lab/enlaces.txt` | lab | H01P6, ya hecho el 2026-10-27 |
| E05 | Ramas ya mergeadas: `docs/dependencias-en-vivo` (edge), `docs/plan-agente-linux` (agents), `docs/sesion-como-entidad` (extended), `doctrina/orquestacion-desde-meta` (meta) | git | H01P6; borrar ramas remotas solo con aprobacion |
| E06 | Handoffs ya consumidos (core, extended, edge, replanificacion) | `ia_nest_lab/comun/traspasos/` | descartado porque son historia; se conservan |
