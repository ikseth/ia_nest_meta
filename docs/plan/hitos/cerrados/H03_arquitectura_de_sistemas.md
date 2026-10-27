# H03: arquitectura de sistemas y servicios de IA_NEST

Estado: cerrado (2026-10-27)
Abierto: 2026-10-27

## Objetivo

Dejar escrita la arquitectura de referencia de IA_NEST, para que cada decision
de infraestructura que venga (montar el directorio en `H04`, TLS en `H06`,
llevar el ente a otro entorno) la tenga en cuenta y no rompa la compatibilidad
con ella. Y fijar el estado de ahora, que es un paso hacia esa referencia.

Referencia (decidida por el usuario el 2026-10-27): IA_NEST es un recinto
autocontenido detras de una pasarela (`gate`). La pasarela tiene un interfaz
delantero, que publica solo los puntos de uso del ente, y uno trasero, que da a
una red contenida los recursos que el ente necesita (directorio, DNS, DHCP,
proxies, CA). Todo el ente vive detras. El anfitrion de la GPU solo da
recursos.

Recursos conectados (decidido por el usuario el 2026-10-27): el backend de
modelos (GPU) y el almacen del RAG no son parte integrada del ente sino
recursos a los que llega por configuracion de conexion. Cada uno debe poder
vivir donde convenga, independiente del resto; en una instalacion grande, el
almacen del RAG (la memoria del ente) tendria su propia maquina. Que hoy viva
en el anfitrion lo confirma: el ente ya lo trata como conectado.

Ahora (decidido por el usuario el 2026-10-27): el ente sigue en la red donde
esta; la pasarela nace con un solo interfaz y lleva el directorio. El interfaz
trasero se anade al migrar al recinto.

Hereda de meta ADR 0015: C4 (fallo cerrado: la disponibilidad del directorio
es requisito), C7 (LDAPS desde la red de servicios y desde la critica) y C8
(ningun tramo del perimetro en claro).

No monta nada ni elige producto del directorio. Los nombres de maquina del
laboratorio, las direcciones y el inventario viven en `ia_nest_lab`; aqui,
papeles, decisiones y motivos.

## Criterio de salida

1. Hay un ADR de meta con la arquitectura de referencia: el recinto con
   pasarela, lo que el entorno anfitrion tiene que dar (una direccion delantera
   y la GPU) y la regla de compatibilidad (los servicios de la pasarela se
   configuran por papel, delante o detras, no por direccion).
2. Cada servicio de IA_NEST (capas, borde, directorio, DNS, almacen del RAG,
   backend de modelos, CA) tiene escritos su papel en la referencia y, ahora,
   maquina, red y copia de seguridad.
3. Cada tramo entre redes que usa IA_NEST ahora esta en una tabla con origen,
   destino, servicio y motivo; un tramo que no esta en la tabla no se abre.
4. Cada recurso conectado (backend de modelos, almacen del RAG) se alcanza
   solo por configuracion de conexion, y su ubicacion de ahora esta declarada.
   En el anfitrion no corre ningun otro servicio con datos del ente.
5. El efecto de reiniciar la pasarela esta escrito (que se cae, cuanto, quien
   lo nota), con la prueba que lo comprueba al montarla en `H04`: mientras
   dura, el borde responde 503 a todos; al volver, todo entra sin tocar nada.
6. Esta escrito donde vive la CA propia y quien emite certificados, para `H06`.
7. Cada decision de una sola capa queda como nota para su hilo.

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H03P1 | Arquitectura de referencia: recinto con pasarela | este hilo | - | decidido: patron y nombre de papel `gate` |
| H03P2 | Ahora: el ente donde esta, pasarela con un interfaz; tabla de tramos (A05) | este hilo | H03P1 | hecho: tabla en `ARQUITECTURA_DE_SISTEMAS.md`; el ente no necesita excepciones de entrada hacia la red critica, el resto de A05 es tarea de la casa |
| H03P3 | La pasarela: servicios, sistema operativo, instalacion y copia de seguridad | este hilo | H03P1 | decidido: openSUSE MicroOS; instalador en `ia_nest_lab` por ahora, `ia_nest_gate` previsto en `CAPAS_FUTURAS.md`; copia: exportacion cifrada del directorio, la CA y los secretos al volumen de copias del anfitrion (RAID 1 en discos distintos de los de las VMs y los datos) y snapshot antes de cambios grandes; el resto se rehace con el instalador (decidido) |
| H03P4 | Disponibilidad (C4): efecto del reinicio, una o mas | este hilo | H03P3 | decidido: una sola pasarela (una segunda en el mismo anfitrion no cubre su caida, y sin anfitrion tampoco hay GPU); autoarranque antes que el ente; reinicios de MicroOS en ventana nocturna; el apagado y arranque seguros del anfitrion la reconocen y la ordenan (ultima en apagarse, primera en arrancar) |
| H03P5 | Recursos conectados: almacen del RAG (A06) y la costura de la GPU | este hilo | H03P1 | decidido: el almacen se queda donde esta, declarado como recurso conectado; no se migra hasta que se monte el recinto; falta su copia (hoy no la tiene): volcado periodico y ligero al volumen de copias del anfitrion, tarea de lab |
| H03P6 | Base de TLS: CA propia y emision de certificados (para `H06`) | este hilo | H03P3 | decidido: raiz autofirmada de laboratorio sin caducidad practica que firma los certificados de cada servicio, clave fuera de linea; `H06` pasa delante de `H04` |
| H03P7 | ADR de meta; evidencia en `ia_nest_lab`; hoja de ruta al dia y PR | meta, lab | H03P1-P6 | hecho: meta ADR 0016, `docs/ARQUITECTURA_DE_SISTEMAS.md`, evidencia en `ia_nest_lab/labs/moria-gate/` (nombre de laboratorio); PR meta#25, mergeado |

## Cierre

Cerrado el 2026-10-27 con meta#25 mergeado: meta ADR 0016 fija la arquitectura
de referencia (recinto con pasarela, recursos conectados) y el estado de ahora,
y `docs/ARQUITECTURA_DE_SISTEMAS.md` lleva servicios, tramos, copias,
disponibilidad y TLS. `H06` pasa delante de `H04`. Quedan para el laboratorio:
el volcado del almacen del RAG a las copias del anfitrion y, al montar la
pasarela en `H04`, su lugar en el apagado y arranque seguros del anfitrion.

## La costura de la GPU

Es la unica pata del ente en el anfitrion. Opciones para el recinto, a futuro y
con sus riesgos escritos:

- Contenedor del backend en el anfitrion conectado solo a la red trasera, sin
  publicar nada delante. Hueco: quien administra el anfitrion entra al
  contenedor; el hipervisor ya esta dentro de la base de confianza.
- VM con la GPU por paso de PCI, que la sirve dentro de la red trasera. Aisla
  mas, pero ata la VM al hierro y cambia de hipervisor.

## Recursos que se citan

- La segunda GPU del anfitrion (GTX 1060, inventario B09) es recurso del
  anfitrion, fuera del plan; se cita como capacidad, no como servicio.

## Reglas del hito

- Una decision cada vez, con diagrama y explicacion breve; primero la solucion
  corta.
- Criterio falsable antes que mecanismo: no se elige producto del directorio
  (`H04`) ni se monta TLS (`H06`).
- Sistemas operativos: openSUSE Tumbleweed, luego Debian. La pasarela, openSUSE
  MicroOS (decidido el 2026-10-27): solo corre contenedores, se actualiza por
  transacciones y vuelve atras sola si el arranque falla.
- En meta, nombres de papel (`gate`); el nombre de cada maquina del
  laboratorio, en `ia_nest_lab`.
