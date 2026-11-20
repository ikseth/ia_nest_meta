# CR-0007: la capa de Home Assistant publica el inventario de la casa, sin estado

- id: CR-0007
- fecha: 2026-11-20
- capa origen: `ia_nest_extended`
- capa destino: `ia_nest_external_home_assistant`
- estado: propuesto
- tanda: A de `H12` (meta H12P6)

## Caso de uso motor

La linea base de `H12` (H12P1b, 2026-11-19): con el dominio fijado en
`domotica`, el modelo acierta 6 de 24 preguntas sobre datos de la casa. No sabe
que hay: la lectura en vivo (`get_live_context`) devuelve demasiados datos, y el
modelo no reintenta con el nombre, dice no tener herramientas para la energia o
se inventa el nombre de la entidad.

Meta ADR 0018 decide que el catalogo de recursos es memoria y vive en extended:
extended lo descubre leyendo al proveedor declarado, lo guarda en entities con
lo observado separado de lo curado (meta H12P3, D2) y compone en cada turno un
resumen por area y clase y una preseleccion de las entradas que tocan a la
peticion (0018, D3). Para descubrir, extended necesita leer QUE hay en la casa,
sin estado y sin poder actuar.

## Que se pide

1. **Publicar el inventario** de lo que Home Assistant expone a Assist: por
   cada entidad, su id, nombre, alias, tipo (dominio), clase y area (id y
   nombre). **Sin estado**: el estado se sigue leyendo en vivo en el turno
   (0018, "el estado se lee en vivo"). Las areas, con su id y nombre (0018, D2:
   las areas salen del proveedor).
2. **Un segundo token que solo lea el inventario.** Con el, la capa responde al
   inventario y a nada mas: ni acciones ni lectura de estado. El token del core
   sigue igual. Es el techo de 0018, punto 2: extended se conecta al proveedor
   con un secreto que no puede ordenar nada.
3. **Dato, no instruccion.** Nombres y alias se publican tal cual vienen de
   Home Assistant; extended los trata como dato (0018, punto 7, envenenamiento
   de herramientas).

## Forma sugerida (no vinculante)

- Una herramienta MCP nueva, de solo lectura (`readOnlyHint`), por ejemplo
  `inventory_list`, que devuelve `areas` y `entities` en JSON estructurado.
  Puede salir de la misma lectura de Assist que ya hace la capa, quitando el
  estado.
- La capa la publica en `tools/list` solo para el token de inventario, y ese
  token no ve ninguna otra herramienta. Nuevo fichero de token en el entorno,
  por ejemplo `IANEST_EXTERNAL_HA_INVENTORY_TOKEN_FILE`, opcional: sin el, no
  hay inventario.

## Impacto SemVer

- Capa de Home Assistant: anadir una herramienta es MINOR (external_home_assistant
  ADR 0002, impacto) -> `v0.2.0`.
- Extended necesitaria esa version para refrescar el catalogo (tanda A).

## Alternativas consideradas

- **Extended lee la API de Home Assistant directamente.** El registro completo
  pide un usuario administrador, y la API de Assist con estado es justo lo que
  satura al modelo. Ademas abriria otra puerta a Home Assistant fuera de la capa
  que ya lo presenta.
- **El core pasa a extended lo que lee en `tools/list`.** Las herramientas no
  son el inventario: dicen que se puede hacer, no que hay en la casa.
- **Usar el token del core.** Daria a extended acceso a las acciones; el techo
  de 0018 pide un secreto que solo lea.

## Tension doctrinal

Ninguna con el alcance de la capa: sigue presentando Home Assistant y
clasificando; no decide que se ofrece (eso es del core) ni que se confia (eso
es del catalogo de extended, 0018, D1).
