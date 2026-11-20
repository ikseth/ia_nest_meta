# H12P7: tanda A, catalogo y efecto

Estado: D1 decidido por el usuario el 2026-11-20; entrega en curso
Version: 1.0 - 2026-11-20

Paso de `H12`. Entrega y mide la tanda A de `pasos/H12P6_entregas.md`.

## D1: se aparca el conector; la tanda A entrega lo que no depende de el

Decidido por el usuario el 2026-11-20.

### Lo que se encontro

`extended CR-0007` pedia a la capa de Home Assistant un inventario sin estado
con el id y el area (id y nombre) de cada entidad. Comprobado en el Home
Assistant de casa, en solo lectura, con un usuario sin privilegios:

- La lectura de Assist que ya usa la capa (`GetLiveContext`, del servidor MCP
  de Home Assistant) da nombre y alias, dominio, area (nombre y alias), clase y
  unidad, pero **ni id de entidad ni id de area**, y hay nombres repetidos entre
  areas.
- Los ids si estan en los registros de Home Assistant (entidades, areas,
  dispositivos, plantas), que un usuario sin privilegios lee por websocket.

### Que se decide

- **El conector Home Assistant - IA_NEST se aparca en un hito propio, `H13`.**
  El hueco esta en el servidor MCP de Home Assistant, que es el estandar que se
  usa. El camino no es que el ente lo tape: primero se mira que pasa con ese
  MCP en el producto (planes, versiones nuevas), despues soluciones de la
  comunidad, y solo en el peor caso un MCP propio entre Home Assistant e
  IA_NEST, disenado como tal.
- **Nada de lo decidido se reescribe.** H12P3 (D2 y D3) y meta ADR 0018 siguen
  siendo el diseno. Las clases `area` y `appliance` del catalogo se implementan
  cuando haya conector que las alimente.
- **`extended CR-0007` queda en espera** (sigue `propuesto`, con nota de la capa
  origen). Se avisa en el issue de la capa de Home Assistant: no hay nada que
  hacer hasta `H13`.
- **La tanda A se reduce a lo que no depende del conector:**
  - extended: la base de entities (clases, claves externas, observado y curado,
    `entity_versions`, principal `operator`, `entity.*` por CLI, `entity.apply`
    con `ENTITIES_SEED`) y la clase `provider` con el hecho curado `trusted`;
  - core, `extended CR-0006` tramo A: proveedores como techo, confianza desde
    extended, efecto lectura u orden (tambien en dominio fijo) e interseccion;
    sin resumen ni preseleccion de aparatos;
  - laboratorio: la declaracion del proveedor, `entities.toml` con la confianza,
    y la luz y el ventilador de prueba;
  - capa de Home Assistant: sin cambios.
- **Criterio de salida de `H12`:** el criterio 1 y la primera mitad del 6 (un
  recurso nuevo se alcanza sin tocar la configuracion) pasan a `H13`, porque
  dependen del conector. La segunda mitad del 6 (un proveedor no declarado nunca
  se consulta) sigue en `H12`.
- **Tanda B:** el lugar del marco se queda en la casa (de la persona o del
  dispositivo); el area del dispositivo llega con `H13`.
- **Orden:** `H13` va detras de `H12` y antes de `H10` y `H11`, para que el
  criterio 1 no quede sin fecha.

Descartados:

- **Registros por websocket desde la capa.** Da ids, pero abre un segundo canal
  hacia Home Assistant, depende de su API interna y fija una forma del conector
  sin haberlo disenado.
- **Curacion de cada aparato en Home Assistant (alias y exposicion) con clave
  dominio, nombre y area.** Funciona sin websocket, pero es un rodeo al hueco
  del MCP y retocaria H12P3 D2 y D3 para un solo proveedor.
- **Dejar `H12` abierto hasta tener el conector.** Mezcla dos hitos y retrasa
  el marco, que no lo necesita.

Criterio falsable: la tanda A no cambia la capa de Home Assistant; ningun brief
usa una clave de aparato distinta de H12P3 D3; la frase del incidente
(H12P1b), por Luna, sale clasificada como lectura y no ejecuta nada; un
proveedor no declarado nunca se consulta; el criterio 4 no empeora; el
criterio 1 queda en el historico como referencia de antes del conector.

## Pendiente de este paso

- Brief de extended (tanda A reducida) en el laboratorio y su entrega.
- `extended CR-0006` tramo A en el core: requiere aprobacion del usuario.
- Laboratorio y medida: efecto del criterio 7, segunda mitad del 6, el 4, y el
  1 como referencia.
