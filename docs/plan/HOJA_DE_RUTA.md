# Hoja de ruta del ente

Estado: activo
Version: 0.20 - 2026-11-09 (H11 planificado: respuesta de las acciones en su canal)

Nivel 0 del plan (meta ADR 0014). Solo hitos, orden y estado. El detalle vive
en el fichero de cada hito; un hito sin fichero aun no esta abierto.

La idea central y sus mapas estan en `docs/EL_CAMINO.md`. Cada hito indica que parte
de esos mapas mueve.

## Idea final

1. Una infraestructura comun para consumir IA_NEST por REST y MCP, sobre la que
   se incorporan consumidores: Open WebUI, Home Assistant, Kodi, agentes y los
   que vengan.
2. La autenticacion es el punto de partida: un directorio de identidades propio
   de IA_NEST por el que pasan todos los consumidores.
3. La infraestructura del ente se disena antes de montarse: servicios en VMs
   que se migran y aseguran aparte; tron solo da recursos. Referencia: recinto
   con pasarela (`docs/ARQUITECTURA_DE_SISTEMAS.md`).

## Hitos

| Id | Hito | Depende de | Estado |
|---|---|---|---|
| [H08](hitos/H08_entrada_de_la_familia.md) | Entrada de la familia: confirmador en Open WebUI y voz, `directo` con dominio fijo en Kodi (H08P17) | H06, H07, H09, trabajo de capa | abierto |
| H11 | Respuesta de las acciones en su canal: una accion pedida desde un canal (chat, voz, pantalla, agente) termina con su resultado, correcto o fallido, en ese mismo canal y queda registrada; hoy una accion larga contesta "en ello" y su resultado no vuelve (H08P27). Diseno en meta: criterio, ADR situado frente a las tareas largas de MCP, el ciclo de tareas de A2A y los webhooks, y trabajo por capa (borde, core, capas externas, consumidores). Base para conscience | H08 | planificado |
| H10 | Orquestador para la familia: la salida "complejo" del confirmador, `edge CR-0003`, no inventar cuando no entiende y un plan mas rapido; entra en Open WebUI por configuracion cuando pase su bateria (H08P17) | H08, H11 | planificado |

El criterio de salida de H04-H08 esta en meta ADR 0015 (C1-C8, con el hito que
cumple cada uno).

Lo asignado a un hito que aun no tiene fichero esta en el inventario de `H01`
(`pasos/H01P1_inventario.md`): se lee al abrir el hito.

## Trabajo de capa que esta hoja vigila

Se conduce en su capa; aqui solo consta que bloquea a un hito. Asignado en
`H01P2`.

| Que | Donde | Bloquea |
|---|---|---|
| `edge CR-0001` (perfil del interlocutor) | core#44 | - (entregado en core `v0.4.2`, H08P9) |
| `edge CR-0002` (extended lo compone) | extended#81 | H08 (mecanismo hecho en extended `v0.5.4`; falta comprobarlo de punta a punta por el borde, criterio 2) |
| L5r y L4b de la puerta de extended | extended, Deck #125 | H08 (en verde en extended `v0.6.4` con un ejecutor; el segundo, en H08P10) |
| `razonado` entrega su JSON interno y corta por contexto acumulado | core#45 | - (fuera de los menus de la familia mientras siga abierto, H08P1) |
| El instalador del core no reinicia los servicios al actualizar y su verificacion no lo detecta | core#47 | - (entregado en core `v0.4.3`, ficha 0014, desplegado sin paso a mano; H08P12) |
| Una peticion que no se contesta no avisa bien: un tiempo de espera hacia el core llega al cliente como HTTP 400, y un `TimeoutError` a mitad de flujo termina sin evento de error ni rastro en la telemetria (H08P14, H08P21) | extended#98 | H08 (criterio 7; corregido en extended `v0.6.5`, desplegado y comprobado en el laboratorio; el paso por el borde, en H08P10) |
| Un fallo al escribir la memoria despues de entregar la respuesta convierte el turno en error: la voz oye la respuesta y despues el aviso de fallo (H09P6) | extended#100 | H08 (vigilado; deja de bloquear H09 por decision del usuario el 2026-11-09, H09P28) |
| Memoria y conocimiento en paralelo e indice vectorial: recordar cuesta 4,8 s por turno (H08P6, H08P18) | extended | - (sin prioridad, H08P21) |
| La eleccion de dominio cuesta ~3 s por turno | core#48 | - (no se desarma el diseno por latencia, H08P13) |
| Segunda GPU del recurso de modelos (H08P7) | anfitrion, Deck #126 | - (mejora; deja de bloquear H08 al reajustar el criterio 7, H08P23; se aprueba contra la bateria v2 de H09, H09P28) |
| Extended declara `core >=0.4 <0.5` y el laboratorio ya sirve core `v0.5.0` (sin eventos D2 nuevos) | extended#101 | - (re-verificar y ampliar el rango, deber de 2b) |
| El instalador del core reescribe su fichero de entorno y borra la variable de `auth_env` de un proveedor de herramientas | core#56 | - (cubierto en el laboratorio con un drop-in de systemd, H09P16) |
| `edge CR-0003` (modo iterativo de uso normal) | extended#82 | H10 |

## Medidas de referencia

Decidido por el usuario el 2026-11-09 (H08P24). Vale para todos los hitos y
capas.

- Las baterias de medida son fijas: no cambian sin subir de version (v1, v2...).
- Una medida es la referencia tal como sale; no se repite para confirmarla ni
  para explicar una diferencia: la diferencia se anota con lo que cambio.
- Cada cambio desplegado en el laboratorio que pueda afectar a la calidad o al
  tiempo (version de una capa, configuracion, modelos, hardware) se mide con las
  baterias que le tocan y anade una fila al historico, marcada con el cambio o
  el hito.
- Los criterios de salida que hablan de tiempo se juzgan por experiencia de uso
  mirando esa serie (criterio 4 de `H09`, criterio 7 de `H08`).
- El historico, con cifras, versiones y evidencia, vive en el laboratorio
  (privado); aqui solo la pauta.

## Cerrados

- [H00](hitos/cerrados/H00_modelo_de_orquestacion.md): modelo de orquestacion desde meta (meta ADR 0014). Cerrado el 2026-10-27.
- [H01](hitos/cerrados/H01_migracion_al_modelo.md): migracion al modelo, inventario y orden de la casa. Cerrado el 2026-10-27.
- [H02](hitos/cerrados/H02_criterio_de_apertura.md): criterio de apertura revisado (meta ADR 0015). Cerrado el 2026-10-27.
- [H03](hitos/cerrados/H03_arquitectura_de_sistemas.md): arquitectura de referencia, recinto con pasarela (meta ADR 0016). Cerrado el 2026-10-27.
- [H06](hitos/cerrados/H06_tls_en_el_perimetro.md): TLS en los tramos que cruzan el perimetro, raiz de laboratorio con alcance minimo. Cerrado el 2026-10-27.
- [H04](hitos/cerrados/H04_directorio_de_identidades.md): directorio de identidades en la pasarela (lldap, solo LDAPS), con copia, restauracion y orden de arranque probados. Cerrado el 2026-10-27.
- [H05](hitos/cerrados/H05_borde_contra_el_directorio.md): borde contra el directorio (clave de API por cuenta, sin identidades en su configuracion) y punto REST/MCP, con fallo cerrado probado. Cerrado el 2026-10-28.
- [H07](hitos/cerrados/H07_consumidores_uno_a_uno.md): consumidores uno a uno: Open WebUI por el directorio, Kodi y agente Linux con su cuenta y su clase (borde `v0.2.1`); Home Assistant, al diseno aparte vigilado. Cerrado el 2026-10-28.
- [H09](hitos/cerrados/H09_home_assistant_al_ente.md): Home Assistant al ente: Luna por el ente, estado y control por `tool_contracts` (core `v0.5.0`) y la capa externa `v0.1.0` con catalogo cerrado; criterio 4 por experiencia, con la bateria v2 de referencia para la segunda GPU. Cerrado el 2026-11-09.
