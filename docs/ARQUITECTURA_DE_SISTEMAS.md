# Arquitectura de sistemas del ente

Estado: activo
Version: 1.0 - 2026-10-27

Donde CORRE el ente: que maquina, que red, que tramo cruza que perimetro y de
que depende cada pieza. Complementa a `ARQUITECTURA_DE_CAPAS.md`, que dice como
se componen las capas. Este documento no lleva direcciones, nombres de maquina
ni secretos: viven en el laboratorio (`ia_nest_lab`).

Origen: meta ADR 0016 (hito `H03`).

## 1. El invariante

> **Toda decision de infraestructura del ente se mide contra la referencia del
> punto 2: o acerca el ente a ella, o no la aleja.**

Pregunta literal ante cualquier propuesta: el dia que el ente se monte como
recinto, o se lleve a otro entorno, esto hay que deshacerlo? Si la respuesta es
si, la propuesta no cumple o declara el coste.

## 2. Referencia: recinto con pasarela

IA_NEST es un recinto autocontenido detras de una pasarela (papel `gate`).

| Pieza | Papel |
|---|---|
| Pasarela, interfaz delantero | Publica solo los puntos de uso del ente: el borde (REST, MCP, piel OpenAI) y LDAPS. Nada mas. |
| Pasarela, interfaz trasero | Da a la red contenida sus recursos: directorio, DNS, DHCP, proxy de salida, CA, hora. |
| Red contenida | Donde vive el ente (core, extended, borde y las capas que vengan). Solo sale al exterior por la pasarela. |
| Recursos conectados | Backend de modelos (GPU) y almacen del RAG. El ente los alcanza por configuracion de conexion; cada uno puede vivir donde convenga, independiente del resto. |
| Anfitrion | Solo da recursos: GPU y maquinas virtuales. No aloja servicios del ente. |

Lo que un entorno tiene que dar para alojar el ente: una direccion delantera
para la pasarela y la GPU. Llevarse el ente es llevarse la pasarela y la red
contenida, y reconectar los recursos conectados.

Reglas de compatibilidad, para no tener que rehacer nada al llegar al recinto:

1. Los servicios de la pasarela se configuran por papel (delante o detras), no
   por direccion.
2. El directorio escucha detras; delante solo se publica LDAPS.
3. Un recurso conectado entra al ente por su configuracion de conexion, nunca
   por una ruta del sistema de ficheros ni por estar en la misma maquina.
4. La GPU es la unica pata del ente en el anfitrion. Opciones para el recinto,
   con sus riesgos, en el hito `H03`.

## 3. Ahora

El ente sigue en la red donde esta; la pasarela nace en esa red con un solo
interfaz y lleva el directorio. El interfaz trasero se anade al pasar al
recinto. La red del ente comparte VLAN con produccion de la casa: cerrar el
paso de personas hacia esa red es tarea de la casa, y el ente no necesita
ninguna excepcion de entrada para funcionar.

| Servicio | Papel en la referencia | Ahora | Copia de seguridad |
|---|---|---|---|
| core, extended, borde | red contenida | VM del ente, en la red critica | por reinstalacion desde tag; datos en el almacen |
| Directorio | pasarela, detras | pasarela, en la red critica (`H04`) | exportacion cifrada al volumen de copias del anfitrion; snapshot antes de cambios grandes |
| CA | pasarela | raiz autofirmada del laboratorio, clave fuera de linea (`H06`) | con los secretos, cifrada, al volumen de copias |
| DNS, DHCP, proxy de salida | pasarela, detras | no hacen falta todavia: los da la casa | se rehacen con el instalador |
| Almacen del RAG | recurso conectado | contenedor en el anfitrion | volcado periodico al volumen de copias |
| Backend de modelos | recurso conectado | contenedor en el anfitrion, con la GPU | sin datos: los modelos se vuelven a descargar |
| Open WebUI | consumidor (`H07`) | servicios compartidos de la casa, en la red de servicios | de la casa |

El volumen de copias es un RAID 1 del anfitrion en discos distintos de los de
las VMs y los datos. No es una copia fuera del anfitrion: coste aceptado.

## 4. Tramos

Un tramo que no esta en esta tabla no se abre. TLS en todos los que cruzan el
perimetro: `H06` (meta ADR 0015, C8).

| Origen | Destino | Servicio | Cruza perimetro | Motivo |
|---|---|---|---|---|
| Personas | Open WebUI (red de servicios) | HTTPS | si (ya permitido) | las personas entran por Open WebUI |
| Open WebUI (red de servicios) | borde (red critica) | HTTPS | si: excepcion declarada | unica entrada de servicios al ente |
| Open WebUI (red de servicios) | directorio (red critica) | LDAPS | si: excepcion declarada (`H04`) | C7 |
| HA, Kodi (red critica) | borde | HTTPS | no | consumidores en la misma red |
| HA, Kodi (red critica) | directorio | LDAPS | no | C7 |
| borde | directorio | LDAPS | no | resolucion de identidades (C1, C4) |
| ente | backend de modelos, almacen del RAG | HTTP, PostgreSQL | no | recursos conectados |
| pasarela, almacen del RAG | volumen de copias | copia | no | copia de seguridad |
| Red de gestion | todas las maquinas | ssh | si (ya permitido) | administracion |

El operador conserva su acceso a extended por loopback en la VM del ente (meta
ADR 0015, punto 3).

## 5. Disponibilidad

- Una sola pasarela. Una segunda en el mismo anfitrion no cubre su caida, y
  sin anfitrion tampoco hay GPU: la disponibilidad del ente la marca el
  anfitrion.
- Reiniciar la pasarela: mientras dura, el directorio no responde y el borde
  rechaza todo con 503 (fallo cerrado, C4); al volver, todo entra sin tocar
  nada. Es criterio de `H04`.
- La pasarela arranca la primera y se apaga la ultima; el apagado y arranque
  seguros del anfitrion la reconocen. Sus reinicios automaticos por
  actualizacion van en ventana nocturna.

## 6. TLS

- Una raiz autofirmada de laboratorio, sin caducidad practica, que cada
  consumidor instala una sola vez. Firma los certificados de cada servicio.
  Nada de certificados sueltos autofirmados ni de desactivar la verificacion
  (`edge ADR 0002`, punto 8).
- La clave de la raiz vive fuera de linea, con los secretos del laboratorio.
- Aviso: los dispositivos Apple rechazan certificados de servidor de mas de
  825 dias aunque la raiz sea propia. Hoy no hay ninguno; si llega, los
  certificados de servicio pasan a durar unos dos anos.
