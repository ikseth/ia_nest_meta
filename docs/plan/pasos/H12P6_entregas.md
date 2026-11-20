# H12P6: CR por capa y orden de entrega

Estado: decidido por el usuario el 2026-11-20 (D1)
Version: 1.0 - 2026-11-20

Paso de `H12`. Convierte lo decidido en H12P2-H12P5 (meta ADR 0017 y 0018,
`H12P3_entidades.md`, `H12P5_saludo.md`) en trabajo de cada capa y fija el
orden de entrega y de medida.

## Punto de partida

Versiones vigentes: core `v0.5.0`, extended `v0.6.5`, borde `v0.2.1`, capa de
Home Assistant `v0.1.0`. La integracion de Home Assistant (`ha-addons`) es un
consumidor de fuera del ente.

Un CR lo redacta la capa que necesita algo de otra de la que depende, y la de
destino lo dispone en su ADR (`docs/change_requests/README.md`). Meta no es una
capa (core ADR 0033) y no es origen de CR. El trabajo propio de una capa,
decidido en meta, entra por brief y por el ADR de esa capa, como edge ADR 0003
desde H05. Los briefs para el codificador se escriben al lanzar cada entrega,
en el laboratorio (`ia_nest_lab`); la fuente de cada decision son los ficheros
de paso de meta.

## D1: tres CR, briefs para el trabajo propio, dos tandas

Decidido por el usuario el 2026-11-20.

**CR**, de la capa que necesita a la capa de la que depende:

| CR | De -> a | Que | Tanda |
|---|---|---|---|
| `extended CR-0007` | extended -> capa de Home Assistant | inventario sin estado y un token que solo lo lea | A |
| `extended CR-0006` | extended -> core | tramo A: contexto del turno con el catalogo, proveedores como techo, efecto lectura u orden (tambien en dominio fijo), interseccion; tramo B: el marco en el contexto, preguntar como resultado declarado | A y B |
| `edge CR-0004` | borde -> extended | transportar cuenta, persona y dispositivo; componer, resolver y guardar el marco en `session_frames` | B |

**Briefs**, para el trabajo propio de cada capa (cada una lo dispone en su ADR):

| Brief | Que | Tanda |
|---|---|---|
| Extended | A: entities (registro con clases, claves externas, observado y curado, `entity_versions`, principal `operator`, `entity.*` por CLI, `entity.apply` en el instalador con `ENTITIES_SEED`) y catalogo (refresco del proveedor, confianza, resumen y preseleccion). B: clases `person`, `device` y `home`, `session_frames` y la respuesta a `edge CR-0004` | A y B |
| Borde | H12P5, D1-D3: clave `device`, los tres hechos con origen, forma del dispositivo con 400, `person_required` con respaldo infantil, auditoria | B (se puede programar durante A) |
| Integracion de Home Assistant | H12P5, D5-D6: tabla de usuario a correo, `X-IANest-Person` y `X-IANest-Device` (solo `device_id`) | B (se puede programar durante A) |
| Laboratorio | A: una declaracion del proveedor (el techo) que leen los dos instaladores, token de inventario, `entities.toml` con la confianza y los alias, luz y ventilador de prueba (permiso del 2026-11-19). B: personas, casa y la tableta comun en la siembra; usuario de la tableta en Home Assistant y en el directorio (con permiso del usuario); clase `home_assistant` con `person_required: false` y la cuenta en `edge_asserts_people` despues del borde; la tabla de la integracion | A y B |

El core y la capa de Home Assistant trabajan desde su CR, sin brief aparte.

**Tanda A, "sabe que hay en la casa y ofrece solo lo que se pide":** capa de
Home Assistant, despues extended (entities base y catalogo), despues core
tramo A, y laboratorio. Se mide: criterio 1 (`marco-v1` por Home Assistant y el
techo), criterio 6 (una entidad virtual nueva), la parte del efecto del
criterio 7 (la frase del incidente) y el 4. Todo al historico (H08P24).

**Tanda B, "sabe con quien habla":** borde e integracion de Home Assistant,
extended (marco), core tramo B y laboratorio. Se mide: criterios 2, 3, 5, el 7
entero y el 4. Con eso se cierra `H12`.

Descartados: todo en una tanda (mucho tiempo sin mejora medible, y un fallo no
se sabe de que pieza es); el marco primero (la linea base dice que pesa mas el
catalogo: techo 6/24); CR con origen en meta (meta no es una capa); un CR por
cada cambio interno (el proceso de CR es para el contrato entre capas).

Criterio falsable: cada consecuencia de meta ADR 0017 y 0018 y de H12P3 y H12P5
aparece en exactamente una pieza (tabla siguiente); al terminar la tanda A se
miden y se anotan el criterio 1 y el techo sin nada de la tanda B desplegado;
ninguna capa cambia su contrato sin un ADR propio.

## Cada consecuencia, en su pieza

| Consecuencia | Origen | Pieza | Tanda |
|---|---|---|---|
| Afirmar `device` y `person` hacia dentro; persona opcional por clase | 0017; H12P5 D1-D3 | brief del borde | B |
| Afirmar la cuenta hacia dentro | H12P3, consecuencias | brief del borde | B |
| Guardar el marco en la sesion, resolver contra entities, componer siempre | 0017, punto 4; H12P3 D6 | `edge CR-0004` | B |
| Pasar el marco al core | 0017, punto 5 | `edge CR-0004` y `extended CR-0006` tramo B | B |
| Clases, listas cerradas, claves externas, observado y curado, historico, `operator`, `entity.*`, siembra | H12P3 D1-D5 | brief de extended (`person`, `device`, `home` en B) | A y B |
| `session_frames` | H12P3 D6 | brief de extended y `edge CR-0004` | B |
| Catalogo en extended, refresco del proveedor, confianza, resumen y preseleccion | 0018, puntos 1-4, D1-D3 | brief de extended | A |
| Leer modelos y dominios del core | 0018, consecuencias | brief de extended (`capability.list`, ya existe) | A |
| Recibir el marco y la composicion como contexto al enrutar y responder | 0017, punto 5; 0018, D4 | `extended CR-0006` (catalogo en A, marco en B) | A y B |
| Proveedores como techo, confianza desde extended | 0018, punto 2 | `extended CR-0006` tramo A | A |
| Efecto lectura u orden, tambien en dominio fijo; interseccion | 0018, D5 | `extended CR-0006` tramo A | A |
| Preguntar como resultado declarado | 0017, punto 7 | `extended CR-0006` tramo B | B |
| Inventario sin estado y token de lectura | 0018, consecuencias | `extended CR-0007` | A |
| Usuario con sesion y dispositivo desde Home Assistant | 0017; H12P5 D5-D6 | brief de la integracion de Home Assistant | B |
| Declaracion del proveedor, siembra, luz y ventilador de prueba | 0018; H12P3 D5 | laboratorio | A |
| Personas, casa, tableta comun, clase y grupo de la cuenta de Home Assistant | H12P3; H12P5 D3 y D5 | laboratorio | B |
| Comprobar que llega el usuario desde la app y la tableta | H12P5 | laboratorio, al desplegar la integracion | B |

## Pendiente de este paso

- Avisar a cada capa de destino por un canal duradero (issue en su repo) y
  anotarlo en la tabla de notificacion del indice de CR. Pendiente de
  aprobacion del usuario.
