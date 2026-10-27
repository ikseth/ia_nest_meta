# H04: directorio de identidades en la pasarela

Estado: abierto
Abierto: 2026-10-27

## Objetivo

Un directorio de identidades solo de IA_NEST, preparado para crecer, en la
pasarela (papel `gate`, meta ADR 0016). Es el punto de partida de meta ADR
0015: personas, servicios y agentes son cuentas del directorio, y el borde y
los consumidores la resuelven alli.

Decidido antes de abrir (no se reabre):

- Solo IA_NEST, preparado para crecer. Primer alcance: personas y grupos; el
  grupo da el perfil (adulto o infantil). Cuentas de servicio y agentes viven
  tambien en el directorio (C1), sin elegir aun como se autentican.
- Fallo cerrado (C4): sin directorio no entra nadie; el operador conserva
  extended por loopback.
- Vive en la pasarela: una sola, con un interfaz por ahora, openSUSE MicroOS,
  servicios por docker compose; instalador en el laboratorio (repo previsto
  `ia_nest_gate`).
- Delante solo LDAPS, con un certificado de la raiz de laboratorio (`H06`);
  nada en claro (C7, C8).
- Copias: exportacion nocturna del directorio al volumen de copias del
  anfitrion y snapshot antes de cambios grandes. Alcance minimo de
  laboratorio: sin cifrar.

Hereda de meta ADR 0015: C7 entero y la parte de C1 que toca al directorio. De
`H03`: la prueba de reiniciar la pasarela y su lugar en el apagado y arranque
seguros del anfitrion.

No toca el contrato de ninguna capa. El borde no consulta el directorio hasta
`H05`.

## Criterio de salida

Decidido por el usuario el 2026-10-27.

1. LDAPS desde la red de servicios y desde la critica, con la raiz como unica
   autoridad de confianza y la cuenta de solo lectura de un servicio: un
   `ldapsearch` devuelve personas y grupos, con al menos una persona en el
   grupo adulto y otra en el infantil. Sin la raiz, la misma busqueda falla la
   verificacion.
2. Nada en claro: desde las dos redes, LDAP sin TLS no responde y ningun otro
   puerto del directorio (tampoco su administracion web) se publica. Una
   busqueda anonima no devuelve ninguna entrada.
3. Alta y baja en un solo sitio: dar de alta una persona con su grupo, o de
   baja, es una operacion en el directorio, y la busqueda del punto 1 lo
   refleja sin tocar nada mas.
4. Restaurar: la pasarela reinstalada desde su instalador y cargada con la
   ultima copia nocturna devuelve la misma busqueda que antes.
5. Reiniciar la pasarela: vuelve sola y el directorio responde sin tocar nada.
6. El apagado y arranque seguros del anfitrion la reconocen: arranca la
   primera y se apaga la ultima.
7. Ningun paso a mano que el instalador deberia hacer.

Lo que no es de `H04`: "mientras dura el reinicio, el borde responde 503" es
C4 y se prueba en `H05`, cuando el borde consulta el directorio. Hasta
entonces el borde sigue sirviendo desde su fichero (estado de laboratorio,
meta ADR 0015).

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H04P1 | Criterio de salida; donde queda A09 (login de Open WebUI) | este hilo | - | A09 a `H07` (decidido): con C3; si Open WebUI entrase por el directorio antes de que el borde lo consulte (`H05`), habria dos fuentes de personas a la vez. Aqui basta el `ldapsearch` desde la red de servicios. Criterio: decidido, con la restauracion sobre la pasarela reinstalada para probar a la vez la copia y el instalador |
| H04P2 | Producto del directorio | este hilo | H04P1 | decidido: lldap, la solucion corta (un contenedor, LDAPS nativo, alta y baja por su web). Los consumidores solo hablan LDAP estandar: si se queda corto se cambia sin tocarlos. Costes declarados: las contrasenas solo viven en su base, asi que la copia es la base y no un LDIF; siempre escucha LDAP en claro y su web es HTTP, asi que la pasarela publica solo LDAPS y la web se alcanza por tunel ssh desde la red de gestion; permisos de tres roles fijos (admin, gestor de contrasenas, solo lectura) |
| H04P3 | Forma de personas y grupos: como da el perfil | este hilo | H04P2 | decidido: entra quien esta en `ia_nest_users`; el perfil lo da `profile_adult` o `profile_child`; en los dos o en ninguno, `child` (como el borde con un perfil desconocido); fuera de `ia_nest_users` no entra aunque tenga perfil. El `uid` es estable y no se reutiliza nunca: sera el `user_id` de la memoria de extended (`H05`). Para crecer: grupos de acceso por consumidor al lado, sin tocar estos |
| H04P4 | Cuentas de servicio | este hilo | H04P3 | decidido: una cuenta por consumidor que lea el directorio, en `ia_nest_services` y en el grupo de solo lectura del producto; nunca en `ia_nest_users`, asi que nunca entra como persona. Ahora solo `svc_edge` (la usa el borde en `H05` y la prueba del criterio 1); las demas, en el hito de su consumidor. Su clave la genera el instalador y vive con los secretos del laboratorio. Los agentes, con la misma forma, en `H07`. Como se autentican contra el borde: `H05` |
| H04P5 | Alta de la pasarela: direccion, primer arranque, autoarranque, apagado seguro | este hilo, la casa | H04P2 | decidido: direccion fija en la red critica, puesta en el primer arranque, con nombre en el DNS de la casa; VM pequena (2 vCPU, 2 GiB, 20 GiB); imagen oficial de MicroOS y un disco de Combustion que genera el instalador con todo (red, docker, directorio, certificado, cuenta de servicio, copia, ventana de reinicio), en una sola orden; autoarranque con la pasarela sin retraso y el resto de VMs con 60 s; el apagado seguro del anfitrion en dos fases, la pasarela la ultima (cambio de la casa); reinicios por actualizacion en ventana nocturna, despues de la copia. La excepcion del cortafuegos para LDAPS desde la red de servicios es de la casa, no del instalador |
| H04P6 | Copia y restauracion | este hilo | H04P5 | decidido: cada noche, antes de la ventana de reinicio, copia en caliente y consistente de la base del directorio; la pasarela la empuja al volumen de copias del anfitrion con una cuenta que solo puede escribir en su carpeta, asi el destino es configuracion de la pasarela; siete copias, una por dia de la semana, sin borrar nada desde la pasarela. Las claves fijas del directorio (sin la del servidor, una base restaurada no deja entrar a nadie) las genera el instalador una vez, viven con los secretos del laboratorio y el instalador deja su copia en el volumen de copias. Sin cifrar (alcance minimo). El instalador hace un snapshot antes de cambiar una pasarela existente, y restaurar es reinstalar con la ultima copia |
| H04P7 | Montaje en el laboratorio por su instalador; evidencia en `ia_nest_lab` | lab | H04P2-P6 | hecho: pasarela y directorio montados por su instalador; criterio 1-7 PASA. 1 y 2 desde la red de servicios, la critica y la de gestion; 4 con la pasarela destruida y reinstalada (volcado identico, claves validas); 5: el directorio vuelve solo en menos de un minuto; 6 con un reinicio real del anfitrion (la pasarela se apaga la ultima y arranca un minuto antes que el resto). Cuatro fallos del primer montaje corregidos en el instalador; la red critica solo da salida por direccion, asi que los paquetes se instalan en el primer arranque con la direccion fija y no en el initrd. Queda por ejercitar en un apagado normal la parada en dos fases del anfitrion, probada en seco |
| H04P8 | `ARQUITECTURA_DE_SISTEMAS.md` al dia, hoja de ruta y PR | meta | H04P7 | hecho: arquitectura 1.2 y hoja de ruta; PR pendiente de aprobacion |

## Reglas del hito

- Una decision cada vez, con diagrama y explicacion breve; primero la solucion
  corta.
- En meta, papeles y motivos; nombres de maquina, direcciones y evidencia, en
  `ia_nest_lab`.
- Lo que sea de la casa (excepcion del cortafuegos, apagado seguro del
  anfitrion) se pide al usuario o se anota en su ficha.
