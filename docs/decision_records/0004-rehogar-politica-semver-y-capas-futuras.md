# Decision 0004: repartir la politica de SemVer y las capas futuras

Fecha: 2026-07-26

## Decision

Ultimo paso de la migracion abierta en meta ADR 0001. Se reparten dos documentos
del core que mezclaban politica de ente con trabajo del motor.

### SemVer

La POLITICA sube a `ia_nest_meta/docs/POLITICA_SEMVER.md`: esquema
`MAJOR.MINOR.PATCH`, el tag como fuente de verdad, la regla de que numero subir
(pre-1.0 y post-1.0), el proceso de publicacion y el principio de que la version
se mide contra el contrato publico y no contra el codigo interno.

Se queda en `core docs/VERSIONADO.md` lo unico que no puede ser comun: **QUE
cuenta como contrato publico del core** (sus capacidades, esquema de
configuracion, esquema de telemetria, tipos de evento D2, taxonomia de error y
version de protocolo MCP) y **su registro de fichas** en `docs/fixes/`, que es
un mecanismo suyo y no una obligacion del ecosistema.

La politica declara explicitamente que cada capa debe escribir esa lista en su
repo, y que sin ella no puede versionarse. Es parte de sembrar un repo.

La seccion "Colaboracion multi-IA" de `core docs/VERSIONADO.md` queda como
puntero: su contenido ya se re-hogo en meta ADR 0002.

### Capas futuras

`core docs/CAPAS_FUTURAS.md` mezclaba tres contenidos de duenos distintos, el
mismo patron que `FRONTERAS.md` en meta ADR 0003:

1. **Concerns del ente sin repo asignado** -> `ia_nest_meta/docs/CAPAS_FUTURAS.md`.
   Hoy solo uno: la comunicacion entidad-a-entidad.
2. **Backlog del core** (senales que debe exponer, capacidades que otra capa le
   pedira, la regla de no hardcodear personalidad en el combiner) -> **se queda**
   en el core. Es trabajo del motor.
3. **Diseno interno de capas previstas** (conscience, pulse) -> se marca
   `[doctrina de capa]` y mudara al repo de esa capa cuando se siembre, igual
   que en `FRONTERAS.md`.

Ademas, lo que ya tiene dueno deja de describirse en el core y pasa a puntero:
memoria avanzada a `ia_nest_extended`, y el mapa de repos a
`REGISTRO_CAPAS.md`.

## Motivo

`core docs/VERSIONADO.md` se abria diciendo "Regla unica para el usuario, Codex
y Claude Code" y `core ADR 0032` ya reutilizaba su disciplina para gobernar el
vinculo ENTRE capas. Era, de hecho, politica de ente escrita dentro de una capa:
`ia_nest_extended` la cumple citando al core, igual que hacia con la regla de
idioma antes de meta ADR 0002. Al mismo tiempo, la mitad del documento (que
cuenta como contrato del core) no es generalizable a ninguna otra capa.

El criterio de corte es el mismo de todo el charter: lo que obliga a las demas
capas a citar al core, sube; lo que solo describe al core, se queda.

`CAPAS_FUTURAS.md` habia quedado desfasado por exito: se escribio cuando pulse,
conscience y la memoria avanzada no tenian repo, y hoy los tres lo tienen. Lo
que era un registro de concerns sin dueno se habia convertido en tres cosas a la
vez.

## Consecuencia

- Impacto de version en el core: **ninguno**. No se toca contrato publico ni
  comportamiento. Lo que el core considera su contrato publico no cambia: solo
  cambia de vecino la politica que dice como numerarlo.
- `core ADR 0030` recibe una seccion `## Estado posterior`. Su cuerpo queda
  intacto.
- `core docs/VERSIONADO.md` conserva su nombre y sus referencias (nueve
  documentos lo citan). Cambia su contenido, no su direccion.
- Un releaser del core necesita ahora dos documentos para publicar: la politica
  en meta y su lista de contrato en el core. Es el coste aceptado del corte; la
  alternativa -repetir la regla de que numero subir en cada capa- es la
  duplicacion que este repo existe para eliminar.
- Con este paso se agota la tabla de migracion reconciliada en meta ADR 0001.
  Quedan fuera por decision previa las plantillas de brief (diferidas por falta
  de consumidor) y el metodo de trabajo del core (se queda). Meta pasa a
  regimen de uso: crece cuando entra un CR o se siembra una capa.

## Observacion registrada, no corregida

`core docs/CAPAS_FUTURAS.md` afirma que `finish_reason` es una "senal
foundational, hoy inexistente (el adaptador la ignora)". Segun el `CHANGELOG.md`
del core y su ficha `v0.2/0002`, esa senal ya esta implementada. No se corrige
aqui: es backlog del motor, no gobernanza, y en modo ciego una inconsistencia se
senala y no se corrige por inferencia (`DOCTRINA_MULTI_IA.md`).
