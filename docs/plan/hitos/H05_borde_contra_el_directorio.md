# H05: borde contra el directorio y punto REST/MCP

Estado: abierto
Abierto: 2026-10-27

## Objetivo

El borde resuelve todas las identidades en el directorio de IA_NEST (`H04`) y
ofrece, ademas de la piel OpenAI, un punto REST/MCP comun para servicios y
agentes (meta ADR 0015, punto 1). Su configuracion deja de contener personas y
credenciales de consumidor.

Decidido antes de abrir (no se reabre):

- El directorio vive en la pasarela, solo LDAPS con la raiz del laboratorio; el
  borde lo lee con `svc_edge` (solo lectura). Entra quien esta en
  `ia_nest_users`; el perfil es `child` salvo que solo este en `profile_adult`.
  El `uid` es estable y no se reutiliza (H04P3, H04P4).
- Fallo cerrado (C4): sin directorio, 503 a todos y queda en la auditoria; ni
  fichero ni cache como fuente alternativa. El operador conserva extended por
  loopback (meta ADR 0015, punto 3).
- El codigo entra por PR en `ia_nest_edge`, escrito por el codificador lanzado
  desde este hilo; la configuracion de laboratorio, por el instalador del
  borde.

Hereda de meta ADR 0015: C2, C4 y C5 enteros y la parte de C1 que toca al
borde. Del inventario (`H01P1`): A10 (personas y claves en el fichero del
borde) y A11 (reenvio REST/MCP). De `H04`: con la pasarela parada, 503 y
auditoria; al volver, todo entra sin tocar nada.

No es de `H05`: el login de Open WebUI contra el directorio y sus cuentas
locales (A09, C3) ni el alta de Home Assistant, Kodi y el agente Linux (C6):
son `H07`.

## Criterio de salida

Decidido por el usuario el 2026-10-27.

1. **Sin identidades en la configuracion (C2).** La configuracion efectiva del
   borde en el laboratorio no contiene personas, claves ni hashes de
   consumidor. La unica credencial que usa es la del propio borde
   (`svc_edge`), fuera del fichero y referida por ruta.
2. **Alta, cambio y baja en un solo sitio (C1, parte del borde).** Sin tocar ni
   reiniciar el borde: una persona dada de alta en el directorio entra por
   Open WebUI con su perfil en la auditoria; cambiarle el grupo de perfil cambia
   el perfil de su siguiente peticion; en los dos grupos o en ninguno, `child`;
   dada de baja, o fuera de `ia_nest_users`, su siguiente peticion se rechaza.
3. **Punto REST/MCP (C5).** Un cliente MCP generico, con una cuenta de servicio
   del directorio, lista herramientas y llama a una capacidad de extended a
   traves del borde; la misma capacidad responde por REST. Sin credencial, con
   una credencial erronea o con una cuenta dada de baja: 401. La identidad que
   llega a extended es la que resolvio el borde, aunque el cliente mande otra.
4. **Contrato uniforme (meta ADR 0007).** Una capacidad de extended que el
   borde no nombra en su codigo es alcanzable por REST y aparece en MCP sin
   tocar el codigo del borde: como mucho, una linea de su lista de permitidas
   (H05P6). Una capacidad fuera de la lista no aparece en MCP y por REST se
   rechaza (prueba en el repo del borde).
5. **Perimetro (C5).** Core y extended siguen sin responder desde la red de
   servicios ni desde la critica; solo el borde.
6. **Fallo cerrado (C4, prueba heredada de `H04`).** Con la pasarela parada,
   toda peticion al borde (piel OpenAI, REST y MCP, con credencial valida) se
   rechaza con 503 y queda en la auditoria. Al volver, todo entra sin tocar
   nada. Igual si el borde arranca con la pasarela parada. Durante la parada,
   el operador llega a extended por loopback.
7. **Sin regresion.** Open WebUI sigue conversando con el ente por el borde,
   con una persona adulta y una infantil de prueba, y los criterios 2-4 del
   `README.md` del borde siguen pasando.
8. **Ningun paso a mano** que el instalador del borde deberia hacer.

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H05P1 | Criterio de salida | este hilo | - | decidido: con la baja o fuera de `ia_nest_users` hay rechazo, no invitado (si no, una baja seguiria entrando y C1 no se cumple) |
| H05P2 | Como se autentican servicios, agentes y consumidores contra el borde (C1 lo dejo abierto) | este hilo | H05P1 | decidido: clave de API por cuenta del directorio, no su contrasena. El consumidor manda `Bearer <uid>.<secreto>`; el borde lee con `svc_edge` la cuenta, el hash del secreto (atributo de la cuenta) y sus grupos, y compara; una busqueda por peticion, sin cache. Una clave por cuenta: un dispositivo con clave propia es una cuenta propia. La contrasena (login, bind) y la clave del borde son credenciales separadas; la contrasena nunca llega al borde. Las personas no tienen clave en `H05`. `directorio.sh` genera y rota la clave. Motivo: es la practica habitual de claves de API (identificador, secreto aleatorio, solo el hash, revocable); usar la contrasena como Bearer reutiliza un secreto y obliga a un bind lento por peticion. OAuth 2.1/OIDC (lo que pide MCP para servidores remotos) queda como camino: otro adaptador en el punto unico de resolucion, sin tocar a los clientes con clave, que seguiran existiendo (HA y Kodi solo mandan una clave fija). Pendiente de comprobar en el laboratorio: lldap guarda y devuelve el atributo |
| H05P3 | Personas y perfil: como se resuelve la persona afirmada y que pasa si no esta en el directorio | este hilo | H05P2 | decidido: una cuenta de `ia_nest_services` que no afirma personas se atiende como ella misma (`user_id` = su `uid`); quien puede afirmar personas lo dice el grupo `edge_asserts_people` del directorio, no el fichero del borde. La persona afirmada se busca por `mail` y su `uid` es el `user_id`. Perfil, para personas y cuentas: `child` salvo que solo este en `profile_adult`. Persona ausente, de baja o fuera de `ia_nest_users`, o cabecera de persona que falta: 403 y auditoria, sin invitado. Clave ausente, erronea o de cuenta fuera de `ia_nest_services`: 401. Sin directorio: 503 y auditoria. Coste aceptado: la memoria de extended con los `user_id` del fichero queda huerfana; es de laboratorio y se trata mas adelante |
| H05P4 | Open WebUI entre `H05` y `H07`: como convive con el borde nuevo | este hilo | H05P3 | decidido: cuenta `svc_open_webui` (en `ia_nest_services` y `edge_asserts_people`) con su clave del borde en la conexion de Open WebUI; sus cuentas locales usan el email del directorio; el admin local queda como admin de emergencia y no conversa (el borde le da 403), como pide C3 en `H07`; un script del laboratorio lo deja hecho por la API de Open WebUI. Hasta `H07`, la baja en el directorio basta para cortar; el alta necesita directorio y cuenta local (estado de laboratorio, meta ADR 0015). Descartado adelantar el login por el directorio: es `H07` (A09, C3) |
| H05P5 | Forma del punto REST/MCP, dentro del contrato uniforme | este hilo | H05P2 | decidido: tres caras con una sola via hacia extended, su REST en loopback. `/v1/` (piel OpenAI, como hoy); `/rest/<ruta de extended>`, reenvio generico de ruta y cuerpo, flujos evento a evento y errores ajenos tal cual (meta ADR 0009); `/mcp` por Streamable HTTP, con herramientas derivadas de `capability.list` de extended y cada llamada por la misma ruta REST; las capacidades de flujo quedan fuera de MCP, como en extended. En las tres el borde pone el campo `identity`; lo que mande el cliente se ignora y queda en la auditoria. Descartado reenviar al MCP propio de extended: otro proceso, otra puerta y la identidad dentro de los mensajes MCP |
| H05P6 | Que queda en la configuracion del borde (A10) | este hilo | H05P2-P5 | decidido. Por REST y MCP solo pasan las capacidades de una lista de permitidas de la configuracion, una sola para todas las cuentas en `H05`: conversar (`prompt.*`, `reasoning.*`, `task.run`), lo propio de la identidad (`session.*`, `memory.recall`) y leer el estado (`capability.list`, `knowledge.status`, `runtime.health`). Lo que cambia estado compartido (`knowledge.ingest/confirm/reject`, `memory.write/consolidate/maintain`, `memory_type.*`, `config.validate`) queda para el operador por loopback. MCP lista solo las permitidas. Motivo: fallo cerrado; una capacidad nueva del core no se expone sin decidirlo. No rompe meta ADR 0007: el reenvio sigue siendo generico y la lista es autorizacion, no catalogo. Otra lista por grupo del directorio, cuando un consumidor la pida. El directorio dice quien y el fichero dice como. En el directorio: cuentas, hash de su clave, alta y baja, perfil, la clase de cada cuenta (`edge_class_<nombre>`), quien afirma personas (`edge_asserts_people`) y quien ve las entradas prototipo del menu (`edge_prototypes`, hoy solo el operador; edge ADR 0002, punto 5). En el fichero: conexion al directorio (con la ruta de la clave de `svc_edge`), clases de consumidor (cabeceras, menu por perfil, si escribe memoria), lista de permitidas, servidor, TLS, extended y auditoria. Una cuenta pertenece a exactamente una clase; ninguna o dos: 403 y auditoria. En `H05`, clases `open_webui` y `api`; las de HA, Kodi y el agente, en `H07` (una entrada en el fichero y un grupo, sin codigo: C6) |
| H05P7 | Brief, codigo por el codificador y PR en `ia_nest_edge` | edge | H05P6 | pendiente |
| H05P8 | Montaje en el laboratorio por el instalador del borde; evidencia en `ia_nest_lab` | lab | H05P7 | pendiente |
| H05P9 | Arquitectura y hoja de ruta al dia; PR | meta | H05P8 | pendiente |

## Reglas del hito

- Una decision cada vez, con diagrama y explicacion breve; primero la solucion
  corta; criterio falsable antes que mecanismo.
- En meta, papeles y motivos; nombres de maquina, direcciones y evidencia, en
  `ia_nest_lab`.
