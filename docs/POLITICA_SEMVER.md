# Politica de versionado del ente (SemVer)

Estado: activo
Version: 1.0 - 2026-07-26

Regla comun a todas las capas del ente. El objetivo es que cualquiera -usuario,
agente de IA o capa consumidora- pueda saber POR EL NUMERO si un cambio rompe
contrato.

Origen historico: `core ADR 0030` y `core docs/VERSIONADO.md`, donde vivio hasta
meta ADR 0004. Cada capa conserva en su repo lo unico que no es comun: QUE
cuenta como su contrato publico.

## 1. Esquema

Versionado semantico `MAJOR.MINOR.PATCH`. Los tags de git son `vMAJOR.MINOR.PATCH`
(por ejemplo `v0.1.0`).

El TAG es la fuente de verdad. El manifiesto de version de la capa
(`pyproject.toml` u otro segun su stack) SIEMPRE refleja el ultimo tag.

## 2. Que gobierna la version

La version de una capa se mide contra su CONTRATO PUBLICO, no contra su codigo
interno. Un refactor interno que no cambia nada observable desde fuera no sube
MINOR ni MAJOR; a lo sumo PATCH si corrige un fallo observable.

**Cada capa declara en su repo que cuenta como su contrato publico.** Es lo
unico de esta politica que no puede ser comun, porque depende de que expone cada
una. Ejemplo: `core docs/VERSIONADO.md` lista sus capacidades, esquema de
configuracion, esquema de telemetria, tipos de evento, taxonomia de error y
version de protocolo MCP.

Una capa sin esa declaracion no puede versionarse con esta politica: escribirla
es parte de sembrar el repo.

## 3. Que numero subir

**Pre-1.0 (serie `0.y.z`).** La capa aun puede cambiar contrato mientras se
estabiliza:

- Cambio que ROMPE contrato (quitar o renombrar una capacidad, cambio no aditivo
  de un esquema, cambio incompatible de tipos de evento o de taxonomia de
  error): sube **MINOR** (`0.1.0 -> 0.2.0`).
- Adicion compatible (capacidad opcional nueva, campo aditivo, bandera nueva) o
  correccion: sube **PATCH** (`0.1.0 -> 0.1.1`).

**Post-1.0.** SemVer estandar: MAJOR rompe contrato; MINOR anade capacidad
compatible; PATCH corrige de forma compatible.

El salto a `1.0.0` es una decision explicita del usuario, con su ADR en la capa
correspondiente. No es automatico.

## 4. Proceso

1. Toda propuesta que toque contrato publico declara su impacto (`patch` /
   `minor` / `major`) en el commit o PR, y anade una linea a la seccion
   `## [No publicado]` del `CHANGELOG.md` de su capa.
2. Los cambios que no tocan contrato (docs, tests, refactor interno) no exigen
   entrada de CHANGELOG, salvo que el autor lo vea util.
3. Publicar una version:
   - mover las entradas de `[No publicado]` a una seccion `[vX.Y.Z] - FECHA`;
   - fijar la version en el manifiesto de la capa;
   - commit y tag anotado `vX.Y.Z`;
   - push con tags;
   - actualizar la fila de la capa en `REGISTRO_CAPAS.md`.

Quien decide el numero: **el usuario, en la reconciliacion.** Un agente propone
el impacto y no corta tags por su cuenta (regla del registro,
`DOCTRINA_MULTI_IA.md`).

## 5. Dependencias entre capas

Una capa fija por rango SemVer la version de la que depende y lo declara en su
manifiesto. Cuando necesita un cambio en el contrato de otra, el canal es un
Change Request. Ver `REGISTRO_CAPAS.md` y `change_requests/`.

## Lo que decide cada capa por su cuenta

- Que cuenta como su contrato publico (seccion 2).
- Que fichero es su manifiesto de version.
- Si lleva un registro de correcciones pequenas y con que forma (el core usa
  fichas en `docs/fixes/`, agrupadas por linea `MAJOR.MINOR`; no es obligatorio
  para las demas capas).
- Su flujo de ramas y PR.
