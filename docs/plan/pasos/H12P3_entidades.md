# H12P3: interlocutores como entidades, minimo de entities y siembra

Estado: decidido por el usuario el 2026-11-19, decision a decision (D1-D6)
Version: 1.0 - 2026-11-19

Paso de `H12`. Fija que minimo de `entities` de extended (extended ADR 0004)
hace falta para el marco de la conversacion (meta ADR 0017, D2) y como entra en
el catalogo de recursos (meta ADR 0018). Es diseno: el ADR de extended no se
escribe desde aqui, sale del CR a extended en H12P6. Practica habitual en
`H12P3_practica_habitual_entidades.md`.

## Punto de partida (extended `v0.6.5`)

- La tabla `entities` existe (`kind`, `name`, `aliases`, `profile` jsonb,
  `status`, `version`), declarada y sin uso real.
- Escribir sobrescribe el perfil y sube `version`: no hay historico.
- `write_entity` solo existe en el adaptador de Postgres: no hay capacidad, ni
  CLI, ni principal de operador (`Principal` tiene `extended` y `conscience`).
- Solo se busca por `id`: no hay enlace con identificadores de fuera.
- La tabla `sessions` (extended ADR 0015) no guarda marco.
- El borde (`ResolvedIdentity`) lleva la cuenta, `user_id`, `service` y el
  perfil; no lleva dispositivo.

## D1: tres clases para el marco; el area se cita

Se siembran tres clases, cada una con una lista cerrada de hechos de perfil:

| Clase | Hechos | Para que |
|---|---|---|
| `person` | `home`, `language` | criterio 2: Open WebUI responde sobre la casa de quien habla |
| `device` | `area` (citada), `home`, `shared`, `language` | criterios 1 y 3: la voz y la tableta comun; con `shared`, "persona desconocida en un dispositivo compartido" |
| `home` | `locality` (texto), `timezone`, `provider` | criterio 2: "va a llover?" sin lugar en la frase; `provider` ata las areas descubiertas a su casa |

- El area no se siembra: se cita la descubierta del proveedor (meta ADR 0018,
  D2). Las plantas no entran (la API de Assist no las da).
- Un hecho fuera de la lista hace fallar la siembra. No hay hechos de rol, edad
  ni grupos: el criterio 7 se comprueba al validar, no por buena fe.
- `place` del marco: area del dispositivo; si no, casa del dispositivo; si no,
  casa de la persona; si no, `unknown`.
- Lo decide la clase de la entidad encontrada, no de donde vino la clave: el
  usuario compartido de Home Assistant (la tableta comun, H12P5 a) tiene uid en
  el directorio, pero se siembra como `device` con `shared`, y su marco dice
  persona `unknown`.

Descartados: sembrar las areas (0018 ya lo cerro); perfil libre (una siembra
podria escribir `role: adult`); mas clases sin consumidor (core ADR 0035).
`kind` es texto: una clase nueva es una adicion a la lista.

Criterio falsable: con estas tres clases se calcula el marco de cada entrada
del alcance (Luna en Home Assistant, la tableta comun, un satelite, Open WebUI
por texto o microfono) sin ningun hecho que no este en la tabla; una siembra
con `role` o `groups` hace fallar la instalacion.

## D2: el catalogo dentro de entities; lo observado y lo curado, separados

- Un solo registro: el catalogo entra en `entities` con clases nuevas, con la
  misma version, el mismo historico y la misma interfaz.
- Cada fila tiene dos partes con duenos distintos:
  - **observado**: lo que dice el proveedor (nombre, alias, tipo, clase,
    area). Solo lo escribe el refresco de extended, que es mecanico, como el
    etiquetado de `entity_refs` (extended ADR 0004, punto 2);
  - **curado**: el juicio (alias propios, clase corregida, `offered`,
    `trusted`). Solo lo escribe la interfaz (D5): hoy el operador y la
    siembra, despues conscience.
- La confianza en un proveedor es el hecho curado `trusted` de su fila; la de
  un recurso suelto, `offered`. El proveedor existe en el registro porque esta
  en el techo; el refresco lo crea sin confianza y el operador la da, tambien
  por la siembra. La memoria solo resta: `trusted` fuera del techo no hace que
  el core consulte nada (0018, punto 2).
- Clases del catalogo ahora: `provider`, `area` y `appliance`, las que usan el
  marco y la preseleccion. `tool`, `model` y `domain` tienen la misma forma y
  entran cuando algo en extended las lea; hoy las lista el core en cada turno.

Gana: la cita del area se comprueba en un solo registro; un engrama puede
llevar en `entity_refs` un aparato como una persona; una sola interfaz y un
solo historico.

Descartados: tabla aparte con la misma forma (como Home Assistant, un registro
por clase: duplica version, historico e interfaz, y `entity_refs` no llegaria a
un aparato); un solo dueno para todo el perfil (el refresco pisaria los alias
del operador, o habria que dar de alta cada aparato a mano, contra el criterio
6); la confianza en una tabla de politica aparte.

Criterio falsable: un alias del operador sobrevive a que Home Assistant renombre
el aparato; el refresco no puede escribir un hecho curado ni la interfaz uno
observado; retirar `trusted` por la interfaz deja de ofrecer el proveedor en el
turno siguiente; un aparato que desaparece del proveedor queda retirado con
fecha, no borrado.

## D3: claves externas con espacio de nombres

Una tabla `(namespace, external_id) -> entity_id`, unica por espacio e id (el
patron de `identifiers` de Home Assistant). El id de la entidad es siempre suyo;
una entidad puede tener varias claves.

| Que | Espacio | Id | Quien lo pone |
|---|---|---|---|
| Persona (y usuario compartido) | `directory` | uid | el borde, validado contra el directorio |
| Dispositivo | `account:<uid de la cuenta>` | lo que manda el consumidor (H12P5 fija cual) | el espacio, el borde con la cuenta autenticada; el id, el consumidor |
| Proveedor, area, aparato | `provider:<nombre en el techo>` | id del proveedor (`area_id`, `entity_id`) | el refresco |
| Lo que solo existe en la siembra (la casa) | `seed` | clave del fichero | el operador |

- El espacio del dispositivo sale de la cuenta autenticada, no de la peticion:
  dos consumidores no chocan, y Open WebUI no puede afirmar el satelite de
  Home Assistant.
- La siembra cita por id, no por nombre: renombrar un area en Home Assistant no
  rompe la cita (0018, D2).
- Una clave sin entidad no crea nada (crear es juicio, ADR 0004): el marco la
  guarda tal cual y aparece en una lista para el operador, como la mencion sin
  resolver de ADR 0004.

Descartados: el id de fuera como id de la entidad (la ata a un origen); ids de
dispositivo globales (colisiones y suplantacion); que el consumidor ponga el
espacio (autodeclarado); crear la entidad al ver una clave nueva.

Criterio falsable: el mismo uid por Open WebUI y por Home Assistant da la misma
persona; el mismo texto de dispositivo por los dos da dos claves distintas, y la
de Open WebUI no encuentra la tableta sembrada para Home Assistant; dos
entidades con la misma clave fallan al sembrar; un dispositivo no sembrado deja
su clave en el marco y en la lista, sin crear nada.

## D4: fila actual mas historico de solo anadir

- La fila de `entities` es la version actual, la que se lee en cada turno.
- Al lado, `entity_versions`, de solo anadir: copia completa de cada version
  (clase, nombre, alias, observado, curado, estado), quien la escribio
  (`extended` para el refresco, `operator`, `conscience`), de donde vino (la
  siembra y su version, la CLI, el refresco de un proveedor) y cuando. Las dos
  escrituras van en la misma transaccion.
- Lo que no cambia nada no crea version: reaplicar la siembra o refrescar sin
  cambios no ensucia el historico.
- La sesion apunta a la version que uso (D6).
- Retirar es una version mas con estado retirada; sin borrado fisico
  (extended ADR 0002). Las claves externas guardan tambien cuando se anadieron
  y se retiraron.
- Solo tiempo de escritura: los hechos sembrados son declarados.

Descartados: solo el contador de hoy; bitemporal (Zep y Graphiti, para hechos
extraidos de conversaciones); solo registro de eventos con el estado derivado;
diferencias de antes y despues en vez de copias.

Criterio falsable: cambiar el area de la tableta en la siembra deja dos
versiones con autor y origen; una sesion abierta antes sigue mostrando el area
vieja y la siguiente la nueva (meta ADR 0017, punto 9); aplicar dos veces la
misma siembra o refrescar sin cambios no crea version; ninguna version se borra.

## D5: una interfaz y un fichero de siembra

- **Interfaz**, en el contrato de extended, la misma para el marco y el
  catalogo: `entity.put` (alta o cambio por clave externa, solo hechos
  curados), `entity.retire`, `entity.get`, `entity.list`, `entity.history`,
  `entity.unresolved` (claves sin entidad) y `entity.apply` (un fichero entero,
  con plan previo).
- **Quien escribe**: principal nuevo `operator`, junto a `conscience`, el
  respaldo manual que ya preve extended ADR 0004. La autoridad por capacidad de
  extended ADR 0002 se cumple por partes de la fila: lo observado, solo
  `extended`; lo curado, solo `operator` o `conscience`.
- **Por donde, hoy**: solo la CLI en la maquina de extended. Ni REST ni MCP
  escriben entities: ningun consumidor, ni el borde, cambia un perfil.
  Conscience entrara por la superficie que se decida cuando exista, con la
  misma interfaz (0018, punto 8) y con autenticacion de principal entre capas,
  que hoy no hay.
- **Siembra**: un solo fichero `entities.toml` en la configuracion del
  laboratorio (`ia_nest_lab`), declarado en `setup.conf` como `ENTITIES_SEED`,
  con la misma forma para el marco y para la curacion del catalogo. TOML: se lee
  sin dependencias y admite comentarios.
- **Instalador**: como ya hace con el corpus. Migraciones, refresco del
  catalogo, validacion del fichero entero (clases, lista cerrada, citas, claves
  repetidas, proveedor dentro del techo), plan y aplicacion en una transaccion.
  Cualquier error hace fallar la instalacion nombrando la entrada, sin aplicar
  nada. Sin el proveedor alcanzable, las citas se validan contra el ultimo
  catalogo guardado; si nunca se refresco, falla a la vista.
- **Propiedad**: lo que declara la siembra es de la siembra. Un hecho sembrado
  cambiado en la CLI vuelve a su valor en el siguiente apply (el plan lo
  ensena). Lo que desaparece del fichero se retira solo si lo creo la siembra.

Forma, con nombres de ejemplo:

```toml
[[entity]]
key = "seed:casa"
kind = "home"
name = "Casa"
locality = "Pueblo de ejemplo"
timezone = "Europe/Madrid"
provider = "provider:casa_ha"

[[entity]]
key = "directory:ana"
kind = "person"
home = "seed:casa"
language = "es"

[[entity]]
key = "directory:tableta_comun"        # usuario compartido de HA (D1)
kind = "device"
shared = true
area = "provider:casa_ha/salon"        # area citada por id (D3)
home = "seed:casa"

[[entity]]
key = "provider:casa_ha"               # confianza (D2); debe estar en el techo
kind = "provider"
trusted = true

[[entity]]
key = "provider:casa_ha/sensor.produccion_fv"
kind = "appliance"
aliases = ["placas", "paneles solares"]
```

Descartados: dos ficheros (misma forma y mismo registro; uno evita que una cita
cruzada dependa del orden); escribir por REST o MCP ya (extended no autentica
principales entre capas); una siembra que no retira (el fichero dejaria de
describir el estado: paso a mano escondido, criterio 8); que la CLI gane a la
siembra (el apply dejaria de ser idempotente).

Criterio falsable: una instalacion desde cero deja las entidades sembradas sin
paso a mano (criterio 8); repetirla da plan vacio y ninguna version nueva;
quitar una entidad sembrada la retira y una creada en la CLI sigue igual; un
fichero con un error falla nombrando la entrada sin aplicar nada; una escritura
por REST o MCP se rechaza; un sustituto de prueba de conscience da de alta,
cambia y retira por la misma interfaz.

## D6: el marco de la sesion

- Tabla `session_frames`, de solo anadir, colgada de la sesion (usuario y
  sesion, extended ADR 0015). `seq 0` es el marco de apertura, escrito en el
  primer turno que extended ve de esa sesion. Cada cambio anade una copia
  completa con orden, fecha, partes cambiadas y causa: `edge` (cambio un hecho
  afirmado) o `dialog` (lo completo una respuesta, meta ADR 0017, punto 7). Sin
  cambios no se escribe.
- Cada parte tiene un estado declarado, nunca `null` ni ausente (extended
  ADR 0015, punto 3):

  | Estado | Que lleva |
  |---|---|
  | `known` | clave, entidad y version |
  | `unresolved` | clave sin entidad (D3) |
  | `stated` | lo dijo el interlocutor, con el turno |
  | `unknown` | nada mas |

  Y su origen: validado por el borde, afirmado por la cuenta sin validar
  (meta ADR 0017, punto 2), deducido de una entidad o dicho en la conversacion.
- `place` guarda su camino (area y casa, y si vino del dispositivo o de la
  persona).
- `missing` es la lista de partes en `unknown` o `unresolved`. Lo `known` o
  `stated` no se vuelve a preguntar.
- Las entidades se resuelven al abrir: dentro de la sesion se usa la version de
  la apertura.
- Se archiva con la sesion y va en el paquete que un hilo archivado deja a
  conscience (extended ADR 0002, enmienda, punto 5). No es memoria del usuario:
  nunca se escribe en `entities` ni en engramas (0017, punto 4).
- Como la sesion es usuario y sesion, si el borde afirma otra persona ya es otra
  sesion. Dentro de una sesion cambian el dispositivo, lo que completa el
  dialogo y la persona dicha cuando la cuenta no trae persona (la voz, el
  usuario compartido). Una persona `stated` orienta y nunca concede.

Descartados: columnas del marco sobrescritas en la sesion; solo los cambios sin
copia; `null` o ausencia para "no se sabe"; juntar `unresolved` y `stated` en
`unknown` (se perderia que sembrar y se volveria a preguntar).

Criterio falsable: el 100 % de las sesiones de Open WebUI y de Home Assistant
tiene `seq 0` (criterio 5), contado con una consulta; ninguna parte a `null` ni
ausente; una respuesta que completa el lugar anade una copia con causa `dialog`
y no se vuelve a preguntar en esa sesion; un turno sin cambios no escribe; una
persona `stated` no cambia `tools_offered` (criterio 7).

## Consecuencias para los pasos siguientes

- **H12P5**: el id de dispositivo que manda cada consumidor (en Home Assistant,
  satelite o dispositivo de `ConversationInput`) va en el espacio de su cuenta;
  el usuario compartido de Home Assistant se siembra como `device` con
  `shared`.
- **H12P6, CR a extended**: clases y listas cerradas, claves externas,
  observado y curado, `entity_versions`, principal `operator`, capacidades
  `entity.*` solo por CLI, `entity.apply` en el instalador con `ENTITIES_SEED`,
  y `session_frames`. Toca lo decidido en extended ADR 0002 (autoridad por
  partes de la fila), 0004 (operador como escritor, historico) y 0015 (marco en
  la sesion): lo dispone extended en su ADR, a partir del CR.
- **H12P6, CR al borde**: afirmar hacia dentro la cuenta autenticada (no solo la
  clase en `service`), porque es el espacio de nombres del dispositivo; dos
  cuentas de la misma clase no deben compartirlo.
- **Laboratorio**: `entities.toml` y `ENTITIES_SEED` en `ia_nest_lab`, con las
  personas, la tableta comun, la casa y la confianza en el proveedor.
- Nada para el core en este paso.
