# Decision 0003: re-hogar el registro de capas y la regla de vinculo

Fecha: 2026-07-26

## Decision

Se re-hoga a `ia_nest_meta/docs/REGISTRO_CAPAS.md` el indice de capas del ente y
su grafo de dependencias, junto con la regla de vinculo entre capas. Origen:
`core docs/FRONTERAS.md` (seccion "Registro de capas") y `core ADR 0032`.

El corte se hace en TRES partes, no en dos, porque `FRONTERAS.md` mezclaba tres
contenidos de duenos distintos:

1. **Que costura expone el core a cada capa.** Es la promesa del core, vecina de
   `CORE_CONTRACT.md`. **Se queda** en `core docs/FRONTERAS.md`.
2. **Quien existe, para que, quien depende de quien y en que version.** Es del
   ente. **Va a meta.**
3. **Que hace cada capa por dentro.** Es de esa capa. **No va a meta ni se
   queda como doctrina del core**: se marca en `core docs/FRONTERAS.md` como
   `[doctrina de capa]`, deuda declarada que mudara al repo de esa capa cuando
   se siembre. Para `ia_nest_extended`, que ya existe, se sustituye por un
   puntero a sus propios documentos.

Cambios de forma sobre la tabla heredada:

- Se anade la zona **Taller** y la fila de `ia_nest_meta`.
- La columna "Estado" se parte en **Version publicada** y **Estado de trabajo**.
- La columna "Costura" pasa de descripcion a REFERENCIA de pocas palabras.
- Se anade la fila `ia_nest_module_*`, presente en `core ADR 0033` y en el mapa
  de `core IA_NEST_CORE_CONTEXT.md` pero ausente de la tabla.
- Se corrigen los valores obsoletos con los datos reales.

Se anaden al registro dos reglas contra la putrefaccion: la fuente de verdad es
el tag y el manifiesto de cada capa (el registro es indice y puede ir por
detras), y actualizar la fila propia es parte de publicar una version. Esto
ultimo anade un paso al proceso de release de cada capa, redactado en su forma
mas debil.

## Motivo

El argumento estructural: la tabla tenia zonas `Ente` y `Exterior`, y
`ia_nest_meta` no cabe en ninguna. Un registro que no puede listar al repo que
lo hospeda esta en el sitio equivocado.

El argumento empirico: la tabla estaba obsoleta. Declaraba el core en `v0.1.0`
cuando su ultimo tag es `v0.2.0`, y a `ia_nest_extended` como "primera capa (en
diseno)" cuando tiene tres fases de memoria implementadas y verificadas. Se
pudrio porque vivia donde no se consulta y porque nadie tenia el deber de
actualizarla en un momento que ocurriese. Mover la tabla sin arreglar eso solo
habria cambiado de sitio el problema; de ahi la regla del deber de
actualizacion.

El tercer contenido -doctrina interna de capas que aun no existen- es el sintoma
original en su forma mas pura: el core, por ser el primero, no solo absorbio
gobernanza sino tambien diseno de terceros. No se resuelve moviendolo a meta,
que caeria en el mismo error a otra escala. Se resuelve marcandolo y dejandolo
donde esta hasta que exista su dueno.

## Alternativa considerada

Aplazar este paso hasta sembrar la tercera capa, por disciplina de arranque
minimo: el registro rinde de verdad con cuatro o cinco repos, y hoy hay dos
reales. Es una postura coherente con el charter.

Se descarta por tres razones: el consumidor no es futuro (la tabla ya se
pudrio); meta ADR 0001 ya declaraba a meta hogar del registro, y dejarlo en el
core mantenia un estado declarado-pero-no-ejercido; y el coste de mover no crece
con el tiempo, pero si la deuda de punteros hacia el sitio equivocado.

## Consecuencia

- Impacto de version en el core: **ninguno**. No se toca contrato publico ni
  comportamiento.
- `core ADR 0032` recibe una seccion `## Estado posterior`: su regla sigue
  vigente y solo cambia el hospedador del indice. Su cuerpo queda intacto.
- `core ADR 0030/0031/0033/0037` NO reciben apendice: sus menciones a
  `FRONTERAS.md` son enunciados historicos, no reglas vivas.
- `core IA_NEST_CORE_CONTEXT.md` afirmaba que "el core solo hospeda el indice";
  la frase queda corregida, porque ya no es cierta.
- `core docs/VERSIONADO.md` incorpora el deber de actualizar la fila propia al
  publicar.
- `ia_nest_extended` reapunta su `DEPENDENCIAS.md` y sus ficheros de agentes,
  que declaraban que "la doctrina del ente vive en el repo del core".
- La deriva de nombre entre `core ADR 0033` (`ia_nest_core_extended`) y el repo
  real (`ia_nest_extended`) se anota como nota al pie del registro. El ADR no se
  reescribe.
