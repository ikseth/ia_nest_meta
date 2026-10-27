# Decision 0014: meta orquesta el plan del ente, en una jerarquia de tres niveles

Fecha: 2026-10-27

## Contexto

La apertura del ente (meta ADR 0013) construyo un borde correcto, pero no lo
que el operador tenia en la cabeza: una infraestructura comun para consumir el
ente por REST y MCP, con un directorio de identidades como punto de partida. La
autenticacion central se perdio entre hilos: meta ADR 0013 no eligio mecanismo
y su criterio de salida no la recogia; `edge ADR 0002` (punto 2) la dejo como
"camino" con disparador; el brief de la primera entrega la dejo fuera.

Nadie lo decidio asi. El plan vivia repartido en el `PLAN.md` de cada capa, en
los handoffs del laboratorio, en los comentarios de Deck y en los ADR de meta,
y ningun hilo veia el conjunto. El operador saltaba de repo en repo y de hilo
en hilo para sostenerlo.

## Decision

### 1. El orden entre capas vive en meta

El plan del ente -que hitos hay, en que orden, que depende de que y con que
criterio se cierra cada uno- vive en `docs/plan/`. Cada capa conserva su
detalle interno (su `PLAN.md`, sus fases, su contrato), pero no fija orden
respecto a otra capa: para eso apunta al hito de meta.

### 2. Tres niveles, de arriba abajo

| Nivel | Donde | Que contiene | Tope |
|---|---|---|---|
| 0. Estrategia | `docs/plan/HOJA_DE_RUTA.md` | la idea final, los hitos con estado y dependencias, enlace a cada hito | unas 80 lineas |
| 1. Hito | `docs/plan/hitos/HNN_<nombre>.md` | objetivo, criterio de salida, pasos (id, que, donde, depende de, estado, tarjeta) | unas 200 lineas |
| 2. Paso | donde se hace el trabajo | si es de una capa: su brief o su `PLAN.md`; si cruza capas: `docs/plan/pasos/HNNPM_<nombre>.md` | sin tope; vive poco |

- **Se lee de arriba abajo y se para en el nivel que haga falta.** Un hilo lee
  `AGENTS.md`, la hoja de ruta, su hito y su paso; nada mas, salvo que el paso
  lo pida. Asi una tarea pequena no carga el plan entero.
- **Un detalle vive en un solo sitio.** Si el detalle de un paso esta en el repo
  de una capa, meta lo enlaza; no lo copia.
- **Hito cerrado, hito archivado.** Pasa a `docs/plan/hitos/cerrados/` y en la
  hoja de ruta queda en una linea. Lo activo se mantiene pequeno.
- **Un hito se escribe cuando se abre**, no antes. Mientras tanto es una fila de
  la hoja de ruta.

### 3. Identificadores estables

Hito `HNN` (`H03`) y paso `HNNPM` (`H03P2`). Van en el titulo de la tarjeta de
Deck, en los mensajes de commit, en los briefs y en los handoffs. Un `grep` del
identificador en todos los repos encuentra todo lo de ese paso, y su fichero da
el contexto. No se reutilizan.

### 4. Los hilos salen de meta, uno por hito

- El trabajo del ente se conduce desde hilos abiertos en `ia_nest_meta`, uno por
  hito (no uno por repo). Cada hilo nace con un handoff que apunta a la hoja de
  ruta y a su hito.
- Desde un hilo de meta se trabaja en los repos de capa: se disena con el
  usuario, se escriben briefs, se lanza al codificador en el repo de la capa y
  se verifica. **El codigo sigue entrando por PR en el repo de su capa**; meta
  no tiene codigo.
- Antes de tocar una capa, el hilo lee el `AGENTS.md` de esa capa: sus reglas
  siguen mandando dentro de ella.
- Un hilo propio de una capa solo se abre para trabajo largo de esa capa sola
  (por ejemplo, medidas de extended). Nace con handoff desde un paso de meta y
  devuelve su resultado a ese paso.

### 5. Deck refleja el plan; no lo sustituye

- Una tarjeta por hito, con su identificador en el titulo. Los pasos van como
  tarjetas propias (`H03P2 ...`) cuando son largos, o en la descripcion del
  hito cuando son cortos.
- Los comentarios registran lo hecho, lo verificado y lo pendiente. Un cambio de
  plan se escribe en el hito; el comentario solo lo anuncia.

## Criterio de salida

El modelo esta implantado cuando, terminada la migracion (hito `H01`):

1. Toda tarjeta abierta de IA_NEST en Deck lleva identificador de hito o de paso.
2. Ningun `PLAN.md` de capa fija orden respecto a otra capa.
3. Un hilo nuevo encuentra su trabajo leyendo solo `AGENTS.md`, la hoja de ruta
   y su hito.
4. La hoja de ruta no pasa de unas 80 lineas y ningun hito activo de unas 200.

## Motivo

- **Un plan que nadie ve entero se rompe en las juntas.** La autenticacion cayo
  en la junta entre meta ADR 0013 y `edge ADR 0002`.
- **Meta ya es el control global**: el registro, los CR y la doctrina viven
  aqui. El orden entre capas es del mismo genero: no es de ninguna capa.
- **Jerarquia y no documento unico**: un fichero que lo contiene todo crece sin
  tope y cada tarea pequena lo carga entero. Un fichero unico con etiquetas para
  `grep` devuelve trozos sin sus dependencias, que es justo lo que se perdio.
  La jerarquia da el contexto justo; los identificadores dan el `grep`.

## Consecuencias

- **El charter de `README.md` cambia en un punto**: "Su PLAN y su estado" se
  queda en cada capa, salvo el orden entre capas, que vive aqui.
- **`AGENTS.md` cambia el mandato**: sigue siendo por concern y meta sigue sin
  tocar el feature de una capa por su cuenta, pero los hilos que conducen ese
  trabajo salen de meta (punto 4).
- **Migracion deliberada, no big-bang** (`AGENTS.md`): hito `H01`. Inventario de
  lo vivo, asignacion a hitos con el usuario, tarjetas reconvertidas (no se
  borran: pasan a Terminado con puntero), `PLAN.md` de capa sin orden entre
  capas, un commit por repo.
- **El criterio de salida de meta ADR 0013 se revisa** en su propio hito, con
  la autenticacion central y el punto REST/MCP comun dentro del criterio.

## Lo que esta decision NO hace

- No mueve codigo, contrato ni ADR de ninguna capa a meta.
- No cambia los roles de `DOCTRINA_MULTI_IA.md`: el usuario reconcilia y
  registra, el disenador disena y verifica, el codificador implementa en ciego.
- No reescribe ningun ADR: los que se toquen reciben un `## Estado posterior`.

## Impacto de version

Ninguno en el contrato publico de ninguna capa. Cambia donde vive el plan y
desde donde se conducen los hilos.
