# Convenciones transversales del ente

Estado: activo
Version: 1.1 - 2026-08-06

Reglas que aplican a TODOS los repos del ente y a este taller. Cada repo puede
tener convenciones propias (estilo de codigo, cabeceras de script, cuando ADR y
cuando ficha); esas se quedan en su repo. Aqui solo lo que, de vivir en una
capa, obligaria a las demas a citarla.

Origen historico: `core ADR 0016`. La regla se re-hoga aqui porque no es del
motor: gobierna a todo el ecosistema (meta ADR 0002).

## 1. Texto de la documentacion: ASCII puro

Los documentos se escriben en espanol, en **ASCII puro**: sin acentos, sin
dieresis y sin `n` con virgulilla. Se escribe `diseno`, `pequeno`, `dueno`,
`ano`, `codigo`, `version`.

No es una errata a corregir: es convencion deliberada del proyecto. Un agente
que "arregle" los acentos esta rompiendo la convencion, no mejorandola.

Motivo: elimina una clase entera de ruido en diffs, greps, rutas y pipelines
entre varios agentes y varias maquinas, sin coste de comprension.

Precision honesta: `core ADR 0016` fijo "prosa en espanol sin tildes". La
practica posterior en todos los repos ha sido mas estricta (ASCII puro, incluida
la enye). Este documento recoge la forma vigente; el ADR conserva su texto
original como registro de cuando se decidio.

## 2. Identificadores y claves: ingles snake_case

Los identificadores de codigo y las claves de configuracion van en **ingles
snake_case**: `description`, `preferred_model`, `fallback_models`,
`routing_rules`, `status`, `user_id`, `session_id`.

Motivo: no mezclar espanol con Python y sus librerias (en ingles), y evitar
choques de idioma dentro de una misma linea.

La prosa que rodea al identificador sigue en espanol (regla 1). Las dos reglas
conviven: se documenta en espanol un `preferred_model`.

## 3. Citas cruzadas entre repos

La numeracion de ADR es **por repo** y colisiona: existe un `0001` en el core,
otro en `ia_nest_extended` y otro en `ia_nest_meta`. Por tanto:

- Un ADR se cita como `<repo> ADR NNNN`. Ejemplos: `core ADR 0032`,
  `extended ADR 0002`, `meta ADR 0001`. Nunca `ADR NNNN` a secas.
- Un Change Request se cita como `<capa origen> CR-NNNN`, porque su numeracion
  tambien es por capa origen. Ejemplo: `extended CR-0001`.
- Una ficha de correccion se cita con su repo y su linea de version. Ejemplo:
  `core ficha v0.2/0002`.

El nombre de repo puede abreviarse quitando el prefijo `ia_nest_` cuando el
contexto es inequivoco (`core`, `extended`, `meta`).

## 4. Cabecera de los documentos normativos

Todo documento normativo abre con dos lineas:

    Estado: <semilla | propuesta | activo | superado>
    Version: <X.Y> - <AAAA-MM-DD>

Sirve para saber, sin leer el cuerpo, si un documento manda o solo describe. Es
independiente del SemVer del contrato publico de cada capa.

## 5. Repo publico y frontera de lo versionado

Todos los repos del ente son publicos, incluido este taller. Regla completa en
meta ADR 0006.

**Se versiona** el producto y su documentacion normativa: codigo, tests,
contratos, ADR, planes y plantillas de ejemplo. Nada mas.

**No se versiona nunca**: salida de ejecucion (logs, telemetria, trazas);
temporales y artefactos de build; configuracion efectiva; secretos y entorno;
contexto de la maquina o de la instalacion (IPs, hostnames, nombres de equipos,
arquitecturas de prueba, informes de laboratorio, rutas locales); datos
personales; y los ajustes de las herramientas de IA.

**Patron de la plantilla**, para configuracion y secretos: se versiona el
ejemplo, se ignora el efectivo, y el nombre lo declara
(`config/core.example.yaml` frente a `config/core.yaml`; `.env.example` frente
a `.env`). Asi un tercero instala el repo sin aprender nada del entorno de
quien lo desarrolla.

**Donde vive lo excluido**, con dueno distinto y sin mezclarse:

- `local/`: contexto operativo privado del repo. Es del usuario y NEUTRAL entre
  agentes; ahi va tambien el material compartido entre varios agentes. Se
  ignora la raiz entera (`/local/`), para que nada de lo que cuelgue se escape
  por olvido.
- `.claude/`, `.codex/` y equivalentes: ajustes de las herramientas de IA. Son
  de la herramienta, no del repo, y no se usan como almacen de contexto.

**Obligacion al sembrar un repo**: su `.gitignore` cubre como minimo `/local/`,
las carpetas de herramientas de IA, temporales y artefactos de build,
configuracion efectiva y secretos.

## Fuera de este documento (todavia)

- **SemVer y el contrato de vinculo entre capas**: repartidos ya. La politica
  vive en `POLITICA_SEMVER.md` y la regla de vinculo en `REGISTRO_CAPAS.md`
  (meta ADR 0003 y 0004).
- **Metodo de trabajo del core** (filosofia UNIX, cabeceras de script, cuando
  ADR y cuando ficha, anti-entropia): se queda en `core docs/CONVENCIONES.md`.
  Es metodo de esa capa, no obligacion del ecosistema.
