# H07: consumidores, uno a uno

Estado: abierto
Abierto: 2026-10-28

## Objetivo

Cada consumidor (Open WebUI, Home Assistant, Kodi, agente Linux) entra al ente
solo con configuracion y una cuenta del directorio, uno a uno, sin tocar el
codigo de los demas ni el del borde (meta ADR 0015, C6). Open WebUI deja sus
cuentas locales: las personas entran con su cuenta del directorio (C1, C3).

Decidido antes de abrir (no se reabre):

- El directorio dice quien y el fichero del borde dice como. Un consumidor
  nuevo es una cuenta de servicio con su clase (`edge_class_<nombre>`) y una
  clase en la configuracion del borde, sin codigo (H05P6). Clave de API por
  cuenta, `Bearer <uid>.<secreto>` (H05P2).
- Las personas llegan por Open WebUI, afirmadas por `svc_open_webui`, sin
  invitado; el fallo cerrado (C4) ya esta probado (H05).
- C3: Open WebUI sin cuentas locales salvo un admin de emergencia, declarado,
  que no conversa (el borde le da 403, H05P4).

Hereda de meta ADR 0015: C3 y C6 enteros y la parte de C1 que toca a Open
WebUI. Del inventario (`H01P1`): A09 (cuentas locales de Open WebUI), A12
(agente Linux por la piel OpenAI o por REST/MCP) y A15 (tareas de fondo de Open
WebUI). De `H05` (notas para `H07`): el paso coyuntural que deja cada alta y
baja de persona tambien en Open WebUI, los modelos visibles para todos y
`write_memory` sin aplicar en `/rest` y `/mcp`.

No es de `H07`: la entrada de la familia (`H08`), L5r y L4b de la puerta de
extended, ni los CR vigilados en la hoja de ruta.

## Criterio de salida

Decidido por el usuario el 2026-10-28. Consumidor a consumidor; cada
consumidor se cierra por separado y en este orden.

Comun a todos (C6):

1. **Solo configuracion.** El consumidor entra con una cuenta de servicio del
   directorio, una clase en la configuracion del borde y su propia
   configuracion. El codigo del borde y el de los demas consumidores no cambia:
   el borde sigue en el mismo tag durante todo el hito. Si un consumidor
   necesitara codigo del borde, no entra en `H07` sin decidirlo antes.
2. **Baja de la cuenta.** Dada de baja la cuenta del consumidor, su siguiente
   peticion recibe 401, sin tocar el borde ni a los demas consumidores.

Open WebUI (C1, C3):

3. **Alta en un solo sitio.** Una persona dada de alta solo en el directorio
   entra en Open WebUI con su contrasena del directorio y conversa con su
   perfil. Ningun paso toca Open WebUI.
4. **Baja en un solo sitio.** Dada de baja en el directorio, o fuera de
   `ia_nest_users`, no puede iniciar sesion; una sesion ya abierta no conversa
   (403 del borde). La caducidad de la sesion es configurable y esta declarada
   (H07P2); mientras dure, la sesion abierta solo ve sus chats.
5. **Contrasena en un solo sitio.** Cambiada en el directorio, la nueva entra y
   la anterior no.
6. **Sin cuentas locales (C3).** La unica cuenta que inicia sesion con una
   contrasena guardada en Open WebUI es el admin de emergencia, declarado como
   tal. Con la contrasena del directorio por el login local, cualquier persona
   es rechazada.
7. **Chats conservados.** Las personas que ya tenian cuenta conservan sus chats.
8. **Menu por perfil a la vista.** Una persona infantil solo ve en Open WebUI
   las entradas de su menu; una adulta, las suyas. El borde sigue rechazando
   cualquier otra.
9. **Fallo cerrado.** Con el directorio parado, ninguna persona inicia sesion.
10. **Tareas de fondo (A15).** Medido en la auditoria del borde: el primer
    turno de un chat genera como mucho una tarea de fondo y los siguientes
    ninguna, todas por la entrada por defecto del perfil (H07P5).

Home Assistant, Kodi y agente Linux (C6):

11. **Conversa por su clase.** Cada uno conversa con el ente con su cuenta, su
    clase y su perfil (los que no afirman personas se atienden como ellos
    mismos, H05P3); una peticion fuera de su menu, o con herramientas, se
    rechaza.
12. **Agente Linux sin huella.** Una pasada del agente no escribe en la
    memoria del ente (`ia_nest_agents`, `docs/PLAN.md`, decision 3).

Laboratorio:

13. **Ningun paso a mano** que un instalador deberia hacer.

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H07P1 | Criterio de salida | este hilo | - | decidido: el punto 1 es estricto. El borde sigue en el mismo tag todo el hito; un consumidor que necesite codigo del borde no es C6, se decide aparte y antes, y no bloquea a los demas |
| H07P2 | Open WebUI: login contra el directorio, cuentas locales, chats, admin de emergencia; retirar el paso coyuntural de `cuentas.sh` | este hilo | H07P1 | decidido: login LDAP propio de Open WebUI por LDAPS con la raiz del laboratorio; lee con `svc_open_webui` en el grupo de solo lectura del directorio (su contrasena es la del bind; su clave del borde sigue aparte, H05P2) y solo entra quien esta en `ia_nest_users`. Open WebUI guarda de cada persona una ficha por email (chats y grupos), creada con rol de usuario al primer login; las cuentas que ya existian son esa ficha y conservan sus chats. Su contrasena local se sustituye una vez por una aleatoria que nadie conoce: solo el admin de emergencia inicia sesion en local (C3). `directorio.sh` deja de tocar Open WebUI y `cuentas.sh` se retira; la conexion con la clave del borde y la sustitucion de una vez pasan al instalador de Open WebUI. Sesion configurable; en el laboratorio, sin caducidad (uso interno): una baja no conversa desde la siguiente peticion, pero una sesion ya abierta sigue viendo sus chats hasta cerrarla. Que la administracion de `ia_nest_web` cambie estas opciones queda como camino de esa capa. Motivo: es la practica habitual con LDAP (bind de servicio de solo lectura, busqueda, bind de la persona, ficha creada al primer login). Descartados ahora: OIDC delante del directorio (una pieza y una parada mas; queda como camino), cabecera de confianza (otra pieza, y quien llegue a Open WebUI saltandose el proxy entra como quien diga) y seguir sincronizando cuentas (segunda copia de las personas: rompe C1 y C3) |
| H07P3 | Open WebUI: modelos visibles por perfil | este hilo | H07P2 | decidido: Open WebUI copia en cada login los grupos de la persona en el directorio, y cada entrada de la conexion se da a un grupo, ninguna a todos: `ia_nest` (el alias del borde para la entrada por defecto del perfil) a `ia_nest_users`; `directo` y `razonado` a `profile_adult`; `iterativo_prototipo` a `edge_prototypes`. Una persona infantil solo ve `ia_nest`; el admin de emergencia las ve todas (un admin de Open WebUI no pasa por los grupos), pero el borde le da 403. No es otra lista de personas (la objecion de H05P8): sale del directorio en cada login. Lo que se ve es comodidad y quien decide sigue siendo el borde: un grupo atrasado (cambio de perfil con la sesion abierta, o los dos grupos de perfil a la vez) solo deja ver una entrada que el borde rechaza, nunca abre una. Descartado: pedir al borde el menu de cada persona, porque Open WebUI pide la lista una vez con la clave de la conexion y no por persona; haria falta codigo (H07P1) |
| H07P4 | Contrasenas de personas guardadas en claro en el laboratorio | este hilo | H07P2 | decidido: la contrasena de una persona no se guarda en ningun sitio fuera del directorio. El script del directorio la pide por teclado o genera una aleatoria que muestra una sola vez; si se olvida, se restablece. Las cuentas de sonda guardan la suya aparte, en una carpeta solo de pruebas automaticas, y el script la recibe por la entrada estandar. Las contrasenas de personas reales ya guardadas se borran, en el equipo del operador y en la copia de secretos, despues del montaje de H07P6 (hasta entonces `cuentas.sh` las lee). Motivo: es la practica habitual (el administrador restablece, no guarda); una copia en claro es una segunda fuente de las credenciales y deja de tener consumidor con H07P2. Descartado: guardarlas cifradas (sigue siendo una copia) |
| H07P5 | Tareas de fondo de Open WebUI (A15) | este hilo | H07P2 | decidido: solo el titulo del chat, por `ia_nest` (la entrada por defecto del perfil; el borde ya no escribe memoria con las tareas de fondo). Apagadas las etiquetas, las preguntas de seguimiento y la generacion de consultas de busqueda y de RAG (el RAG es de extended). En la configuracion del instalador de Open WebUI. Motivo: cada tarea ocupa el modelo local mientras la persona espera su siguiente respuesta (visita de laboratorio: 1-3 por turno) |
| H07P6 | Montaje de Open WebUI en el laboratorio por su instalador; evidencia en `ia_nest_lab` | lab | H07P2-P5 | hecho: Open WebUI tiene instalador propio en el laboratorio, que deja toda su configuracion por su API (su base manda sobre el entorno tras el primer arranque); el script del directorio pone a `svc_open_webui` en el grupo de solo lectura y deja de tocar Open WebUI. Criterio 1, 3-10 y 13 PASA para Open WebUI con una sonda repetible, y la sonda de `H05` sigue pasando. Comprobado que las personas reales entran por el directorio; sus contrasenas guardadas, borradas (H07P4). Notas: en Open WebUI apagar el login por contrasena apaga tambien el LDAP, por eso las fichas llevan contrasena local aleatoria; un admin de Open WebUI ve todas las entradas, y el borde le da 403 |
| H07P7 | Home Assistant: cuenta, clase y configuracion | este hilo, lab | H07P6 | pendiente |
| H07P8 | Kodi: cuenta, clase y configuracion | este hilo, lab | H07P6 | pendiente |
| H07P9 | Agente Linux: piel OpenAI o REST/MCP (A12), `write_memory`, cuenta y clase | este hilo, agents, lab | H07P6 | pendiente |
| H07P10 | Arquitectura y hoja de ruta al dia; PR | meta | H07P6-P9 | pendiente |

## Reglas del hito

- Una decision cada vez, con diagrama y explicacion breve; primero la solucion
  corta; criterio falsable antes que mecanismo.
- En meta, papeles y motivos; nombres de maquina, direcciones y evidencia, en
  `ia_nest_lab`.
