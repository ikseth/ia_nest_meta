# Decision 0016: arquitectura de referencia del ente (recinto con pasarela)

Fecha: 2026-10-27

Hito `H03` de la hoja de ruta (meta ADR 0014).

## Contexto

El ente crecio pieza a pieza sobre un anfitrion con GPU: capas en una VM, el
almacen del RAG y el backend de modelos en contenedores del anfitrion, Open
WebUI en los servicios compartidos de la casa, todo en una red que comparte
VLAN con produccion. Meta ADR 0015 pide ahora un directorio central del que
depende que entre alguien (C4), LDAPS desde dos redes (C7) y TLS en el
perimetro (C8). Montar esas piezas sin una forma escrita repetiria el patron:
cada una donde cae, y llevar el ente a otro entorno seria rehacerlo.

Lo que el operador quiere poder hacer: llevarse IA_NEST entero a otro entorno,
y que cada decision de infraestructura de hoy no lo impida.

## Decision

1. **Referencia: recinto con pasarela.** El ente vive en una red contenida
   detras de una pasarela (papel `gate`). Delante, la pasarela publica solo los
   puntos de uso (borde y LDAPS); detras, da a la red contenida sus recursos
   (directorio, DNS, DHCP, proxy de salida, CA). Un entorno anfitrion solo
   tiene que dar una direccion delantera y la GPU.
2. **Recursos conectados.** El backend de modelos y el almacen del RAG no son
   parte integrada del ente: los alcanza por configuracion de conexion y cada
   uno puede vivir donde convenga. El almacen se queda donde esta; en una
   instalacion grande tendria su propia maquina.
3. **Ahora, un paso hacia la referencia.** El ente no cambia de red. La
   pasarela nace en esa red con un solo interfaz y lleva el directorio; el
   interfaz trasero se anade al pasar al recinto. Sistema operativo de la
   pasarela: openSUSE MicroOS. Una sola pasarela.
4. **Regla de compatibilidad.** Toda decision de infraestructura se mide contra
   la referencia: los servicios de la pasarela se configuran por papel y no por
   direccion; el directorio escucha detras y delante solo LDAPS; un recurso
   conectado entra por su configuracion de conexion.
5. **TLS sin esperar.** Una raiz autofirmada de laboratorio, sin caducidad
   practica, firma los certificados de cada servicio. `H06` pasa delante de
   `H04`: LDAPS nace con esa CA.
6. **La pasarela es infraestructura, no capa.** Su instalador vive en el
   laboratorio mientras se descubre montandola. Su repo previsto,
   `ia_nest_gate`, queda en `CAPAS_FUTURAS.md` con su disparador.

El detalle vivo (servicios, tramos, copias, disponibilidad, TLS) esta en
`docs/ARQUITECTURA_DE_SISTEMAS.md`.

## Motivos

- Recinto con pasarela: llevar el ente es llevar dos piezas; lo que entra y lo
  que sale pasa por un sitio; el anfitrion queda reducido a recursos.
- Una sola pasarela: el fallo cerrado ya acopla el directorio a todo el ente, y
  una segunda en el mismo anfitrion no cubre la caida que importa.
- No mover el ente de red ahora: el recinto es la referencia, no una urgencia;
  ahora basta con no alejarse de ella.
- MicroOS: la pasarela solo corre contenedores y tiene que arrancar siempre;
  actualizaciones por transacciones con vuelta atras sola si el arranque falla.

## Consecuencias

- Nuevo `docs/ARQUITECTURA_DE_SISTEMAS.md`, en el orden de lectura de
  `AGENTS.md` detras de `ARQUITECTURA_DE_CAPAS.md`.
- Hoja de ruta: `H06` antes que `H04`; `H04` depende de `H06`.
- `CAPAS_FUTURAS.md` registra la pasarela y su repo previsto.
- `edge ADR 0002` (punto 8: certificado de una CA propia instalada una vez en
  cada cliente) ya es compatible; no necesita puntero.
- Lo que queda para el laboratorio: alta de la pasarela, su lugar en el
  apagado y arranque seguros del anfitrion, y la copia del almacen del RAG.

## Lo que esta decision NO hace

- No monta nada: la pasarela y el directorio son `H04`; TLS es `H06`.
- No elige producto del directorio.
- No mueve el ente de red ni el almacen del RAG.
- No cierra el paso de personas hacia la red de produccion de la casa: es tarea
  de la casa, que ya no depende del ente.
- No pide ningun Change Request: ningun contrato cambia.

## Impacto de version

Ninguno en el contrato publico de ninguna capa.
