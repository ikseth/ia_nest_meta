# H12P7: tanda A, catalogo y efecto

Estado: D1 decidido y tanda A entregada y medida el 2026-11-20
Version: 1.2 - 2026-11-20

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

## Entrega (2026-11-20)

- Extended `v0.7.0` (extended ADR 0016, ia_nest_extended#104 y #105): registro de entidades con
  la clase `provider` y el hecho curado `trusted`, historico de solo anadir, principal
  `operator`, `entity.*` por la CLI, siembra (`ENTITIES_SEED`) y techo (`PROVIDER_CEILING`) en el
  instalador; manda `turn_context.trusted_providers` al core solo si el core lo declara.
- Core `v0.6.0` (core ADR 0054, ia_nest_core#58 y #59): dispone `extended CR-0006` tramo A.
  `tool_providers` es el techo y se consulta la interseccion con lo confiado; todo turno con
  herramientas clasifica el efecto (`read` u `order`) con el modelo del enrutador, en la duda
  `read`, y con `read` solo pasan herramientas de solo lectura; `tool_provider.list`.
- La capa de Home Assistant no cambia. Codex programo extended; el core lo programo Claude (Codex
  llego al limite de uso de su cuenta).

## Medida (2026-11-20; evidencia e historico en `ia_nest_lab`)

Con permiso del usuario: lectura, y acciones solo sobre la luz y el ventilador de prueba
(recreados para medir y retirados al terminar), con el modo de pruebas de la casa.

- Efecto, por el techo (dominio `domotica`, perfil `child`): "enciende la luz de prueba al 50 por
  ciento" -> `order`, `light_set`, encendida; "esta encendida la luz de prueba?" -> `read`, solo
  herramientas de lectura; "apaga la luz de prueba" -> `order`, apagada. Clasificar cuesta
  1,0-1,4 s.
- Frase del incidente (H12P1b), por el techo y por Luna (3 vueltas): siempre `read`,
  `kodi_recomendar` no se ofrece, ninguna accion. La lectura falla (el modelo no sabe que entidad
  pedir: es el hueco del catalogo, `H13`) y contesta que no puede comprobarlo, sin inventar.
- Confianza: extended manda `trusted_providers: ["home_assistant"]` y el core la aplica.
- Criterio 4, v1-ha: 55/60, p90 16,3 s frente a 60/60 y 11,9 s del 2026-11-19. Las 20 frases no
  usan herramientas: la tanda A no actua en ellas. Mismo camino y mismos cambios de modelo que el
  19; cada carga de modelo tarda 7,3 s de media frente a 3,3 s, con tron con ~4 GB menos libres.
  Causa del entorno, no del codigo; se vuelve a medir cuando cambie el estado de tron.
- Criterio 1 (`marco-v1`, referencia de antes del conector): no se paso, no cabia en la ventana.

### Remedida (2026-11-20, tarde)

Se bajo la memoria de tres maquinas virtuales del servidor de modelos que no la usaban, y con
16 GiB disponibles se paso de nuevo, con el modo de pruebas de la casa:

- Criterio 4, v1-ha: 60/60, mediana 11,7 s y p90 13,0 s (frente a 11,4 y 11,9 del 2026-11-19),
  sin avisos de fallo. Confirma que el 55/60 era del entorno.
- `marco-v1`, 3 vueltas, como referencia del criterio 1 para `H13`: igual que la linea base del
  2026-11-19. Grupo A por Luna 0/24 y por Open WebUI 0/24; techo 8/24 (6/24 el 19); B 6/6; C 0/6;
  D 24/24; E 18/18. Los 45 turnos de `domotica` salen `read`; solo se usa la lectura de la casa y la
  salvaguarda no salta. B y C se mueven por la redaccion del modelo, sin herramientas de por medio.
  La tanda A no tenia que mover A: falta el catalogo, que es `H13`.

## Pendiente de este paso

Nada. La referencia de `marco-v1` para `H13` es la de la remedida.
