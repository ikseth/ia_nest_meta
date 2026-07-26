# Decision 0002: re-hogar las convenciones transversales y la doctrina multi-IA

Fecha: 2026-07-26

## Decision

Se re-hogan a `ia_nest_meta` dos cuerpos de doctrina que hasta hoy vivian en
`ia_nest_core` y estaban replicados por copia en cada capa:

1. **Convenciones transversales** -> `docs/CONVENCIONES_TRANSVERSALES.md`:
   texto de docs en ASCII puro, identificadores y claves en ingles snake_case,
   citas cruzadas `<repo> ADR NNNN`, cabecera de documento normativo, repo
   publico sin datos internos. Origen: `core ADR 0016` y el bloque de
   identificadores de `core docs/CONVENCIONES.md`.
2. **Doctrina multi-IA** -> `docs/DOCTRINA_MULTI_IA.md`: roles, modo ciego,
   regla de la inconsistencia, regla del registro, disciplina de handoff y
   metodo. Origen: seccion "Colaboracion entre varias IA" de
   `core IA_NEST_CORE_CONTEXT.md`.

Se aplica el mecanismo de puntero de meta ADR 0001, en tres movimientos:

- En los documentos VIVOS del core, el cuerpo re-hogado se sustituye por un
  puntero corto.
- A `core ADR 0016` se le APENDIZA una seccion `## Estado posterior`. Su cuerpo,
  su fecha y su motivo quedan intactos.
- Este ADR registra el movimiento, para que el rastro sea bidireccional.

Los `AGENTS.md` y `CLAUDE.md` de core, extended y meta dejan de contener copias
de estas reglas y pasan a apuntar al documento transversal, conservando solo lo
propio de su repo.

## Que NO se mueve

- `core docs/CONVENCIONES.md` conserva su metodo de trabajo: filosofia UNIX,
  cabeceras de script, "una decision por ADR", cuando ficha y cuando ADR. Es
  metodo de esa capa, no obligacion del ecosistema.
- La regla anti-entropia y el criterio nucleo-vs-herramienta se quedan en
  `core IA_NEST_CORE_CONTEXT.md`: definen la frontera del motor.
- SemVer (`core ADR 0030`, `core docs/VERSIONADO.md`) y el registro de capas
  (`core docs/FRONTERAS.md`, `core ADR 0032`) NO se tocan en este paso. Su
  reparto es un paso posterior de la migracion.

## Motivo

La regla de idioma de identificadores no tiene nada que ver con enrutar ni
inferir: es una regla de ecosistema que quedo en el motor por ser el primer
repo. La consecuencia observable es que `ia_nest_extended` cita `core ADR 0016`
para saber como nombrar sus propias variables: una capa mirando dentro de otra
para encontrar doctrina que no es de esa otra.

La doctrina multi-IA estaba peor: definida en `core IA_NEST_CORE_CONTEXT.md` y
copiada, con variaciones, en cinco ficheros (`AGENTS.md` y `CLAUDE.md` del core,
`AGENTS.md` y `CLAUDE.md` de extended, y el `AGENTS.md` de meta). Ya habia
derivado: el `CLAUDE.md` del core incluia la regla de no corregir acentos y el
`AGENTS.md` del mismo repo no. Cinco copias de una regla es una garantia de
divergencia, no una redundancia util.

Se eligio este paso como primero de la migracion por ser el de menor riesgo:
no toca contrato publico de ninguna capa, no cambia comportamiento observable y
es reversible con un revert.

## Consecuencia

- Impacto de version en el core: **ninguno**. No se toca contrato publico. Por
  `core docs/VERSIONADO.md`, un cambio que altera una decision normativa
  corresponde a un ADR y no a una ficha de fix; el ADR es este, y su registro
  del lado del core es el apendice de `core ADR 0016`. Se anade una linea al
  `CHANGELOG.md` del core por trazabilidad, marcada sin impacto de version.
- La forma vigente de la regla de texto (ASCII puro, incluida la enye) queda
  escrita por primera vez. `core ADR 0016` decia "sin tildes"; la practica era
  mas estricta. El documento transversal recoge la forma vigente y lo declara.
- Cualquier capa nueva hereda estas reglas apuntando a un solo documento, sin
  copiarlas y sin citar al core.
