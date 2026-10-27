# H06: TLS en los tramos que cruzan el perimetro

Estado: abierto
Abierto: 2026-10-27

## Objetivo

Que ningun tramo del perimetro vaya en claro (meta ADR 0015, C8), con la base
decidida en `H03` (meta ADR 0016, punto 5): una raiz autofirmada de
laboratorio, sin caducidad practica, que firma un certificado por servicio.
Nada de certificados sueltos autofirmados ni de desactivar la verificacion
(`edge ADR 0002`, punto 8). Clave de la raiz fuera de linea, con los secretos
del laboratorio, y copia en el volumen de copias del anfitrion.

Alcance (decidido por el usuario el 2026-10-27): las piezas funcionando y
comprobadas, con la seguridad minima de laboratorio. La red del ente se
asegura en los proximos meses; endurecer la CA (frase de paso, copia cifrada,
clave del servicio sin viajar) se hace entonces, sin rehacer lo de aqui.

Tramos de hoy (`docs/ARQUITECTURA_DE_SISTEMAS.md`, punto 4): personas a Open
WebUI, Open WebUI al borde, Home Assistant y Kodi al borde. LDAPS no existe aun:
nace con esta misma raiz en `H04`, que depende de este hito.

No toca el contrato de ninguna capa: el borde ya sirve HTTPS con certificado y
clave declarados en su configuracion. Si al montarlo falta algo en su
instalador, es fallo de codigo y va por PR en su repo.

Aviso heredado: los dispositivos Apple rechazan certificados de servidor de mas
de 825 dias. Hoy no hay ninguno.

## Criterio de salida

Decidido por el usuario el 2026-10-27.

1. El borde no responde en claro: una peticion HTTP a su puerto, desde la red
   de servicios y desde la critica, no obtiene respuesta HTTP.
2. Con la raiz como UNICA autoridad de confianza, una peticion HTTPS sin clave
   al borde devuelve 401 desde la maquina de Open WebUI. Sin la raiz, la misma
   peticion falla la verificacion: prueba que el consumidor verifica y no la
   ignora. Home Assistant y Kodi no hablan con el borde todavia: la misma
   prueba, desde el entorno del propio consumidor, es criterio de `H07`.
3. Open WebUI se publica a las personas solo por HTTPS: desde la red de
   personas, un navegador con la raiz instalada abre sin aviso y el puerto en
   claro no sirve la aplicacion.
4. Un chat de Open WebUI responde a traves del borde por HTTPS verificado. Con
   la raiz retirada de su almacen de confianza, la conexion al borde falla.
5. Ninguna configuracion de consumidor ni del borde desactiva la verificacion.
   Unica excepcion declarada: la sonda local del instalador del borde.
6. El certificado del borde esta en su configuracion y entra por la segunda
   pasada de su instalador; no hay ningun paso a mano que el instalador
   deberia hacer.
7. La copia de la clave de la raiz en el volumen de copias casa con el
   certificado raiz publicado.

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H06P1 | Criterio de salida falsable desde C8 | este hilo | - | decidido |
| H06P2 | Donde se genera y guarda la raiz | este hilo | H06P1 | decidido: en el equipo del operador con `openssl`, por un script del laboratorio que tambien emite los certificados de servicio; la clave de la raiz, sin frase de paso (alcance minimo), con los secretos del laboratorio y una copia en el volumen de copias; el certificado raiz es publico y se reparte; el certificado y la clave de cada servicio llegan a su maquina como requisito previo de su instalador |
| H06P3 | Duracion de los certificados de servicio | este hilo | H06P2 | decidido: lo maximo razonable, raiz 25 anos y servicios 20, para no depender de recordar una renovacion; Apple queda fuera hoy; cada certificado emitido se apunta en un inventario del laboratorio (servicio, nombres, emision, caducidad, huella) que pueda leer una futura interfaz de gestion |
| H06P4 | Como se instala la raiz en cada consumidor (navegadores, Open WebUI, HA, Kodi) | este hilo | H06P2 | decidido (el usuario lo delega con un criterio: coste minimo, ningun paso que haya que recordar): Open WebUI la lleva en la definicion de su stack, como unica autoridad; navegadores, a mano una vez por dispositivo con guia en el laboratorio; Home Assistant y Kodi, en `H07` y DENTRO de su propia instalacion (configuracion de la integracion o del complemento), para que un consumidor nuevo no pida un paso aparte |
| H06P5 | Quien termina TLS de Open WebUI | este hilo | H06P1 | decidido: un proxy inverso en el mismo stack que Open WebUI, en el mismo puerto de siempre (ninguna regla del perimetro cambia); Open WebUI deja de publicarse; el borde sirve solo HTTPS en su puerto de siempre |
| H06P6 | Montaje en el laboratorio por la via natural; evidencia en `ia_nest_lab` | lab, repo del borde si falta algo | H06P2-P5 | hecho: borde por su configuracion y segunda pasada del instalador (no le falta nada; con la configuracion ya instalada hay que pedir que se sustituya); proxy inverso y raiz en la definicion del stack de Open WebUI; conexion de Open WebUI al borde cambiada a https; criterio 1, 2, 4-7 PASA, 3 PASA salvo el navegador real (pendiente de que el operador importe la raiz); ningun cambio de cortafuegos |
| H06P7 | `ARQUITECTURA_DE_SISTEMAS.md` al dia, hoja de ruta y PR | meta | H06P6 | hecho en rama local; PR pendiente de aprobacion |

## Notas para despues

- Interfaz de gestion del ente (sin repo ni hito): deberia vigilar la caducidad
  de los certificados a partir del inventario del laboratorio. No se registra
  en `CAPAS_FUTURAS.md` hasta que tenga consumidor.
- Endurecimiento de la CA al asegurar la red del ente: frase de paso, copia
  cifrada, peticiones de firma en la maquina de cada servicio.
