# Hoja de ruta del ente

Estado: activo
Version: 0.8 - 2026-10-28 (H07 abierto)

Nivel 0 del plan (meta ADR 0014). Solo hitos, orden y estado. El detalle vive
en el fichero de cada hito; un hito sin fichero aun no esta abierto.

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
| [H07](hitos/H07_consumidores_uno_a_uno.md) | Consumidores, uno a uno: Open WebUI, Home Assistant, Kodi, agente Linux | H05 | abierto |
| H08 | Entrada de la familia | H06, H07, trabajo de capa | pendiente |

El criterio de salida de H04-H08 esta en meta ADR 0015 (C1-C8, con el hito que
cumple cada uno).

Lo asignado a un hito que aun no tiene fichero esta en el inventario de `H01`
(`pasos/H01P1_inventario.md`): se lee al abrir el hito.

## Trabajo de capa que esta hoja vigila

Se conduce en su capa; aqui solo consta que bloquea a un hito. Asignado en
`H01P2`.

| Que | Donde | Bloquea |
|---|---|---|
| `edge CR-0001` (perfil del interlocutor) | core#44 | H08 |
| `edge CR-0002` (extended lo compone) | extended#81 | H08 |
| L5r y L4b de la puerta de extended | extended, Deck #125 | H08 |
| `razonado` entrega su JSON interno y corta por contexto acumulado | core#45 | H08 |
| ~24 s hasta el primer token en `directo` | extended (y core) | H08 |
| `edge CR-0003` (modo iterativo de uso normal) | extended#82 | - |

## Cerrados

- [H00](hitos/cerrados/H00_modelo_de_orquestacion.md): modelo de orquestacion desde meta (meta ADR 0014). Cerrado el 2026-10-27.
- [H01](hitos/cerrados/H01_migracion_al_modelo.md): migracion al modelo, inventario y orden de la casa. Cerrado el 2026-10-27.
- [H02](hitos/cerrados/H02_criterio_de_apertura.md): criterio de apertura revisado (meta ADR 0015). Cerrado el 2026-10-27.
- [H03](hitos/cerrados/H03_arquitectura_de_sistemas.md): arquitectura de referencia, recinto con pasarela (meta ADR 0016). Cerrado el 2026-10-27.
- [H06](hitos/cerrados/H06_tls_en_el_perimetro.md): TLS en los tramos que cruzan el perimetro, raiz de laboratorio con alcance minimo. Cerrado el 2026-10-27.
- [H04](hitos/cerrados/H04_directorio_de_identidades.md): directorio de identidades en la pasarela (lldap, solo LDAPS), con copia, restauracion y orden de arranque probados. Cerrado el 2026-10-27.
- [H05](hitos/cerrados/H05_borde_contra_el_directorio.md): borde contra el directorio (clave de API por cuenta, sin identidades en su configuracion) y punto REST/MCP, con fallo cerrado probado. Cerrado el 2026-10-28.
