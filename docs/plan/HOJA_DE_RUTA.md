# Hoja de ruta del ente

Estado: activo
Version: 0.1 - 2026-10-27 (esqueleto, meta ADR 0014)

Nivel 0 del plan (meta ADR 0014). Solo hitos, orden y estado. El detalle vive
en el fichero de cada hito; un hito sin fichero aun no esta abierto.

## Idea final

1. Una infraestructura comun para consumir IA_NEST por REST y MCP, sobre la que
   se incorporan consumidores: Open WebUI, Home Assistant, Kodi, agentes y los
   que vengan.
2. La autenticacion es el punto de partida: un directorio de identidades propio
   de IA_NEST por el que pasan todos los consumidores.
3. La infraestructura del ente se disena antes de montarse: servicios en VMs
   que se migran y aseguran aparte; tron solo da recursos.

## Hitos

| Id | Hito | Depende de | Estado |
|---|---|---|---|
| [H01](hitos/H01_migracion_al_modelo.md) | Migrar plan, tarjetas y repos al modelo; orden de la casa | H00 | en curso |
| H02 | Criterio de apertura revisado: identidad central y punto REST/MCP (adenda a meta ADR 0013) | H00 | pendiente |
| H03 | Arquitectura de sistemas y servicios de IA_NEST (VM de recursos: directorio, DNS, DHCP...) | H01 | pendiente |
| H04 | Directorio de identidades, solo IA_NEST y preparado para crecer | H02, H03 | pendiente |
| H05 | Borde: identidades del directorio y punto REST/MCP comun | H04 | pendiente |
| H06 | TLS en todos los tramos que cruzan el perimetro | H03 | pendiente |
| H07 | Consumidores, uno a uno: Open WebUI, Home Assistant, Kodi, agente Linux | H05 | pendiente |
| H08 | Entrada de la familia | H06, H07, trabajo de capa | pendiente |

## Trabajo de capa que esta hoja vigila

Se conduce en su capa; aqui solo consta que bloquea a un hito. Se asigna en
`H01`.

| Que | Donde | Bloquea |
|---|---|---|
| `edge CR-0001` (perfil del interlocutor) | core#44 | H08 |
| `edge CR-0002` (extended lo compone) | extended#81 | H08 |
| L5r y L4b de la puerta de extended | extended, Deck #125 | H08 |

## Cerrados

- [H00](hitos/cerrados/H00_modelo_de_orquestacion.md): modelo de orquestacion desde meta (meta ADR 0014). Cerrado el 2026-10-27.
