# Registro de capas del ente y grafo de dependencias

Estado: activo
Version: 1.1 - 2026-08-14

Indice de los repos que forman el ente IA_NEST y de sus dependencias. Responde
a "quien existe, para que, y quien depende de quien" sin que ningun repo tenga
que crecer para alojar la respuesta.

Origen historico: `core ADR 0032` y la seccion "Registro de capas" de
`core docs/FRONTERAS.md`, donde vivio hasta meta ADR 0003.

## 1. Regla de vinculo entre capas

Vigente desde `core ADR 0032`; lo unico que cambia es quien hospeda el indice.

- Cada capa versiona su PROPIO contrato publico con SemVer.
- Una capa que depende de otra (o del core) FIJA por rango SemVer la version de
  la que depende, y lo declara de forma explicita en un manifiesto de su repo
  (por ejemplo `ia_nest_extended/docs/DEPENDENCIAS.md`).
- El contrato consumido vive en la capa que se consume, no en el core. El core
  NO absorbe logica entre-capas.
- Este registro es el INDICE. No sustituye a ningun contrato.

Cuando una capa necesita un cambio en el contrato de otra, el canal es un
Change Request (`docs/change_requests/`).

## 2. Fuente de verdad y deber de actualizacion

**Fuente de verdad.** La version de una capa es su tag de git; sus dependencias,
su manifiesto. Este registro es un indice y puede ir por detras. Ante
discrepancia, mandan el tag y el manifiesto, no esta tabla.

**Deber de actualizacion.** Si tu capa aparece en este registro, actualizar su
fila es parte de publicar una version; crear su fila es parte de sembrar el
repo. Sin un momento concreto al que engancharse, un indice se pudre: esta tabla
ya se pudrio una vez por no tenerlo (meta ADR 0003).

## 2b. Deber de aviso y deber de re-verificacion

El Change Request es el canal ASCENDENTE: la capa de abajo pide un cambio a la
de arriba. Faltaba el DESCENDENTE: avisar de que un contrato ha cambiado a quien
depende de el. Sin el, una capa sigue prometiendo compatibilidad con un rango que
ya no cumple, y el fallo es silencioso.

**Deber de aviso (capa que publica).** Publicar una version que toca tu contrato
publico incluye notificar a tus dependientes, por un canal DURADERO. Misma
exigencia que la notificacion de un CR: no vale un hilo de conversacion con un
agente, porque se evapora. Valen un issue en el repo del dependiente, un brief en
`docs/handoff/`, o la entrada de `CHANGELOG.md` de la version si el dependiente
tiene declarado que la vigila.

**Deber de re-verificacion (capa que depende).** Publicada una version nueva
arriba, re-verificar tu rango declarado y dejar el RESULTADO escrito en tu
manifiesto: rango que queda vigente, que se comprobo y como. No basta con subir
el techo del rango; hay que poder responder por que.

Precedente de la forma esperada: `ia_nest_extended/docs/DEPENDENCIAS.md` al
subir su rango a `>=0.2 <0.4`, donde consta que capacidades del core consume, que
ADR de la version nueva las tocan y que la compatibilidad se verifico en vivo.
Eso era buena practica; aqui pasa a ser deber.

## 3. El registro

Zonas: **Ente** y **Exterior** segun `core ADR 0033`; **Taller** para este repo,
que no es ninguna de las dos (ver `IA_NEST_META_CONTEXT.md`).

La columna "Costura" es una REFERENCIA, no una descripcion: la costura de cada
capa con el core esta descrita en `core docs/FRONTERAS.md`.

| Zona | Repo | Que es | Costura | Depende de | Version publicada | Estado de trabajo |
|---|---|---|---|---|---|---|
| Taller | `ia_nest_meta` | Gobernanza del ente | - | - | sin SemVer (meta ADR 0001) | activo |
| Ente | `ia_nest_core` | El motor: enruta, infiere, itera | - | - | `v0.4.0` | v0.4 entregada: catalogo + `capability.list`, `task.plan`, `routing_rules` retirada |
| Ente | `ia_nest_extended` [1] | La memoria/conocimiento | enriquecimiento (`core ADR 0031`) | `core >=0.4 <0.5` | `v0.1.0` | contrato publicado y activo; las tres pieles (CLI, REST, MCP) sirven el contrato uniforme; F6 (datos web) y F8 (despliegue) pendientes |
| Ente | `ia_nest_core_conscience` | La mente voluntaria | checkpoints (`core ADR 0034`) | core, extended | - | prevista |
| Ente | `ia_nest_core_pulse` | La mente involuntaria | telemetria + perillas (`core ADR 0037`) | core, extended, conscience | - | prevista |
| Ente | `ia_nest_web` | La cara: GUI | contratos publicos | core, extended, conscience, pulse | - | prevista |
| Exterior | `ia_nest_agents` | Agentes que consumen el ente | contratos publicos | core (+ las que use) | - | prevista |
| Exterior | `ia_nest_external_*` | Integraciones que ACTUAN | `tool_contracts` (`core ADR 0007`) | core | - | diferida |
| Exterior | `ia_nest_module_*` | Modulos propios | contratos publicos | core | - | prevista |
| Exterior | Otras entidades IA_NEST | Comunicacion entidad-a-entidad | por definir | - | - | futura |

[1] `core ADR 0033` nombra esta capa `ia_nest_core_extended`. El repo real es
`ia_nest_extended`, y es el nombre que usan el propio registro, el mapa de
`core IA_NEST_CORE_CONTEXT.md` y la capa misma. El ADR conserva su texto: es
historia, y la deriva de nombre se anota aqui en lugar de reescribirlo.

Zona "Ente" = el pack basico con identidad propia simulada (`core ADR 0033`).
Zona "Exterior" = consume el ente por sus contratos publicos y NO dirige su
pensar.

## 4. Lo que NO esta aqui

- **La costura de cada capa con el core**, descrita: vive en
  `core docs/FRONTERAS.md`. Es la promesa del core, vecina de su contrato.
- **Lo que hace cada capa por dentro**: vive en el repo de esa capa. Meta
  gobierna COMO se construye el ente, no que hace cada pieza. El diseno ya
  reconciliado de capas aun no sembradas (conscience, pulse, web) sigue en
  `core docs/FRONTERAS.md` marcado como deuda declarada, y mudara al repo
  correspondiente cuando se siembre.
- **Los concerns del ente sin repo asignado**: `docs/CAPAS_FUTURAS.md`. Cuando
  uno recibe repo, sale de alli y entra aqui como fila "prevista".
- **El backlog del motor** (senales y capacidades que el core debe a otras
  capas): `core docs/CAPAS_FUTURAS.md`.
