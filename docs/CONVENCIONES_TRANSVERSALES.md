# Convenciones transversales del ente

Estado: activo
Version: 1.0 - 2026-07-26

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

## 5. Repo publico

Todos los repos del ente son publicos. Nunca se commitean IPs, hostnames, rutas
de maquinas concretas, credenciales ni secretos. El contexto local de cada
maquina se mantiene fuera del control de versiones.

## Fuera de este documento (todavia)

- **SemVer y el contrato de vinculo entre capas**: repartidos ya. La politica
  vive en `POLITICA_SEMVER.md` y la regla de vinculo en `REGISTRO_CAPAS.md`
  (meta ADR 0003 y 0004).
- **Metodo de trabajo del core** (filosofia UNIX, cabeceras de script, cuando
  ADR y cuando ficha, anti-entropia): se queda en `core docs/CONVENCIONES.md`.
  Es metodo de esa capa, no obligacion del ecosistema.
