# Change Requests (CR) entre capas del ente

Estado: activo (hogar autoritativo)
Version: 1.1 - 2026-07-26

Canal formal para que una capa solicite un cambio en el contrato de otra capa de
la que depende (tipicamente hacia el core). Complementa el grafo de dependencias
por SemVer (core ADR 0032), que define de QUIEN depende cada capa pero no COMO
se pide un cambio de vuelta.

Este es el hogar autoritativo del proceso y de los CR emitidos. Un CR es un
artefacto BILATERAL: no pertenece ni a la capa que lo emite ni a la que lo
decide, sino a las dos. Por eso vive en el taller y no en ninguna capa
(meta ADR 0001).

## Regla de oro

La capa de abajo PROPONE; la capa de arriba DISPONE. La capa origen justifica la
NECESIDAD y sugiere una forma; la capa destino decide su contrato, su forma y su
version. La forma sugerida no es vinculante. Solo el resultado reconciliado por
el usuario se registra (coherente con el modo multi-IA).

## Cuando se usa un CR

Cuando una capa necesita una capacidad o un cambio de contrato de una capa de la
que depende, y NO puede resolverlo en su propia capa. Si se puede resolver en la
propia capa (por ejemplo enriquecimiento sobre el core, via 2), no es un CR.

## Anatomia de un CR

Un fichero `CR-NNNN-<slug>.md` con:

- `id`, `fecha`, `capa origen`, `capa destino`, `estado`.
- Caso de uso motor (por que se necesita, con evidencia).
- Que se pide (la capacidad o cambio).
- Forma sugerida (contrato propuesto; NO vinculante).
- Impacto SemVer previsto en la capa destino y version que necesitaria la origen.
- Alternativas consideradas y por que no bastan.
- Tension doctrinal, si la hay (que ADRs toca).

La numeracion `NNNN` es correlativa POR CAPA ORIGEN. Un CR se cita como
`<origen> CR-NNNN` (por ejemplo `extended CR-0001`), igual que los ADR se citan
como `<repo> ADR NNNN`.

## Estados

    propuesto -> aceptado | reformulado | rechazado -> entregado

## Donde vive cada CR

Arbol por capa origen, con el estado proyectado en el directorio:

    docs/change_requests/
      from-<capa-origen>/
        solicitado/     CR abiertos
        resuelto/       CR cerrados

**El campo `estado` del fichero es autoritativo. El directorio es su proyeccion
gruesa**, y responde a la pregunta "que hay abierto" sin leer los ficheros:

- `solicitado/` mientras el estado sea `propuesto`, `aceptado` o `reformulado`.
- `resuelto/` cuando sea `entregado` o `rechazado`.

Mover el fichero es parte de cerrar el CR. Si directorio y campo divergen, manda
el campo y el fichero esta mal colocado.

El arbol por origen responde "que he pedido yo". La pregunta complementaria
-"que me deben decidir a mi"- la responde el indice de abajo, que lleva ambos
ejes. No hay un segundo arbol por destino: duplicaria el estado.

## Flujo

1. La capa origen redacta el CR en este repo, bajo
   `from-<su-capa>/solicitado/`, estado `propuesto`, y anade su linea al indice.
2. La capa origen NOTIFICA a la capa destino por un canal DURADERO (ver abajo).
3. La capa destino DISPONE: si acepta (o reformula), lo convierte en un ADR
   propio + item de su PLAN, con su forma y su version objetivo. Si rechaza, con
   motivo. El CR refleja el resultado en su campo `estado`.
4. Al entregarse (nueva version publicada de la capa destino), la capa origen
   actualiza su manifiesto de dependencias, marca el CR `entregado` y lo mueve a
   `from-<su-capa>/resuelto/`.

## La notificacion tiene que ser duradera

Lo que se exige es que la capa destino reciba el aviso por algo que SOBREVIVA a
la conversacion en que se produjo. El mecanismo concreto es libre. Valen:

- Un issue en el repo de la capa destino. Es la forma preferente cuando esta
  disponible: vive en el repo de quien decide y es visible sin conocer meta.
- Un brief de handoff en `docs/handoff/`, enlazado desde el indice de abajo.
  Sirve tambien cuando hay contexto que entregar y no solo un aviso.

NO vale como unica notificacion un hilo de conversacion con un agente. Se
evapora, y el siguiente agente que abra la capa destino no encuentra rastro: el
CR se quedaria esperando a alguien que no sabe que le esperan.

Esta regla se escribio pidiendo issue de forma obligatoria. Se relajo al primer
uso real, cuando resulto que la herramienta de issues no siempre esta a mano y
el requisito habria quedado incumplido por sistema. Lo que importaba era la
durabilidad, no el formulario.

## Indice de CR

| id | origen | destino | asunto | estado | version objetivo |
|---|---|---|---|---|---|
| [CR-0001](from-ia_nest_extended/solicitado/CR-0001-checkpoint-enriquecimiento-por-subtarea.md) | `ia_nest_extended` | `ia_nest_core` | checkpoint de enriquecimiento por subtarea en `task.run` | reformulado | core MINOR, sin fijar |

Notificacion a la capa destino:

| id | forma | referencia |
|---|---|---|
| CR-0001 | brief de handoff | [cr_0001_brief_core.md](../handoff/cr_0001_brief_core.md) |
| CR-0001 | respuesta del core | [cr_0001_respuesta_core.md](../handoff/cr_0001_respuesta_core.md) |
