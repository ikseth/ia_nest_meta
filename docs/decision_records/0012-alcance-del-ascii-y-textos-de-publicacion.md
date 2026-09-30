# Decision 0012: alcance del ASCII puro y textos de publicacion

Fecha: 2026-09-30

## Decision

La regla 1 de `CONVENCIONES_TRANSVERSALES.md` (ASCII puro) se acota a lo que
circula entre agentes y maquinas: documentacion tecnica y normativa, codigo,
identificadores y datos de entrada o salida.

Se exceptuan los **textos de publicacion**: los escritos para lectores de fuera
del ente (manifiestos, cartas abiertas, articulos). Se escriben en espanol
correcto, con acentos y enye.

Para que la excepcion no dependa del juicio de cada agente, se declara por
UBICACION: vive solo bajo un directorio `publications/` en la raiz de un repo.
Fuera de ese directorio, la regla 1 aplica sin cambios.

## Motivo

La regla 1 existe para eliminar ruido en diffs, greps, rutas y pipelines entre
varios agentes y varias maquinas. Un texto de publicacion no esta en ese
circuito: no se cita como norma, no alimenta ninguna herramienta y lo leen
personas ajenas al proyecto. Para ese lector, un texto formal en espanol sin
tildes parece un error, y resta credibilidad justo al documento que mas la
necesita.

Aparece al sembrar `ia_nest_core_conscience`, cuyo manifiesto se publica en
ingles y espanol.

## Consecuencia

- `CONVENCIONES_TRANSVERSALES.md` pasa a la version 1.3 con el alcance y la
  excepcion.
- Un agente no quita los acentos de un fichero bajo `publications/`, ni los
  anade fuera de el.
- Un texto de publicacion no es normativo: no manda sobre ningun documento del
  ente. Si contradice a uno normativo, manda el normativo y la contradiccion se
  senala.

## Impacto de version

Ninguno en el contrato publico de ninguna capa. Es convencion de construccion.
