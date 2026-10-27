# H02: criterio de apertura revisado (adenda a meta ADR 0013)

Estado: en curso
Abierto: 2026-10-27

## Objetivo

Que el criterio de salida de la apertura del ente recoja como punto de partida
lo que se perdio entre meta ADR 0013 y `edge ADR 0002`: la identidad sale de un
directorio central de IA_NEST, el borde ofrece un punto REST/MCP comun ademas de
la piel OpenAI, y los tramos que cruzan el perimetro van cifrados antes de que
entre la familia. Se hace con un ADR de meta que adenda 0013, sin reescribirlo.

No decide mecanismo ni maquina del directorio: eso es H03 y H04.

## Criterio de salida

1. Hay un ADR de meta que adenda meta ADR 0013 y cuyo criterio de salida
   incluye los tres puntos de partida, cada uno con al menos un criterio
   falsable y el hito que lo cumple (H04-H08).
2. Meta ADR 0013 tiene un `## Estado posterior` con el puntero; su punto 5
   (orden: el borde primero con Open WebUI) queda sustituido por la hoja de ruta.
3. El ADR deja escrita la leccion: lo que es punto de partida va en el criterio
   del ADR que abre el trabajo; un camino con disparador es un aplazamiento.
4. Cada entrada del inventario asignada a H04-H07 que toque identidad, punto
   REST/MCP o TLS (A09, A10, A11, A12, A14) esta cubierta por un criterio del
   ADR o fuera de el con motivo; A13 queda resuelta.

## Pasos

| Id | Que | Donde | Depende de | Estado |
|---|---|---|---|---|
| H02P1 | Situar al usuario y cerrar lo que el criterio no puede dejar abierto | este hilo | - | hecho: C4 decidido por el usuario (fallo cerrado) |
| H02P2 | ADR de meta con el criterio revisado y la leccion | meta `docs/decision_records/` | H02P1 | hecho: meta ADR 0015 |
| H02P3 | `## Estado posterior` en meta ADR 0013 | meta | H02P2 | hecho |
| H02P4 | Hoja de ruta al dia y PR de meta | meta | H02P3 | en curso |

## Reglas del hito

- Criterio falsable antes que mecanismo: ningun criterio nombra producto,
  protocolo de login ni maquina, salvo LDAPS, que es la forma de comprobarlo.
- Semilla de criterios: seccion 5 del diagnostico de la replanificacion
  (artifact del hilo de meta del 2026-10-27).
- Los ADR de otras capas (`edge ADR 0002`) no se reescriben; si hace falta
  rastro, se les apendiza puntero en su repo y con aprobacion.
